---
title: Sync Data from MongoDB to MongoDB
parent: Use Cases
has_children: false
nav_order: 3
---
# Sync Data from MongoDB to MongoDB

## Background

In many cases, we need to transfer MongoDB data to another MongoDB, such as:

- 「Database replication」Transfer a database of MongoDB instance to another MongoDB instance in the other Server/IDC/Cloud/Region.
- 「Table Aggregation and Data Development」Merge multiple tables into one  or split one table to many with UDF

The supported versions of MongoDB are shown [here](../../Connectors/pre-build-connectors.md#MongoDB)

## Database replication

### Prerequisites

#### Source 

| Filed                      | **Value**                 |
| -------------------------- | ------------------------- |
| Source Instance  (MongoDB) | 192.168.1.1               |
| Source Instance Port       | 27017                     |
| Source Database            | Car_shop                  |
| Source Tables              | All the table in Car_shop |

#### Target 

| Filed                      | **Value**   |
| -------------------------- | ----------- |
| Target Instance  (MongoDB) | 192.168.1.2 |
| Target Instance Port       | 27017       |
| Target Database            | Car_shop    |

### Action

```
> # Create Source DataSource
> Source_MongoDB = DataSource("mongodb","Source_MongoDB",'source').uri("mongodb://root:password@192.168.1.1:27017/Car_shop?authSource=admin")
> Source_MongoDB.save()

# Create Target DataSource
> Target_MongoDB = DataSource("mongodb","Target_MongoDB",'target').uri("mongodb://root:password@192.168.1.2:27017/Car_shop?authSource=admin")
> Target_MongoDB.save()

# Create a job that transform all the tables in Source-Mysql to Target-MongoDB.
> replication_job = Pipeline("replication_job").readFrom(Source(Source_MongoDB,table_re=".*")).writeTo(Target_MongoDB)

> replication_job.start()

# Check the status of job
> show jobs
> monitor job replication_job

# Check the log of job
> logs job replication_job limit=5 tail=True 
```

After these steps you can login to the  target MongoDB and see the new data.

## Table Aggregation and Data Development

### Prerequisites

#### Source 

| Filed                     | **Value**            |
| ------------------------- | -------------------- |
| Source Instance (MongoDB) | 192.168.1.1          |
| Source Instance Port      | 3306                 |
| Source Database           | Car_shop             |
| Source Tables             | Orders<br />Products |

#### Target 

| Filed                        | **Value**           |
| ---------------------------- | ------------------- |
| Target Instance  （MongoDB） | 192.168.1.2         |
| Target Instance Port         | 27017               |
| Target Database              | Car_shop            |
| Target Table                 | Orders_and_Products |

#### Requirement

![](/Users/edisonchow/tapdata/tapdata-community-docs/assets/mysql-to-mysql-1.png)



![](/Users/edisonchow/tapdata/tapdata-community-docs/assets/mysql-to-mysql-2.png)







### Action

```
# Write the js script  「find_product_name.js」

	record["updated_at"] = new Date();
	var rs = target.executeQuery({
    database: “Car_shop”,
    collection: “Products”,
    filter: {id: record.product_id}，
    limit: 1
});
	if (rs) {
		record["product_name"] = rs.name;
	} else {
		log.warn("Unable to find product_name is products tabel with : " + record.product_id);
		record["product_name"] = "Not Found";
	}
	return record;

```





```
> # Create Source DataSource
> Source_MongoDB = DataSource("mongodb","Source_MongoDB",'target').uri("mongodb://root:password@192.168.1.1:27017/Car_shop?authSource=admin")
> Source_MongoDB.save()

# Create Target DataSource
> Target_MongoDB = DataSource("mongodb","Target_MongoDB",'target').uri("mongodb://root:password@192.168.1.2:27017/Car_shop?authSource=admin")
> Target_MongoDB.save()

# Create a job that transform from Source_Mysql to Target_Mysql.

> replication_job = Pipeline("replication_job").readFrom(Source_MongoDB.Orders).filterColumn(["id","detail","created_at","product_id"],FilterType.keep).js("/path/find_product_name.js").writeTo("Target_MongoDB.Orders_and_Products",writeMode=WriteMode.upsert, association=[("id", "id")])


> replication_job.start()

# Check the status of job
> show jobs
> monitor job replication_job

# Check the log of job
> logs job replication_job limit=5 tail=True 
```

After these steps you can login to the  target MongoDB and see the new data.