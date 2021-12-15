---
layout: post
title: Monitoring Apache Airflow failed DAG runs with Python
date: 2021-12-1
author: Fraser Clark
comments: true
tags: [airflow, python, aws, lambda]
---

Recently I was asked to write a script to monitor & alert on failed DAG runs on Apache Airflow.
This script was to run on AWS lambda.

# Airflow
To really really over simplify Airflow, it is just a UI for cron. It runs python scripts which are called DAG's on a cron schedule.

All I wanted to be able to do was query failed DAG runs and write to cloudwatch logs and also report to a slack channel.

## The API endpoint
What allowed this is that Airflow released a new API endpoint to query DAG runs, this was previously experimental.
With this release it meant I could query for DAG runs and pick out the failed ones.

## Basic overview

The basic running is as follows:

- Queries airflow API
- Filters for any failed DAG runs
- Logs these failed runs to cloudwatch logs
- Then checks cloudwatch logs to see if these DAG ID's have alerted in the last 30 mins
- If not, then it will alert slack/email etc.

## Caveats

- All infrastructure is created via Terraform
- The environment variables are passed in during the Terraform apply to the lambda function

## Difficulties
The major issue with the current iteration is that it cannot alert on state changes, such as a DAG moving from `failed` to `success`.

This is because I am not keeping a list of the DAG's and their current state in a database or anything like that. Future iterations may have this functionality however.

## The script

<details>
    <summary>Click to expand script</summary>

