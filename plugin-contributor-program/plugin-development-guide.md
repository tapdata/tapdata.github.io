## **Tapdata数据源开发测试指南**

- 开源代码库：https://github.com/tapdata/tapdata

### **一、数据源调研**
接入新的数据源，建议最好对此数据源有过一定的调研和了解，便于提升开发速度，以及减少 Bug 的出现。以下需要特别关注，可能会颠覆后续的开发输入：

#### **1. 数据源的版本**
版本信息的重要性不言而喻，不同的版本使用的广度各不相同，各方面兼容的层面也有区别。因此在挑选数据源版本时，按照使用范围广、Java兼容性好、高版本为优先级。
#### **2. 数据源对 Java 的适配兼容程度**
市面上绝大多数的数据源对 Java 适配都友好，但是根据数据源版本的不同，Jar 包的对应版本也随之变化，笔者推荐兼容性较高的 Jar 包。目前 PDK 整体框架是将 Connector 作为 Plugin 热加载的，已经按照 ClassLoader 作了类隔离，因此不同版本的 Jar 包却有冲突类的不能放在同一个 Connector下。这点直接影响到了新数据源适配到了哪些版本，Connector需要拆解为几个。
#### **3. 数据源的数据模型适配**
目前 PDK 的模型主要是 database—(schema)—table—column，针对上述模型，新数据源需要向它靠拢。尤其针对非关系型数据源，确定合理的模型边界非常重要。
- 以消息队列举例，database 可能就是一个消息队列服务，table 指的是 queue 或者 topic，column信息就需要解析 topic 或者 queue 的部分数据（可能是 json，xml，jsonbyte 等）来大致明确。
- 还有 Redis 这类 KV 数据源，table 可以是一个 key，table 也可以是同一个前缀的所有 key集合，模型的边界需要开发去确定，在能力范围内，甚至应该将它们配置化。
#### **4. 数据源的部署**
部署虽然不在开发的范畴，但是在贡献数据源的时候，是非常重要的环节，有相当的时间成本。首先在开发完成进行自测时，这关必不可少（有现成的环境的除外），笔者比较推荐用 docker 镜像去部署数据源环境。其次，在开发前期需要了解到该数据源有哪些认证和部署方案，最常见的可能有各种连接认证方式、单例集群主从等部署方案，这些开发者都需要按页面不同的配置方式书写对应的代码逻辑，很影响贡献数据源的覆盖能力，是很重要的评估加分指标。
#### **5. 数据源的数据字段类型**
数据源的核心能力还是数据本身，那么组成这些数据的数据类型就很关键：
- 光从关系型数据库来说，不同的数据源有五花八门的数据字段类型，每种类型都有长度，精度，边界，结构等属性。另外需要试着调研到数据源底层，它真正字段类型是什么，哪些是衍化而来的新类型，哪些只是同义词，开发中对于异构数据源的字段类型映射非常重要。
- 对于弱schema的数据源来说，字段类型相对较少，只有常用的String，Integer，Double，Date等，但是这些往往需要与Java中的常用数据类型作对应，这样才能提升这些数据源向关系型数据源转化时，数据类型覆盖得精准性。
  后面字段类型映射的json中会详细介绍，不赘述
#### **6. 数据源的独有特性**
美其名数据源的独有特性，其实就是数据源的各种踩坑点，这些给开发上增加很多阻力。开发者最好可以在md文档中给出说明，让用户能提前了解它的特性，提升体验感。下面举例说明：
- postgres数据库在无主键的情况，更新和删除会有异常，需要open Identity。
- sqlServer，Db2等数据源在做数据CDC时，需要提前对表执行启用CDC命令。
- 部分分析型国产数据库不支持事务，或者不支持主键约束等等。

### **二、数据源模块开发（ ⭐️ 标记难度）**

#### **1. 模块基础结构**
<img src="https://user-images.githubusercontent.com/25000091/192274495-5e5f185f-580d-4847-8dca-d5a21b2a297b.png" width="300px"/>

假如新数据源昵称：abcde
- 在 Connectors 模块下新建 Maven 子模块 abcde-connector，模块基础结构如下（如图）：
- 模块根目录编辑 pom.xml
- 新建 java 包路径 io.tapdata.connector.abcde，新建核心类 AbcdeConnector
- 资源文件（以下为推荐名）：
  resources/docs/abcde_en_US.md——英文说明文档
  resources/docs/abcde_zh_CN.md——中文简体说明文档
  resources/docs/abcde_zh_TW.md——中文繁体说明文档
  resources/icons/abcde.png——数据源图标（64*64）
  resources/spec_abcde.json——数据源配置及字段映射Json文件

