## **Tapdata 数据源 Connector 大赛规则**
### **一、行为守则**
我同意遵守该项目的[行为准则](https://www.apache.org/foundation/policies/conduct)
### **二、如何参与？**
1. 领取任务：Fork本页面，在数据源列表中相应数据源的贡献者栏位里，写上自己的github username，提交PR
2. 审核任务：管理人员会review PR， 合并PR即表示领取成功
3. 开发阶段：成功认领后，可进行开发
4. 提交结果：开发结束后可将结果以 issue/PR 的形式提交认领任务的评论列表中，管理人员将会进行审核/通过。
5. 奖品申领： 页面管理人员会将审核通过的结果，呈现在此报名页面，显示通过者可以提交[奖品申领表单](https://tapdata.mike-x.com/PEM8K)。
6. 通过奖励：Tapdata 产研会协同评估该新数据源的完成质量和市场价值给予开发者丰厚的奖励。每月15日中午12：00前完成任务的，会在25日发放对应奖品激励。每月30日中午12：00前完成任务的，会在次月10日发放对应奖品激励。
7. “邀请参赛者” 玩法：邀请参赛者，赢 Tapdata 定制礼品
   无论是参赛者还是未参赛者，都可以将挑战赛分享给好友并邀请好友参赛，达标后您将获得 Tapdata 双肩包、T恤、多功能无线充电套装、技术图书等丰厚礼品。
   ![image](https://user-images.githubusercontent.com/25000091/192415999-ba035cc2-4224-4bad-b374-2d25ad7d6264.png)

注意：避免重复认领，合理分配任务时间，让每位开发者都有机会参与每个任务中。
- 每位开发者至少认领1个任务，1名开发者可领取多个任务；
- 认领任务后，如果认领者超过3周，未提交对应的数据源结果，则该数据源任务释放，除认领者以外的开发者可重新认领。

### **三、如何使用 Tapdata 完成数据源接入任务？**
1. 开发环境
- 目前该项目兼容Windows、Linux、Mac 等操作系统
- 由于 GitHub 网络状况，最好有 VPN 翻墙工具
- JDK 8
- Maven 3.6.3以上（配置阿里云、腾讯云镜像）
2. 开发准备
- GitHub右上角在fork标签下create  a new fork
- 拉取代码：git clone https://github.com/【username】/tapdata.git
- 新建feat-<数据源昵称>分支
- Load maven project（仅需plugin-kit、connectors-common、connectors）
- 按plugin-kit、connectors-common、connectors先后顺序编译：
  mvn clean package -DskipTests -P not_encrypt
  等全量都编译通过，那么开发环境已经就绪，下面就正式进入研发新数据源阶段

3. [开发测试指南](https://github.com/tapdata/tapdata.github.io/blob/main/plugin-contributor-program/plugin-development-guide.md)

### **四、在运行过程中有任何问题可在 issues 中提问**
https://github.com/tapdata/tapdata/issues
### **五、任务奖励列表（详情见开发指南）**

| 任务  |  	Connector 主流程  |          	奖品金额          |
|:---:|:----------------:|:-----------------------:|
| 任务一 | 	目标数据写入、源数据全量读取	 |          2500           |
| 任务二 |    	源数据增量读取	     | 简易-第1档：2500 困难-第2档：5000 |

### **六、数据连接器列表**

|             数据库              |  源/目标  | 贡献者 | Issue/PR | 难度划分 |
|:----------------------------:|:------:|:---:|:--------:|:----:|
|           PolarDB            | Source |     |          |  1   |
|           PolarDB            | Target |     |          |  1   |
|            TDSQL             | Source |     |          |  2   |
|            TDSQL             | Target |     |          |  1   |
|          AnalyticDB          | Target |     |          |  1   |
|          SequoiaDB           | Source |     |          |  1   |
|          SequoiaDB           | Target |     |          |  1   |
|           TDengine           | Source |     |          |  1   |
|           TDengine           | Target |     |          |  1   |
|          TcaplusDB           | Source |     |          |  1   |
|          TcaplusDB           | Target |     |          |  1   |
|           GoldenDB           | Source |     |          |  1   |
|           GoldenDB           | Target |     |          |  1   |
|         GaussDB(DWS)         | Target |     |          |  1   |
|            AntDB             | Source |     |          |  1   |
|            AntDB             | Target |     |          |  1   |
|           OushuDB            | Target |     |          |  1   |
|            SUNDB             | Source |     |          |  1   |
|            SUNDB             | Target |     |          |  1   |
|            MogDB             | Source |     |          |  1   |
|            MogDB             | Target |     |          |  1   |
|             UXDB             | Source |     |          |  1   |
|             UXDB             | Target |     |          |  1   |
|             神舟通用             | Source |     |          |  1   |
|             神舟通用             | Target |     |          |  1   |
|          DolphinDB           | Source |     |          |  1   |
|          DolphinDB           | Target |     |          |  1   |
|           LightDB            | Source |     |          |  1   |
|           LightDB            | Target |     |          |  1   |
|           RapidsDB           | Source |     |          |  1   |
|           RapidsDB           | Target |     |          |  1   |
|           GreatDB            | Source |     |          |  1   |
|           GreatDB            | Target |     |          |  1   |
|          StarRocks           | Target |     |          |  1   |
|          CirroData           | Target |     |          |  1   |
|             TGDB             | Target |     |          |  1   |
|            Nebula            | Target |     |          |  1   |
|           Gbase 8a           | Source |     |          |  1   |
|           KunlunDB           | Target |     |          |  1   |
|           DataHub            | Source |     |          |  1   |
|           DataHub            | Target |     |          |  1   |
|          MaxCompute          | Target |     |          |  1   |
|        PolarDB MySQL         | Source |     |          |  1   |
|        PolarDB MySQL         | Target |     |          |  1   |
|       阿里云消息队列 Kafka 版        | Source |     |          |  1   |
|       阿里云消息队列 Kafka 版        | Target |     |          |  1   |
|       表格存储 Tablestore        | Target |  jiuyetx   |          |  2   |
|   阿里云数据库 RDS SQL Server 版    | Source |     |          |  1   |
|   阿里云数据库 RDS SQL Server 版    | Target |     |          |  1   |
|     云原生分布式数据库 PolarDB-X      | Source |     |          |  1   |
|     云原生分布式数据库 PolarDB-X      | Target |     |          |  1   |
|             DDS              | Source |     |          |  1   |
|             DDS              | Target |     |          |  1   |
|  华为云数据库 GaussDB(for MySQL)   | Source |     |          |  2   |
|  华为云数据库 GaussDB(for MySQL)   | Target |     |          |  1   |
|          华为云 Kafka           | Source |     |          |  1   |
|          华为云 Kafka           | Target |     |          |  1   |
|          MySQL分库分表           | Source |     |          |  1   |
|          MySQL分库分表           | Target |     |          |  1   |
|  华为云数据库 RDS for SQL Server   | Source |     |          |  1   |
|  华为云数据库 RDS for SQL Server   | Target |     |          |  1   |
|     华为云数据库 RDS for MySQL     | Source |     |          |  1   |
|     华为云数据库 RDS for MySQL     | Target |     |          |  1   |
|  华为云数据库 RDS for PostgreSQL   | Source |     |          |  1   |
|  华为云数据库 RDS for PostgreSQL   | Target |     |          |  1   |
|           Percona            | Source |     |          |  1   |
|           Percona            | Target |     |          |  1   |
|        腾讯云数据库 MariaDB        | Source |     |          |  1   |
|        腾讯云数据库 MariaDB        | Target |     |          |  1   |
|       腾讯云原生数据库 TDSQL-C       | Source |     |          |  1   |
|       腾讯云原生数据库 TDSQL-C       | Target |     |          |  1   |
|       TDSQL-C MySQL 版        | Source |     |          |  1   |
|       TDSQL-C MySQL 版        | Target |     |          |  1   |
|     TDSQL-C PostgreSQL 版     | Source |     |          |  1   |
|     TDSQL-C PostgreSQL 版     | Target |     |          |  1   |
|       Microsoft Access       | Source |     |          |  1   |
|       Microsoft Access       | Target |     |          |  1   |
|            SQLite            | Source |     |          |  1   |
|            SQLite            | Target |     |          |  1   |
|          Cassandra           | Target |     |          |  1   |
|           Redshift           | Target |     |          |  1   |
|          Snowflake           | Target |     |          |  2   |
|            Splunk            | Target |     |          |  1   |
|       Amazon DynamoDB        | Source |     |          |  1   |
|       Amazon DynamoDB        | Target |     |          |  1   |
| Microsoft Azure SQL Database | Source |     |          |  1   |
| Microsoft Azure SQL Database | Target |     |          |  1   |
|           Teradata           | Target |     |          |  1   |
|            Neo4j             | Source |     |          |  1   |
|            Neo5j             | Target |     |          |  1   |
|          Databricks          | Target |     |          |  1   |
|             Solr             | Target |     |          |  1   |
|           SAP HANA           | Source |     |          |  2   |
|           SAP HANA           | Target |     |          |  1   |
|       Google BigQuery        | Target |     |          |  2   |
|     SAP Adaptive Server      | Source |     |          |  2   |
|     SAP Adaptive Server      | Target |     |          |  1   |
|            HBase             | Source |     |          |  1   |
|  Microsoft Azure Cosmos DB   | Source |     |          |  1   |
|  Microsoft Azure Cosmos DB   | Target |     |          |  1   |
|           PostGIS            | Source |     |          |  1   |
|           PostGIS            | Target |     |          |  1   |
|           InfluxDB           | Target |     |          |  1   |
|          Couchbase           | Target |     |          |  1   |
|          Vika           | Target |     |          |  1   |