```python

import json
import logging
import os
import time
from datetime import datetime, timedelta

import boto3
from botocore.exceptions import ClientError
import requests
from requests.exceptions import HTTPError

# Overview
#
# script runs every 15 minutes
# lambda_handler() function triggers get_dag_runs() function and loops over the output
# Then all failed runs are sent to cloudwatch logs
# check_cw() function then runs and checks cloudwatch logs for any mention of the failed dag ID's
# if failed dags are found in the logs then we can ignore it because it has been alerting already
# but if no logs are found, then it will alert to slack


# initalise and set logging level
logging.basicConfig()
logger = logging.getLogger()
logger.setLevel(logging.INFO)

# get time, convert to utc, minus 15 minutes
# then convert to a string that the api is expecting
time_minus = datetime.utcnow() - timedelta(minutes=15)
date_time = time_minus.strftime("%Y-%m-%dT%H:%M:%S")

# get environment variables

# get api url and remove trailing "/"
api_url = os.environ.get("AIRFLOW_URL").strip("/")
# get airflow server name for logging & error messages
airflow_server = os.environ.get("AIRFLOW_SERVER_NAME", "unknown")
generic_slack_lambda = os.environ.get("GENERIC_SLACK_LAMBDA")
slack_channel = os.environ.get("SLACK_ALERT_CHANNEL_NAME") 


lambda_client = boto3.client("lambda")


def get_dag_runs():
    """ 
    - queries the dags endpoint to get a list of active dags
    - gets a list of dag ids from the output and adds them to a list

    Returns:
        [list]: [list of dictionaries of dag runs]
    """

    try:
        logger.info("Retrieving DAGs...")
        response = requests.get("https://{}/api/v1/dags".format(api_url))
        dags = response.json()
    except HTTPError as e:
        logger.error("Error retrieving DAGs: {}".format(e))
        return

    # get list of dags if the dag is active and not paused
    list_of_dags = [x['dag_id'] for x in dags['dags']
                    if x['is_active'] and not x['is_paused']]

    # empty list to hold dag runs
    dag_runs = []
    failed_dag_runs = []

    # loop over dags returned
    for x in list_of_dags:
        try:
            logger.info("Retrieving DAG run ID's for {}".format(x))
            url = "https://{}/api/v1/dags/{}/dagRuns".format(api_url, x)
            # limit can be modified to increase runs returned.
            params = {'execution_date_gte': '{}'.format(date_time)}

            response_dags = requests.get(url, params=params)
            output = response_dags.json()
            # removes the the outside list to make working with the data easier
            dag_runs.append(output['dag_runs'])
        except HTTPError as e:
            logger.error("Error retrieving DAG runs: {}".format(e))
        # loop over dag runs and get the failed ones
    for i in dag_runs:
        for j in i:
            if j['state'] == "failed":
                failed_dag_runs.append(j)
            else:
                pass
                    
    # if there are no failed runs, then exit the program
    # otherwise, return the failed_dag_runs list
    if not failed_dag_runs:
        raise SystemExit("No DAG's found with state 'failed'")
    else:
        return failed_dag_runs

def check_cw(failed_list):
    """This function is used to check cloudwatch logs over the last 30 minutes
      - the idea is that it will loop over the dag ID's inside of failed_list
      - if it can find the dag ID in a cloudwatch log entry then we can assume it is still alerting and then do nothing
      - if the dag ID is not found in a cloudwatch log entry then we assume this is a new alert and the dag ID is sent to the notify_slack() function

    Args:
        failed_list ([list]): [list of dag ID's that have failing runs]

    Returns:
        [list]: [list of dag ID's to add the ignore list, stored in paramater store]
    """

    client = boto3.client('logs')
    # remove duplicates from the list so that slack doesn't get spammed with alerts for one DAG that is failing 50 times.
    # we just want to alert to the fact that a DAG is failing, not that it has x amount of failures
    remove_dupes = list(set(failed_list))

    for x in remove_dupes:
        # cloudwatch log insights query, searching for the specific dag ID
        query = "fields @timestamp, @message | sort @timestamp desc | limit 1 | filter dag_id = {}".format(x)
        print(query)
        # set log group name
        log_group = (f"{airflow_server}/failed_dag_runs").replace(".", "-")

        # run query against cloudwatch logs
        # this only returns a 'query_id', it does not return any results/logs
        try:
            logger.info("querying cloudwatch logs...")
            start_query_response = client.start_query(
                logGroupName=log_group,
                # search from 30 minutes ago
                startTime=int((datetime.today() - timedelta(minutes=30)).timestamp()),
                endTime=int(datetime.now().timestamp()),
                queryString=query,
            )
        except ClientError as e:
            logger.error(e)

        query_id = start_query_response['queryId']
        response = None

        # wait for the query to finish
        while response == None or response['status'] == 'Running':
            time.sleep(1)
            response = client.get_query_results(
                queryId=query_id
            )

        # if not alerted in last 30 minutes then alert
        if not response['results']:
            logger.info(f"alerting slack with the following failed runs - {x}")
            notify_slack(failed_list)
        # otherwise, ignore it & log
        else:
            logger.info(f"{response['results']} have been ignored")



def send_to_cw_logs(log):
    """Function to send to cloudwatch logs

    Args:
        log (json): json formatted log
    """

    # configure boto3 cloudwatch logs client
    client = boto3.client('logs')
    # set log stream name
    # set log group name, this is created in cloudwatch_dag_monitor.tf
    log_group  = (f"{airflow_server}/failed_dag_runs")
    log_stream = '{}'.format(time.strftime('%Y-%m-%d'))

    # tries to create a new log stream
    try:
        client.create_log_stream(
            logGroupName=log_group, logStreamName=log_stream)
    except client.exceptions.ResourceAlreadyExistsException:
        pass

    # querying the log stream is required to get the 'sequenceToken'
    # if you do not pass the 'sequenceToken' then it will error (unless it is a new log stream)
    # the 'sequenceToken' will be empty if is a new log stream
    response = client.describe_log_streams(
        logGroupName=log_group,
        logStreamNamePrefix=log_stream
    )
    # get the event batch ready for cloudwatch logs
    event_log = {
        'logGroupName': log_group,
        'logStreamName': log_stream,
        'logEvents': log,
    }
    # this if statement sets the 'sequenceToken' if required
    if 'uploadSequenceToken' in response['logStreams'][0]:
        event_log.update(
            {'sequenceToken': response['logStreams'][0]['uploadSequenceToken']})

    # now send the events to cloudwatch
    response = client.put_log_events(**event_log)

def notify_slack(dag_id):
    """sends events to slack

    Args:
        dag_id (list): list of dag ID's which should be sent to slack
    """

    lambda_client.invoke(
    FunctionName=generic_slack_lambda,
    InvocationType="Event",
    Payload=json.dumps({
        "type": "DAG Failure Alert",
        "message": f"The following DAGs on {api_url} have failed runs.",
        "channel": slack_channel,
        "additional": {
            "dag_ids": dag_id
        }
    })
    )


def lambda_handler(event, context):
    """
    - lambda entrypoint
    - triggers get_dag_runs function and adds the output to a dictionary, which is then added to a list and then passed to the cloudwatch function
    - the reason for this is so that the messages can be sent as a batch, rather than PUTing each message individually
    - it also grabs the dag_id from each message and adds this to a list which the check_cw function will take as an argument
    - once all logs are sent to cloudwatch, the check_cw function is triggered
    - finally the time it took to run is printed out
    """
    # start recording execution time
    start_time = datetime.now()

    log_events = []
    dag_ids = []

    # loop over output of get_dag_runs() function
    for x in get_dag_runs():
        # add each entry to a dictionary that is formatted for cloudwatch logs
        batch_dict = {
            'timestamp': int(round(time.time() * 1000)),
            'message': json.dumps(x)
        }
        # add that dictionary to the log_events list
        log_events.append(batch_dict)
        # grab the dag_id's and add it to the dag_ids list
        dag_ids.append((json.dumps(x['dag_id'])))
    
    # send the logs to cloudwatch logs
    send_to_cw_logs(log_events)
    # trigger the check_cw function
    check_cw(dag_ids)

    # print the time it took to run this because why not
    logger.info(f"Elapsed time: {str(datetime.now() - start_time)}")
```
</details>