#### **2. ⭐️ pom.xml 示例（仅需修改划横线的内容，留意log4j的Jar包冲突）**

```
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>connectors</artifactId>
        <groupId>io.tapdata</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <name>Abcde Connector</name>
    <artifactId>abcde-connector</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <properties>
        <java.version>8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>io.tapdata</groupId>
            <artifactId>tapdata-pdk-api</artifactId>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>io.tapdata</groupId>
            <artifactId>tapdata-pdk-runner</artifactId>
            <scope>test</scope>
        </dependency>
        <!--添加新数据源开发需要的依赖-->
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.1.1</version>

                <configuration>
                    <finalName>${connector.file.name}</finalName>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                    <archive>
                        <manifestEntries>
                            <Implementation-Title>${project.artifactId}</Implementation-Title>
                            <Implementation-Version>${project.version}</Implementation-Version>
                            <Implementation-Vendor>${project.groupId}</Implementation-Vendor>

                            <Build-OS>${os.name} ${os.version}</Build-OS>
                            <Build-Java>Java ${java.version}</Build-Java>
                            <PDK-Runner-Version>${tapdata.pdk.runner.version}</PDK-Runner-Version>
                            <PDK-API-Version>${tapdata.pdk.api.version}</PDK-API-Version>
                            <Tapdata-API-Version>${tapdata.api.version}</Tapdata-API-Version>
                            <Version>${project.version}</Version>
                            <Git-Build-Time>${git.build.time}</Git-Build-Time>
                            <Git-Branch>${git.branch}</Git-Branch>
                            <Git-Commit-Id>${git.commit.id}</Git-Commit-Id>
                            <Git-Build-User-Name>${git.build.user.name}</Git-Build-User-Name>
                            <Git-Build-User-Email>${git.build.user.email}</Git-Build-User-Email>
                        </manifestEntries>
                    </archive>
                    <appendAssemblyId>false</appendAssemblyId>
                </configuration>

                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>

            </plugin>
            <plugin>
                <groupId>pl.project13.maven</groupId>
                <artifactId>git-commit-id-plugin</artifactId>
                <version>2.2.3</version>
                <executions>
                    <execution>
                        <id>get-the-git-infos</id>
                        <goals>
                            <goal>revision</goal>
                        </goals>
                        <phase>validate</phase>
                    </execution>
                </executions>
                <configuration>
                    <verbose>true</verbose>
                    <generateGitPropertiesFile>true</generateGitPropertiesFile>
                    <failOnNoGitDirectory>true</failOnNoGitDirectory>
                    <injectAllReactorProjects>true</injectAllReactorProjects>
                    <dotGitDirectory>${project.basedir}/../../.git</dotGitDirectory>
                </configuration>
            </plugin>
            <plugin>
                <artifactId>maven-resources-plugin</artifactId>
                <version>3.1.0</version>
                <executions>
                    <execution>
                        <id>copy-resource-one</id>
                        <phase>package</phase>
                        <goals>
                            <goal>copy-resources</goal>
                        </goals>

                        <configuration>
                            <outputDirectory>../dist</outputDirectory>
                            <resources>
                                <resource>
                                    <directory>${basedir}/target/</directory>
                                    <includes>
                                        <include>${connector.file.name}.jar</include>
                                    </includes>
                                </resource>
                            </resources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.felix</groupId>
                <artifactId>maven-bundle-plugin</artifactId>
                <version>4.2.1</version>
                <extensions>true</extensions>
            </plugin>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.7.0</version>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

前四个资源文件，仅只是展示功能，按要求提供即可。

接下来最重要也是开发最关键的两个核心文件即Json文件与AbcdeConnector.java文件
#### **3. ⭐️⭐️⭐️⭐️ Json文件示例（详细可以参见代码，完成优秀的Json配置，开发成功一半）**

<img src="https://user-images.githubusercontent.com/25000091/192289584-5443615b-b1dd-48fb-a0fb-2e2067712a81.png" width="300px"/>

- properties.name——数据源在前端页面上显示的名称
- properties.icon——数据源的图标路径
- properties.doc——说明文档路径
- properties.id——需要与abcde-connector的前缀对上（abcde）
- configOptions.connection——提供到页面的连接配置
- configOptions.node——提供到页面的任务节点配置
- configOptions.capabilities——数据源能提供的写入策略和更新策略（不强制要求实现）
- connection&node的参数都按前端低代码标准

>"required": true //是否必填<br>
"default": xxxx //默认值<br>
"type": "string" //固定写法<br>
"x-decorator": "FormItem" //固定写法<br>
"title": "${param}" //param是参数名<br>
"x-component": "Input" //前端组件的类型<br>
"x-index": 1 //参数在页面的顺序<br>
"x-reactions": [] //配置参数联动

前端组件包括：
- Input：文本输入框
- InputNumber：数字输入框
- Password：密码框
- Select：下拉选择框（配合enum）
- Radio.Group：选择按钮组（配合enum）
- Switch：开关
- Text：文本提示（文本提示）
- TextFileReader：文件选择框（配合"x-component-props": {"base64": true}使用）

下面每类参数提供一个案例：
```
"connection": {
  "type": "object",
  "properties": {
    "host": {
      "required": true,
      "type": "string",
      "title": "${host}",
      "x-decorator": "FormItem",
      "x-component": "Input",
      "apiServerKey": "database_host",
      "x-index": 1
    },
    "port": {
      "required": true,
      "type": "string",
      "title": "${port}",
      "x-decorator": "FormItem",
      "x-component": "InputNumber",
      "apiServerKey": "database_port",
      "x-index": 2
    },
    "password": {
      "type": "string",
      "title": "${password}",
      "x-decorator": "FormItem",
      "x-component": "Password",
      "apiServerKey": "database_password",
      "x-index": 3
    },
    "logPluginName": {
      "required": true,
      "type": "string",
      "title": "${logPluginName}",
      "default": "PGOUTPUT",
      "x-decorator": "FormItem",
      "x-component": "Select",
      "apiServerKey": "logPlugin",
      "x-index": 4,
      "enum": [
        {
          "label": "DECODERBUFS",
          "value": "decoderbufs"
        },
        {
          "label": "WAL2JSON",
          "value": "wal2json"
        },
        {
          "label": "WAL2JSONRDS",
          "value": "wal2json_rds"
        },
        {
          "label": "WAL2JSONSTREMING",
          "value": "wal2json_streaming"
        },
        {
          "label": "WAL2JSONRDSSTREAMING",
          "value": "wal2json_rds_streaming"
        },
        {
          "label": "PGOUTPUT",
          "value": "pgoutput"
        }
      ]
    },
    "thinType": {
      "type": "string",
      "title": "${thinType}",
      "enum": [
        {
          "label": "SID",
          "value": "SID",
          "disabled": false
        },
        {
          "label": "SERVICE NAME",
          "value": "SERVICE_NAME",
          "disabled": false
        }
      ],
      "default": "SID",
      "required": true,
      "x-decorator": "FormItem",
      "x-component": "Radio.Group",
      "x-component-props": {
        "optionType": "button"
      },
      "x-reactions": [
        {
          "target": "*(host,port)",
          "fulfill": {
            "state": {
              "visible": "{{$self.value==='SID' || $self.value==='SERVICE_NAME'}}"
            }
          }
        },
        {
          "target": "*(sid)",
          "fulfill": {
            "state": {
              "visible": "{{$self.value==='SID'}}"
            }
          }
        },
        {
          "target": "*(database)",
          "fulfill": {
            "state": {
              "visible": "{{$self.value==='SERVICE_NAME'}}"
            }
          }
        },
        {
          "target": "*(tnsName)",
          "fulfill": {
            "state": {
              "visible": "{{$self.value==='TNS_NAME'}}"
            }
          }
        }
      ]
    },
    "standBy": {
      "type": "boolean",
      "title": "${standBy}",
      "default": false,
      "x-decorator": "FormItem",
      "x-component": "Switch",
      "x-index": 16
    },
    "ssl_tips1": {
      "type": "void",
      "title": " ",
      "x-decorator": "FormItem",
      "x-decorator-props": {
        "colon": false
      },
      "x-component": "Text",
      "x-component-props": {
        "content": "${ssl_tips1}"
      },
      "x-reactions": {
        "dependencies": [
          "ssl"
        ],
        "fulfill": {
          "schema": {
            "x-decorator-props.style.display": "{{$deps[0]===true ? null:\"none\"}}"
          }
        }
      },
      "x-index": 81
    },
    "sslKey": {
      "type": "string",
      "title": "${dataForm_form_sslKey}",
      "x-decorator": "FormItem",
      "x-component": "TextFileReader",
      "fileNameField": "sslKeyFile",
      "required": true,
      "x-index": 90
    }
  }
}
```

message——Json中所有变量的国际化（看代码样例就明白）<br>
dataType——数据源具备的数据类型<br>
dataType详细使用说明可以见以下官网：<br>
https://tapdata.github.io/docs/Connectors/docs/data-type-expressions.html<br>
这里简单介绍下：
- 通用规则
  "queryOnly": true （在目标数据源不支持创建的数据类型）
  "pkEnablement": false （作为目标数据源主键时不出现）
  中括号[]表示内部可选或数值范围
- 字符串类型（TapString）
  $byte：最大字节数（支持k,m,g,t等）
  bitRatio：一个中文占字节数（不设置默认为1）
  defaultByte：默认字节数
  preferByte：相对于最大字节数更普遍适用的字节数
  fixed：若为true，会补齐空格
- 数值类型（TapNumber）
  $precision：数值长度
  $scale：数值精度
  defaultPrecision：默认长度
  defaultScale：默认精度
  preferPrecision：适用长度
  preferScale：适用精度
  value：数值范围（支持科学计数）
  fixed：若为true，表示双精度
- 日期时间类型（TapDateTime、TapDate、TapTime）
  $fraction：时间精度
  defaultFraction：默认精度
  range：时间或者日期范围
  pattern：上述范围的java格式
  withTimeZone：带时区
- 二进制字节类型（TapBinary）
- 布尔类型（TapBoolean）
- 数据结构类型（TapArray、TapMap）
- 无法适配（TapRaw）

特别注意：dataType的字段类型映射推演不是百分百精准的，有一些数据源它对字段类型规则定义不同，还是需要 Connector 代码层的能力去干预，如果还有问题，就需要用户人为在页面上配置了。

#### **4. Connector 的 Java 开发**
##### **4.1 开发流程及注意事项**
```
package io.tapdata.connector.abcde;

