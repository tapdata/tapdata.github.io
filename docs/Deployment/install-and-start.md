---
title: 「WIP」Install and Start
parent: Deployment
has_children: false
nav_order: 2
---
# Install and Start
This document will be updated to the official version on June 30.

## Prerequisites

Please make sure that you have read the following documents:

- [Requirements and checklist](./requirements-and-checklist.md)

## Installation

## With Docker

	docker pull xxx
	docker run 

## With git

	git clone https://github.com/tapdata/tapdata.git
	cd tapdata
	mvn clean package
	# add ./bin to the execution path
	tapdata start


​	

## Verify Installation

	



## Backup

```
```









## What's next

After you've installed Tapdata, you can make your data on tap, see the following documents:

- [Transform Data from MySQL to MySQL in Real-Time](../Use Cases/ETL/mysql-to-mysql.md)
- [Transform Data from MySQL to MongoDB in Real-Time](../Use Cases/ETL/mysql-to-mongodb.md)
- [Data processing during the Transform](../Use Cases/ETL/data-develop-via-movement.md)
