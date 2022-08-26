---
layout: post
title: Kinesis Data Analytics boto3 waiter
date: 2022-08-26
author: Fraser Clark
comments: true
tags: [python, aws, boto3, kda]
---

There is no waiter configuration for kinesis data analytics applications. 
Recently I had to write the config for one myself.
The below can be used in a file and imported to a script as a module.

## KDA waiter

<details><summary markdown='span'>Click to expand</summary>

```python

from botocore.waiter import WaiterModel, create_waiter_with_client

class KdaWaiter():
    """
    Class that contains all KDA waiter config & functions
    """

    def __init__(self,app,snapshot_name,boto_client):
        self.ApplicationName = app
        self.IncludeAdditionalDetails = False
        self.SnapshotName = snapshot_name
        self.boto_client = boto_client

    def kda_running(self):
        """
        Waits for KDA app to reach "RUNNING" status
        The apps must be in this status to be worked with

        Returns:
            None
        """
        waiter_name = "KdaRunning"
        waiter_config = {
            "version": 2,
            "waiters": {
                "KdaRunning": {
                    "operation": "describe_application",
                    "delay": 15,  # Number of seconds to delay
                    "maxAttempts": 30,  # Max attempts before failure
                    "acceptors": [
                        {
                            "matcher": "path",
                            "expected": "STARTING",
                            "argument": "ApplicationDetail.ApplicationStatus",
                            "state": "retry"
                        },
                        {
                            "matcher": "path",
                            "expected": "UPDATING",
                            "argument": "ApplicationDetail.ApplicationStatus",
                            "state": "retry"
                        },                        
                        {
                            "matcher": "path",
                            "expected": "RUNNING",
                            "argument": "ApplicationDetail.ApplicationStatus",
                            "state": "success"
                        }
                    ]
                }
            }
        }
        waiter_model = WaiterModel(waiter_config)
        kda_app_status_waiter = create_waiter_with_client(
            waiter_name, waiter_model, self.boto_client)

        kda_app_status_waiter.wait(ApplicationName=self.ApplicationName,
                                   IncludeAdditionalDetails=self.IncludeAdditionalDetails)

        return kda_app_status_waiter

    def kda_stopped(self):
        """
        Waits for KDA app to reach "READY" status
        This status is when the app is disabled/stopped. Calling it ready is a bit of a misnomer

        Returns:
            None
        """
        waiter_name = "KdaStopped"
        waiter_config = {
            "version": 2,
            "waiters": {
                "KdaStopped": {
                    "operation": "describe_application",
                    "delay": 15,  # Number of seconds to delay
                    "maxAttempts": 30,  # Max attempts before failure
                    "acceptors": [
                        {
                            "matcher": "path",
                            "expected": "STOPPING",
                            "argument": "ApplicationDetail.ApplicationStatus",
                            "state": "retry"
                        },
                        {
                            "matcher": "path",
                            "expected": "FORCE_STOPPING",
                            "argument": "ApplicationDetail.ApplicationStatus",
                            "state": "retry"
                        },
                        {
                            "matcher": "path",
                            "expected": "READY",
                            "argument": "ApplicationDetail.ApplicationStatus",
                            "state": "success"
                        }
                    ]
                }
            }
        }
        waiter_model = WaiterModel(waiter_config)
        kda_app_status_waiter = create_waiter_with_client(
            waiter_name, waiter_model, self.boto_client)

        kda_app_status_waiter.wait(ApplicationName=self.ApplicationName,
                                   IncludeAdditionalDetails=self.IncludeAdditionalDetails)

        return kda_app_status_waiter

    def snapshot_waiter(self):
        """
        Waits for snapshot of app to reach "READY" status
        This means the snapshot has been created successfully

        Returns:
            None
        """
        waiter_name = "SnapshotReady"
        waiter_config = {
            "version": 2,
            "waiters": {
                "SnapshotReady": {
                    "operation": "describe_application_snapshot",
                    "delay": 5,  # Number of seconds to delay
                    "maxAttempts": 30,  # Max attempts before failure
                    "acceptors": [
                        {
                            "matcher": "path",
                            "expected": "FAILED",
                            "argument": "SnapshotDetails.SnapshotStatus",
                            "state": "failure"
                        },
                        {
                            "matcher": "path",
                            "expected": "CREATING",
                            "argument": "SnapshotDetails.SnapshotStatus",
                            "state": "retry"
                        },
                        {
                            "matcher": "path",
                            "expected": "READY",
                            "argument": "SnapshotDetails.SnapshotStatus",
                            "state": "success"
                        },
                    ]
                }
            }
        }
        waiter_model = WaiterModel(waiter_config)
        kda_snap_status_waiter = create_waiter_with_client(
            waiter_name, waiter_model, self.boto_client)

        kda_snap_status_waiter.wait(
            ApplicationName=self.ApplicationName, SnapshotName=self.SnapshotName)

        return kda_snap_status_waiter

```
</details>

