---
title: Quick Start
has_children: false
nav_order: 3
---
# Quick Start

This guide walks you through the quickest way to get started with Tapdata using [Docker](https://github.com/docker) .


If you want to use the existing mongodb to save the tapdata metadata, please refer to [Installation](../Deployment/install-and-start.md) for steps. 

## Prerequisites

Make sure [Docker](https://www.docker.com/) and Git has been successfully installed. 



## Let's make data on tap!

1. Install  

```
# Clone tapdata repo
git clone https://github.com/tapdata/tapdata.git

# Get start!
bash build/quick-use.sh
```



2. Check the service status

```
docker  ps |grep tapdata

```



3. Create Tapdata DataSource & Job to transform the Data from mysql to mongoDB.

```
Suppose the demo Mysql infomation:
host: demo-mysql
port: 3306
username: root
password: password
database: demo
```

```
Suppose the demo MongoDB infomation:
host: demo-mongo
port: 27017
username: root
password: password
database: demo
```

```
# Login to Tapdata using ishell
docker-compose exec ishell ishell 

# Demo Database is 'demo'
# Demo Table is 'Customer'

# Create Mysql DataSource
> Demo_Mysql = DataSource("mysql","Demo_Mysql").host("demo-mysql").port(3306).username('root').password('password').db('demo')
> Demo_Mysql.save()

# Create MongoDB DataSource
> Demo_Mongo = DataSource("mongodb","Demo_Mongo").uri("mongodb://root:password@demo-mongo:27017/demo?authSource=admin")
> Demo_Mongo.save()

# Create a job that transform the Customer table in Mysql to  MongoDB  and add/set filed 'updated' at the same time.
> Demo_job = Pipeline("Demo_job").readFrom(Demo_Mysql.Customer).js('record["updated"]=new Date() ;return record;').writeTo(Demo_Mongo.Customer-v1)

> Demo_job.start()

# Check the status of job
> show jobs
> monitor job Demo_job

# Check the log of job
> logs job Demo_job limit=5 tail=True 


```










