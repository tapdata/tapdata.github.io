---
title: Pre-Build Connectors
parent: Connectors
has_children: false
nav_order: 1
---
# Pre-Build Connectors

| DataBase Type | **Supported Version**        |
| ------------- | ---------------------------- |
| MongoDB       | 3.2, 3.4, 3.6, 4.0, 4.2      |
| Mysql         | 5.0、5.1、5.5、5.6、5.7、8.x |
| PostgreSQL    | 9.4+                         |



## MongoDB

### **Supported Version**

3.2, 3.4, 3.6, 4.0, 4.2

### Source Requirements

1. The source mongodb supports replica sets and sharding clusters.
2. f the source mongodb has only one node, you need configure it as a single member replication set to enable the oplog function.
3. You should configure enough oplog space. We recommend at least enough to accommodate 24-hour oplog.

4. If the source mongodb has security authentication enabled, the user account used by tapdata to connect to the source mongodb must have the following built-in roles

   1. `clusterMonitor`（For reading oplog ）

   2. `readAnyDatabase`

      ```
      > use admin
      > db.createUser({            
          "user" : "johndoe",
          "pwd"  : "my_password",            
          "roles" : [
              {
                  "role" : "clusterMonitor",
                  "db" : "admin"
              },
              {
                  "role" : "readAnyDatabase",
                  "db" : "admin"
              }
          ]
      }
      ```

      If you do not want to grant the `readanydatabase` role, you can also grant read permissions to specific databases and local and config databases

      ```
      > use admin
      > db.createUser({            
          "user" : "johndoe",
          "pwd"  : "my_password",            
          "roles" : [
              {
                  "role" : "clusterMonitor",
                  "db" : "admin"
              },
              {
                  "role" : "read",
                  "db" : "my_db"
              }，
              {
                  "role" : "read",
                  "db" : "local"
              },
              {
                  "role" : "read",
                  "db" : "config"
              }
          ]
      }
      ```

      





### Destination Requirements

1. The target mongodb supports replica sets and sharding clusters.

2. If the target mongodb has only one node, you need configure it as a single member replication set to enable the oplog function.

3. Ensure that sufficient resources are configured for the target mongodb to handle the workload of the source database.

4. If the target mongodb has security authentication enabled, the user account used by tapdata must have the following roles / permissions

   1. `clusterMonitor`

   2. `readWrite`

      ```
      > use admin
      > db.createUser({            
          "user" : "johndoe",
          "pwd"  : "my_password",            
          "roles" : [
              {
                  "role" : "clusterMonitor",
                  "db" : "admin"
              },
              {
                  "role" : "readWrite",
                  "db" : "my_db"
              },
              {
                  "role" : "read",
                  "db" : "local"
              }
          ]
      }
      ```

For more details you can see [here](https://www.yuque.com/tapdata/enterprise/data-source_about-dbs_mongodb)



## Mysql

### **Supported Version**

5.0、5.1、5.5、5.6、5.7、8.x  

### Requirements

1. You need to enable the binlog write function with row method.For example:

   ```
   [mysqld]
   server_id         = 223344
   log_bin           = mysql-bin
   binlog_format     = row
   binlog_row_image  = full
   ```

   

2. Prepare the account for Tapdata Source

   ```
   # 5.x
   create user 'username'@'localhost' identified by 'password';
   
   # 8.x
   create user 'username'@'localhost' identified with mysql_native_password by 'password';
   alter user 'username'@'localhost' identified with mysql_native_password by 'password';
   
   # grant privilege for the a database
   GRANT SELECT, SHOW VIEW, CREATE ROUTINE, LOCK TABLES ON <DATABASE_NAME>.<TABLE_NAME> TO 'tapdata' IDENTIFIED BY 'password';
   
   # grant privilege for all the databases
   GRANT RELOAD, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'tapdata' IDENTIFIED BY 'password';
   
   FLUSH PRIVILEGES;
   ```

3. Prepare the account for Tapdata Destination

   ```
   # 5.x
   create user 'username'@'localhost' identified by 'password';
   
   # 8.x
   create user 'username'@'localhost' identified with mysql_native_password by 'password';
   alter user 'username'@'localhost' identified with mysql_native_password by 'password';
   
   # grant privilege for the a database
   GRANT ALL PRIVILEGES ON <DATABASE_NAME>.<TABLE_NAME> TO 'tapdata' IDENTIFIED BY 'password';
   
   # grant privilege for all the databases
   GRANT PROCESS ON *.* TO 'tapdata' IDENTIFIED BY 'password';
   ```

   

For more details you can see [here](https://www.yuque.com/tapdata/enterprise/data-source_about-dbs_mysql)

## PostgreSQL

### **Supported Version**

9.4+

