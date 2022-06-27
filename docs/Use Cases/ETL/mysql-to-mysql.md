---
title: Sync Data from MySQL to MySQL
parent: Use Cases
has_children: false
nav_order: 1
---
# Sync Data from MySQL to MySQL


## Background

In many cases, we need to transfer MySQL data to another MySQL, such as:

- 「Database replication」Transfer a database of MySQL instance to another MySQL instance in the other Server/IDC/Cloud/Region.
- 「Building Materialized View」Merge multiple tables into one  or split one table to many  

The supported versions of MySQL are shown [here](../../Connectors/pre-build-connectors.md#Mysql)

## Database replication

### Setup

Assuming you have two MySQL databases as follows, and you want to sync all the data from 192.168.1.1 to 192.168.1.2.

#### Source 

| Filed                | **Value**                  |
| -------------------- | -------------------------- |
| Source Instance      | 192.168.1.1                |
| Source Instance Port | 3306 (Version 5.7)         |
| Source Database      | Car_shop (already created) |
| Source Tables        | All the table in Car_shop  |

#### Target 

| Filed                | **Value**                  |
| -------------------- | -------------------------- |
| Target Instance      | 192.168.1.2                |
| Target Instance Port | 3306 (Version 5.7)         |
| Target Database      | Car_shop (already created) |



### Action

```
> # Create Source DataSource
> Source_Mysql = DataSource("mysql","Source_Mysql",'source').host("192.168.1.1").port(3306).username('root').password('password').db('Car_shop')
> Source_Mysql.save()

# Create Target DataSource
> Target_Mysql = DataSource("mysql","Target_Mysql",'target').host("192.168.1.2").port(3306).username('root').password('password').db('Car_shop')
> Target_Mysql.save()

# Create a job that transform all the tables in Source_Mysql to Target_Mysql.
> replication_job = Pipeline("replication_job").readFrom(Source(Source_Mysql,table_re=".*")).writeTo(Target_Mysql)
# Source(Source_Mysql,table_re=".*"): Specify the data source, second parameter is a regular expression, you may it to specify multiple tables. 

> replication_job.start()

# Check the status of job
> show jobs
> monitor job replication_job

# Check the log of job
> logs job replication_job limit=5 tail=True 
```

After these steps you can login to the  target MySQL and see the new data.

## Create Materialized View (of Orders)

### Setup

Assuming you have two MySQL databases as follows:

#### Source 

| Filed                | **Value**                  |
| -------------------- | -------------------------- |
| Source Instance      | 192.168.1.1                |
| Source Instance Port | 3306 (Version 5.7)         |
| Source Database      | Car_shop (already created) |
| Source Tables        | Orders<br />Products       |

#### Target 

| Filed                | **Value**                             |
| -------------------- | ------------------------------------- |
| Target Instance      | 192.168.1.2                           |
| Target Instance Port | 3306 (Version 5.7)                    |
| Target Database      | Car_shop                              |
| Target Table         | Orders_and_Products (already created) |

#### Requirement

We would like to create a new Orders table with some product fields as part of the schema . Order & Product are two tables joined by product_id.

![](../../../assets/mysql-to-mysql-1.png)



![](../../../assets/mysql-to-mysql-2.png)



### Action
 



```
> # Create Source DataSource
> Source_Mysql = DataSource("mysql","Source_Mysql",'source').host("192.168.1.1").port(3306).username('root').password('password').db('Car_shop')
> Source_Mysql.save()

# Create Target DataSource
> Target_Mysql = DataSource("mysql","Target_Mysql",'target').host("192.168.1.2").port(3306).username('root').password('password').db('Car_shop')
> Target_Mysql.save()

# Create a job that transform from Source_Mysql to Target_Mysql.

> replication_job = Pipeline("order_rep_job").readFrom(Source_Mysql.Orders).filterColumn(["id","detail","created_at","product_id"],FilterType.keep)


> job2 = Pipeline("product_info").readFrom(Source_MySQL.Products).filterColumn(["id","product_name"])

> replication_job.merge(job2).writeTo(Target_Mysql.Orders_and_Products,writeMode=WriteMode.upsert, association=[("id", "id")])

> replication_job.start()

# Check the status of job
> show jobs
> monitor job replication_job

# Check the log of job
> logs job replication_job limit=5 tail=True 
```

After these steps you can login to the  target MySQL and see the new data.
