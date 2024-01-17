---
layout: post
title: Migrating grafana from sqllite to RDS postgres
date: 2024-01-17
author: Fraser Clark
comments: true
tags: [aws, rds, grafana, sqllite, pgloader]
---

I had to migrate our self-hosted grafana instance from the default sqllite DB to RDS postgres.
I ran into a few issues while doing so, so have documented them here.


## steps 

The steps to migrate were:

1. Create RDS db
2. Prepare DB
3. Connect grafana to new DB

It seems simple but step 2 was where I had the most issues.
I will not cover creating an RDS database.

### Preparing database

I found this guide which got me most of the way there - [link](https://polyglot.jamie.ly/programming/2019/07/01/grafana-sqlite-to-postgres.html)

Once you have the database created you need to get create the schema grafana is expecting. The blog above explains that. Basically you create a dummy postgres DB using docker and get grafana to run DB migrations against it then export the schema.

Once you have the schema you can then run it as a `.sql` script against your new database which will create all the tables etc.

Then you can use a tool called `pgloader` to load the old sqllite DB into the new postgres one.
I was using Amazon Linux 2 and this is where I had the most issues.

### pgloader on Amazon Linux 2

To get pgloader working I had to follow the rpmbuild steps [here](https://github.com/dimitri/pgloader/blob/master/INSTALL.md) for Redhat/CentOS.

The steps for dependencies were:
```bash
#install EPEL
sudo amazon-linux-extras install epel -y
#install rpmbuild dependencies
sudo yum -y install yum-utils rpmdevtools @"Development Tools"
#install pgloader build dependencies
sudo yum-builddep pgloader.spec
```
It was at this point you should then use the `pgloader.spec` file to build the rpm etc. but I tried this and at the end the rpm would not install.

What I ended up doing was just downloading the latest release .tar and simply running `make`.

Once I did this I was able to run `./build/bin/pgloader –help`