import io.tapdata.base.ConnectorBase;
import io.tapdata.entity.codec.TapCodecsRegistry;
import io.tapdata.entity.schema.TapTable;
import io.tapdata.pdk.apis.annotations.TapConnectorClass;
import io.tapdata.pdk.apis.context.TapConnectionContext;
import io.tapdata.pdk.apis.entity.ConnectionOptions;
import io.tapdata.pdk.apis.entity.TestItem;
import io.tapdata.pdk.apis.functions.ConnectorFunctions;

import java.util.List;
import java.util.function.Consumer;

@TapConnectorClass("spec_abcde.json")
public class AbcdeConnector extends ConnectorBase {

    @Override
    public void onStart(TapConnectionContext connectionContext) throws Throwable {

    }

    @Override
    public void onStop(TapConnectionContext connectionContext) throws Throwable {

    }

    @Override
    public void registerCapabilities(ConnectorFunctions connectorFunctions, TapCodecsRegistry codecRegistry) {

    }

    @Override
    public void discoverSchema(TapConnectionContext connectionContext, List<String> tables, int tableSize, Consumer<List<TapTable>> consumer) throws Throwable {

    }

    @Override
    public ConnectionOptions connectionTest(TapConnectionContext connectionContext, Consumer<TestItem> consumer) throws Throwable {
        return null;
    }

