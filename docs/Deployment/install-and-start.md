---
title: Install and Start
parent: Deployment
has_children: false
nav_order: 2
---
# Install and Start

## Prerequisites

Please make user you have Docker running on your env, before all things started.

Only linux system is tested until now.


## Installation

### With Git
1. Clone Tapdata Repo.
2. Run `bash build/quick-use.sh` will pull and start a all in one container


## Quick Dev
1. Clone Tapdata Repo.
2. run `bash build/quick-dev.sh` will build a docker image from source and start a all in one container

If you want to build in docker, please install docker and set build/env.sh tapdata_build_env to "docker" (default)

If you want to build in local, please install:
1. JDK
2. maven
set build/env.sh tapdata_build_env to "local"

run `bash build/clean.sh` If you want to clean build target
	



## What's next

After you've installed Tapdata, you can make your data on tap, see the following documents:

- [Transform Data from MySQL to MySQL in Real-Time](../Use Cases/ETL/mysql-to-mysql.md)
- [Transform Data from MySQL to MongoDB in Real-Time](../Use Cases/ETL/mysql-to-mongodb.md)
- [Data processing during the Transform](../Use Cases/ETL/data-develop-via-movement.md)