    @Override
    public int tableCount(TapConnectionContext connectionContext) throws Throwable {
        return 0;
    }
}
```

- Connector 必须继承 ConnectorBase 覆盖上述方法，@TapConnectorClass 注解上面的json文件
- ⭐️ Connector的生命周期由onStart方法开始、onStop方法结束。
    - onStart一般需要初始化全局变量，初始化资源，比如数据源连接，线程池，自定义的工具类；
    - onStop需要去释放全局引用的资源，比如数据源连接，线程池，自定义的工具类；
- ⭐️⭐️⭐️ ConnectionTest（连接测试）是唯一比较独立的，不包含在onStart和onStop的周期中，需要开辟自己的资源。
```
@Override
public ConnectionOptions connectionTest(TapConnectionContext connectionContext, Consumer<TestItem> consumer) throws Throwable {
    DataMap configInfo = connectionContext.getConnectionConfig();
    ConnectionOptions connectionOptions = ConnectionOptions.create();
    connectionOptions.connectionString("192.168.1.1/aaa/bbb");
    AbcdeTest abcdeTest = new AbcdeTest();
    TestItem firstItem = abcdeTest.testFirst();
    consumer.accept(firstItem);
    TestItem secondItem = abcdeTest.testSecond();
    consumer.accept(secondItem);
    TestItem thirdItem = abcdeTest.testThird();
    consumer.accept(thirdItem);
    return connectionOptions;
}
```

json配置的连接参数，可以通过connectionContext.getConnectionConfig()获取；最终生成的连接在页面展示的连接信息需要 connectionOptions.connectionString 来设置；这些测试项可以递进式测试，也可以并发，看开发者的实现，测试有成功、失败、成功带警告三种类型，带警告的需要提供告警信息。
- ⭐️ tableCount  主要是查询当前连接配置下，table 或模拟 table 的数量，如果它返回小于1，连接测试会有提示告警，并且不会调用加载模型 discoverSchema。
- ⭐️⭐️⭐️⭐️ discoverSchema（加载模型）：
  前期调研数据源时，很重要的一项就是怎么去查询这个数据源特定配置下所有 table 或模拟 table 的详情，强 schema 数据源：table 详情在 table 层级需要查询所有的table名称和注释；在 column 层级需要查询 column 名称，dataType 数据类型（需要完整的类似 varchar(100)，且符合 Json 中定义的规则），nullable 可否为空，primaryKey 是否主键字段，defaultValue 默认值，column 注释等；在 index 层级需要查询 index 名称，isPrimary 是否主键索引，isUnique 是否唯一索引，fieldList 索引字段列表，甚至索引正序反序。将上述信息统统封装到 TapTable 对象里，然后分页提交。
- 弱 schema 数据源：column的信息需要sample一些数据去解析，如果暂时没有数据，可以只提供table名称，忽略column。

##### **4.2 discoverSchema 的实现需要关注加载性能问题**
- ⭐️⭐️⭐️ registerCapabilities（数据源能力注册）：
```
@Override
public void registerCapabilities(ConnectorFunctions connectorFunctions, TapCodecsRegistry codecRegistry) {
    //test
    connectorFunctions.supportConnectionCheckFunction(this::checkConnection);
    //target
    connectorFunctions.supportWriteRecord(this::writeRecord);
    connectorFunctions.supportCreateTableV2(this::createTableV2);
    connectorFunctions.supportClearTable(this::clearTable);
    connectorFunctions.supportDropTable(this::dropTable);
    //source
    connectorFunctions.supportBatchCount(this::batchCount);
    connectorFunctions.supportBatchRead(this::batchReadV2);
    connectorFunctions.supportStreamRead(this::streamRead);
    connectorFunctions.supportTimestampToStreamOffset(this::timestampToStreamOffset);
    //query
    connectorFunctions.supportQueryByFilter(this::queryByFilter);
    connectorFunctions.supportQueryByAdvanceFilter(this::queryByAdvanceFilter);
    connectorFunctions.supportGetTableNamesFunction(this::getTableNames);
    //ddl
    connectorFunctions.supportNewFieldFunction(this::fieldDDLHandler);
    connectorFunctions.supportAlterFieldNameFunction(this::fieldDDLHandler);
    connectorFunctions.supportAlterFieldAttributesFunction(this::fieldDDLHandler);
    connectorFunctions.supportDropFieldFunction(this::fieldDDLHandler);
    //connectorFunctions.supportCreateIndex(this::createIndex);

    codecRegistry.registerFromTapValue(TapRawValue.class, "CLOB", tapRawValue -> {
        if (tapRawValue != null && tapRawValue.getValue() != null) return tapRawValue.getValue().toString();
        return "null";
    });
    codecRegistry.registerFromTapValue(TapMapValue.class, "CLOB", tapMapValue -> {
        if (tapMapValue != null && tapMapValue.getValue() != null) return toJson(tapMapValue.getValue());
        return "null";
    });
    codecRegistry.registerFromTapValue(TapArrayValue.class, "CLOB", tapValue -> {
        if (tapValue != null && tapValue.getValue() != null) return toJson(tapValue.getValue());
        return "null";
    });
    codecRegistry.registerFromTapValue(TapBooleanValue.class, "INTEGER", tapValue -> {
        if (tapValue != null && tapValue.getValue() != null) return tapValue.getValue() ? 1 : 0;
        return 0;
    });

    codecRegistry.registerToTapValue(TIMESTAMP.class, (value, tapType) -> {
        try {
            return new TapDateTimeValue(new DateTime(((TIMESTAMP) value).timestampValue()));
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    });
    codecRegistry.registerToTapValue(TIMESTAMPTZ.class, (value, tapType) -> {
        try {
            return new TapDateTimeValue(new DateTime(((TIMESTAMPTZ) value).toZonedDateTime()));
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    });
    codecRegistry.registerToTapValue(TIMESTAMPLTZ.class, (value, tapType) -> {
        try {
            return new TapDateTimeValue(new DateTime(((TIMESTAMPLTZ) value).toLocalDateTime(oracleConnection)));
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    });
    codecRegistry.registerToTapValue(CLOB.class, (value, tapType) -> {
        try {
            return new TapStringValue(((CLOB) value).stringValue());
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    });
    codecRegistry.registerToTapValue(NCLOB.class, (value, tapType) -> {
        try {
            return new TapStringValue(((NCLOB) value).stringValue());
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    });
    codecRegistry.registerToTapValue(BLOB.class, (value, tapType) -> new TapBinaryValue(DbKit.blobToBytes((BLOB) value)));
    //TapTimeValue, TapDateTimeValue and TapDateValue's value is DateTime, need convert into Date object.
    codecRegistry.registerFromTapValue(TapTimeValue.class, "CHAR(8)", tapTimeValue -> formatTapDateTime(tapTimeValue.getValue(), "HH:mm:ss"));
    codecRegistry.registerFromTapValue(TapDateTimeValue.class, tapDateTimeValue -> tapDateTimeValue.getValue().toTimestamp());
    codecRegistry.registerFromTapValue(TapDateValue.class, "CHAR(10)", tapDateValue -> formatTapDateTime(tapDateValue.getValue(), "yyyy-MM-dd"));
}
```

> 能力注册以上面 Oracle 举例，难度不大，但是需要理解这个是做什么的。<br>
connectorFunctions.support 系列主要是这个数据源支持哪些 API 能力，这些 API 实现有难易，不同组合可以在页面上支撑不同类型的任务，至少需要实现一个能力，才能向 Tapdata 引擎注册 connector。（后面会继续详细介绍）<br>
引擎框架层面向TapType转换已经覆盖了绝大多数 Java 对象，但是还是会有部分从数据源中查询出来的生僻对象尤其jdbc包内第三方定义的类，我们需要用 codecRegistry.registerToTapValue 告诉框架，这个生僻对象希望转成哪种 TapType，并且将数据精确地转换。特别注意的是这个生僻对象需要它本身的类，不可以是它的超类，否则框架无法识别。<br>
引擎框架层面需要向目标数据源写入，但是事件中的数据对象都是 TapType 类型，其中有些已经在Json配置中定义，框架会按优先级去适配，转换成目标数据源可以接收的 Java 类型，但是还有一部分Json中并没有映射关系的，比如 TapMap，TapRaw，TapArray 这些不怎么会出现在 Json 中的类型，我们需要用 codecRegistry.registerFromTapValue 告诉框架这些 TapType 该怎么去写进目标数据源，以什么数据类型写。另外像 oracle 中，TapDateTime 类型是有 timestamp 或者 date 类型来适配的，但是写入时需要它转换成 to_date(...)方式，显然这种方式比较麻烦，于是就可以指明转换为 Timestamp 类型用 setObject 方式去写入。时间类型（TapDate，TapTime，TapDateTime）都需要codecRegistry.registerFromTapValue。<br>

- 下面逐个来分析实现数据源能力：
    - ⭐️ checkConnection：类似心跳监测，保证数据源的连接是否正常，实现上一定要简单高性能（非必须实现）
    - ⭐️⭐️ createTableV2：向目标数据源建表，通过给到的TapTable模型，构建一条建表 SQL 语句（和加载模型一样，column 需要考虑到可否为空，是否主键字段，默认值，注释等），或者给数据源增加一个符合模型的 table，如果同名 table 已经存在则跳过，且返回告诉框架表是否已经存在（作为目标写入且强schema数据源必须实现）。
    - ⭐️ clearTable：清空表，easy（目前没应用场景）
    - ⭐️ dropTable：删除表，easy（仅 TDD 测试依赖）
    - ⭐️⭐️⭐️⭐️ writeRecord：目标写入增删改核心 API，难点在于引擎上层会通过多线程来调用，一定要保证线程安全的同时，最大程度地提升性能；另外难点在写入有策略配置，虽然不强求但尽可能需要实现四种策略插入冲突时更新（默认）、插入冲突时忽略、未更新到时忽略（默认）、未更新到时插入。（目标写入必须实现）
    - ⭐️ batchCount：查询表总记录数，除了个别消息队列，easy（源全量读取统计需要）
    - ⭐️⭐️ batchRead：支持偏移量无限读取源表记录，并向引擎提交offset断点。目前超大数据量或无唯一约束的场景，全量断点开始读性能较差。（源全量读取必须实现）
    - ⭐️⭐️⭐️⭐️⭐️ streamRead：贡献数据源难点主要集中在这个API，也是接入该数据源的最大亮点，源增量读取，源端数据需要Change Data Capture实时（或极小时间差）写入目标数据源。开发streamRead首先要调研CDC方案，如果该数据源本身有一套成熟的API，可以大大降低难度。可以不依赖数据源运行直接解析日志的为最优方案，依赖数据源运行但不侵入改动源库源表的方案次之，最差的方案就是需要侵入数据源且损耗相当一部分数据源性能的。开发者可以按自身的能力选择合适的方案。（特别提醒：该API是需要加任务运行标志的阻塞性方法，引擎不中止任务，不会退出完成）（源增量读取必须实现）
    - ⭐️⭐️⭐️⭐️⭐️⭐️ 更加难的点就是在 streamRead 方法中除了 DML 的监听，还同步 DDL 的操作。由于绝大多数数据源对于 DDL 的日志都是简单的一句 SQL，需要把 SQL 解析为 DDLEvent；另外 SQL中包含用户随意写的各种数据类型同义词，且这些同义词在我们Json中未必会全部都定义到，所以再额外处理。（特别提醒：DDL 同步后 DML 需要正常运行，所以有些全局信息需要动态更新）
    - ⭐️⭐️ timestampToStreamOffset：为了 streamRead 断点服务，offset 时间若为空，取当前时间的断点，否则按 offset 时间取增量读的断点。某些数据源增量读方案不支持指定时间返回断点的比如Postgres，需要设计方案将 streamRead 的部分逻辑（创建Slot）前置到 batchRead。
    - ⭐️ queryByFilter：一般是按主键的精确查询（仅TDD测试依赖）
    - ⭐️⭐️ queryByAdvanceFilter：更多条件的批量高级查询（数据校验依赖）
    - ⭐️ getTableNames：查询所有表名称，页面上的表包含功能（通过模糊匹配）依赖
    - ⭐️⭐️⭐️ createIndex：创建表索引，有同名需要跳过，部分数据源不支持无索引名（非必须实现）
    - ⭐️⭐️ newField、alterFieldAttr、alterFieldName、dropField：（作为目标写入DDL需要实现）
    -
下面附上任务对应API列表，完成这些API后，开发的任务就告一段落了，进入测试阶段。


|   板块    |       能力        |           API昵称           | 等级  | 依赖  |      难度      |
|:-------:|:---------------:|:-------------------------:|:---:|:---:|:------------:|
|  基本功能   |  基本配置与连接测试（全面）  |    Json、connectionTest    |  1  | 强依赖 |     ⭐️⭐️     |
|  基本功能   |  数据源字段类型定义（精确）  | Json、registerCapabilities |  2  | 强依赖 |    ⭐️⭐️⭐️    |
|  基本功能   |   数据源模型加载（性能）   |      discoverSchema       |  3  | 强依赖 |    ⭐️⭐️⭐️    |
| 多条件数据过滤 |      数据校验       |   queryByAdvanceFilter    |  4  | 		  |      ⭐️      |
| 目标数据写入  |   目标数据源新建表模型    |       createTableV2       |  5  |  	  |     ⭐️⭐️     |
| 目标数据写入  |   目标数据源写入（性能）   |        writeRecord        |  6  |  	  |     ⭐️⭐️     |
| 目标数据写入  |  目标数据源写入策略（性能）  | insertPolicy、updatePolicy |  7  |  6  |   ⭐️⭐️⭐️⭐️   |
| 目标数据写入  |   目标数据源DDL写入    |      fieldDDLHandler      |  8  |  6  |     ⭐️⭐️     |
| 源数据全量读取 |     源表数据总量      |        batchCount         |  9  | 		  |     ⭐️⭐️     |
| 源数据全量读取 |    源表数据批量读取     |         batchRead         | 10  | 		  |      ⭐️      |
| 源数据增量读取 | 源表数据实时增量读取（CDC） |        streamRead         | 11  | 10  |  ⭐️⭐️⭐️⭐️⭐️  |
| 源数据增量读取 |  源表数据指定时间戳增量读取  |  timestampToStreamOffset  | 12  | 11  |     ⭐️⭐️     |
| 源数据增量读取 |   源表数据DDL增量读取   |       streamRead++        | 13  | 11  | ⭐️⭐️⭐️⭐️⭐️⭐️ |

### **三、新数据源测试**
#### **1. TDD集成自测**
TDD测试可以在plugin-kit下子模块tapdata-pdk-cli进行：<br>
TDD自测的前提是本地有部署新数据源：首先资源文件resources/config下新增abcde.json，按部署的新数据源配置原来Json中需要的那些参数，如下面json所示：<br>
```
{
  "connection": {
    "host": "192.168.1.1",
    "port": 5432,
    "database": "database",
    "schema": "schema",
    "user": "user",
    "password": "pass"
  }
}
```
然后java中新建Main方法：
```
package io.tapdata.pdk.cli;

import io.tapdata.pdk.core.utils.CommonUtils;
import picocli.CommandLine;

public class TDDAbcdeMain {
    public static void main(String... args) {
        CommonUtils.setProperty("pdk_external_jar_path", "./connectors/dist");
        args = new String[]{
                "test", "-c", "plugin-kit/tapdata-pdk-cli/src/main/resources/config/abcde.json",
//                "-t", "io.tapdata.pdk.tdd.tests.source.StreamReadTest",
//                "-t", "io.tapdata.pdk.tdd.tests.source.BatchReadTest",
//                "-t", "io.tapdata.pdk.tdd.tests.target.CreateTableTest",
//                "-t", "io.tapdata.pdk.tdd.tests.target.DMLTest",
                "connectors/abcde-connector",};
      Main.registerCommands().execute(args);
    }
}
```
上述方法可以全量测试，可以针对CreateTable、DML、BatchRead、StreamRead分别单测<br>
建议debug模式多走几遍，也欢迎开发者补充强壮TDD测试的用例和范围

#### **2. 线上联调自测**
有条件的小伙伴可以去下载Tapdata的docker镜像，还是建议在本地部署运行，GitHub上都有相关资料，更容易debug
- Connector开发完成后可以Maven直接打出Jar包，connectors/dist目录下存在了abcde-connector-v1.0-SNAPSHOT.jar文件
- 代码中找到RegisterMain，将新数据源补充至ConnectorEnums枚举
>Abcde(BASE_PATH + "connectors/dist/abcde-connector-v1.0-SNAPSHOT.jar", "all", "xxxxx"),

并将下面的Server地址换成线上联调地址，结尾不要带‘/’
>String server = System.getProperty("server", "http://localhost:3000");

屏蔽其它数据源的枚举，运行main方法，提示successful。
至此，新数据源就已经注册成功！
- 可以从连接配置——测试连接——加载模型——数据开发（分别作为目标作为源）去测试基础的数据同步功能。

#### **3. 代码合并**
- TDD 测试与线上联调自测已基本满足预期后，开发者可以通过分支合并提交 PR。
- Tapdata 数据源管理人员会尽快做简单的代码 review，通过线上联调环境对新数据源做简单的 P0冒烟测试，确认无明显功能缺陷后会提交测试伙伴正式进入测试流程。
- 测试基本通过后，代码会合并主分支，Tapdata 产研会协同评估该新数据源的完成质量和市场价值给予开发者丰厚的奖励。
#### **4. 其他**
- 测试过程中如果有 BUG，可以加群/加入Slack方便沟通。
    - Slack 加入链接：https://join.slack.com/t/tapdatacommunity/shared_invite/zt-1biraoxpf-NRTsap0YLlAp99PHIVC9eA
      <img src="https://user-images.githubusercontent.com/25000091/192302210-1aaae8b2-c79a-4b88-8026-09797b0a4520.png" width="300px"/>

