# 导入 HBase 数据

本文以一个示例说明如何使用 Exchange 将存储在 HBase 上的数据导入 Nebula Graph。

## 数据集

本文以 [basketballplayer 数据集](https://docs-cdn.nebula-graph.com.cn/dataset/dataset.zip)为例。

在本示例中，该数据集已经存入 HBase 中，以`player`、`team`、`follow`和`serve`四个表存储了所有点和边的信息。以下为各个表的部分数据。

```sql
hbase(main):002:0> scan "player"
ROW                                COLUMN+CELL
 player100                         column=cf:age, timestamp=1618881347530, value=42
 player100                         column=cf:name, timestamp=1618881354604, value=Tim Duncan
 player101                         column=cf:age, timestamp=1618881369124, value=36
 player101                         column=cf:name, timestamp=1618881379102, value=Tony Parker
 player102                         column=cf:age, timestamp=1618881386987, value=33
 player102                         column=cf:name, timestamp=1618881393370, value=LaMarcus Aldridge
 player103                         column=cf:age, timestamp=1618881402002, value=32
 player103                         column=cf:name, timestamp=1618881407882, value=Rudy Gay
 ...

hbase(main):003:0> scan "team"
ROW                                COLUMN+CELL
 team200                           column=cf:name, timestamp=1618881445563, value=Warriors
 team201                           column=cf:name, timestamp=1618881453636, value=Nuggets
 ...

hbase(main):004:0> scan "follow"
ROW                                COLUMN+CELL
 player100                         column=cf:degree, timestamp=1618881804853, value=95
 player100                         column=cf:dst_player, timestamp=1618881791522, value=player101
 player101                         column=cf:degree, timestamp=1618881824685, value=90
 player101                         column=cf:dst_player, timestamp=1618881816042, value=player102
 ...

hbase(main):005:0> scan "serve"
ROW                                COLUMN+CELL
 player100                         column=cf:end_year, timestamp=1618881899333, value=2016
 player100                         column=cf:start_year, timestamp=1618881890117, value=1997
 player100                         column=cf:teamid, timestamp=1618881875739, value=team204
 ...
```

## 环境配置

本文示例在 MacOS 下完成，以下是相关的环境配置信息：

- 硬件规格：
  - CPU：1.7 GHz Quad-Core Intel Core i7
  - 内存：16 GB

- Spark：2.4.7，单机版

- Hadoop：2.9.2，伪分布式部署

- HBase：2.2.7

- Nebula Graph：{{nebula.release}}。使用 [Docker Compose 部署](../../4.deployment-and-installation/2.compile-and-install-nebula-graph/3.deploy-nebula-graph-with-docker-compose.md)。

## 前提条件

开始导入数据之前，用户需要确认以下信息：

- 已经[安装部署 Nebula Graph](../../4.deployment-and-installation/2.compile-and-install-nebula-graph/2.install-nebula-graph-by-rpm-or-deb.md) 并获取如下信息：

  - Graph 服务和 Meta 服务的的 IP 地址和端口。

  - 拥有 Nebula Graph 写权限的用户名和密码。

- 已经编译 Exchange。详情请参见[编译 Exchange](../ex-ug-compile.md)。本示例中使用 Exchange {{exchange.release}}。

- 已经安装 Spark。

- 了解 Nebula Graph 中创建 Schema 的信息，包括 Tag 和 Edge type 的名称、属性等。

- 已经安装并开启 Hadoop 服务。

## 操作步骤

### 步骤 1：在 Nebula Graph 中创建 Schema

分析数据，按以下步骤在 Nebula Graph 中创建 Schema：

1. 确认 Schema 要素。Nebula Graph 中的 Schema 要素如下表所示。

    | 要素  | 名称 | 属性 |
    | :--- | :--- | :--- |
    | Tag | `player` | `name string, age int` |
    | Tag | `team` | `name string` |
    | Edge Type | `follow` | `degree int` |
    | Edge Type | `serve` | `start_year int, end_year int` |

2. 在 Nebula Graph 中创建一个图空间** basketballplayer**，并创建一个 Schema，如下所示。

    ```ngql
    ## 创建图空间
    nebula> CREATE SPACE basketballplayer \
            (partition_num = 10, \
            replica_factor = 1, \
            vid_type = FIXED_STRING(30));
    
    ## 选择图空间 basketballplayer
    nebula> USE basketballplayer;
    
    ## 创建 Tag player
    nebula> CREATE TAG player(name string, age int);
    
    ## 创建 Tag team
    nebula> CREATE TAG team(name string);
    
    ## 创建 Edge type follow
    nebula> CREATE EDGE follow(degree int);

    ## 创建 Edge type serve
    nebula> CREATE EDGE serve(start_year int, end_year int);
    ```

更多信息，请参见[快速开始](../../2.quick-start/1.quick-start-workflow.md)。

### 步骤 2：修改配置文件

编译 Exchange 后，复制`target/classes/application.conf`文件设置 HBase 数据源相关的配置。在本示例中，复制的文件名为`hbase_application.conf`。各个配置项的详细说明请参见[配置说明](../parameter-reference/ex-ug-parameter.md)。

```conf
{
  # Spark 相关配置
  spark: {
    app: {
      name: Nebula Exchange {{exchange.release}}
    }
    driver: {
      cores: 1
      maxResultSize: 1G
    }
    cores {
      max: 16
    }
  }

  # Nebula Graph 相关配置
  nebula: {
    address:{
      # 以下为 Nebula Graph 的 Graph 服务和 Meta 服务所在机器的 IP 地址及端口。
      # 如果有多个地址，格式为 "ip1:port","ip2:port","ip3:port"。
      # 不同地址之间以英文逗号 (,) 隔开。
      graph:["127.0.0.1:9669"]
      meta:["127.0.0.1:9559"]
    }
    # 填写的账号必须拥有 Nebula Graph 相应图空间的写数据权限。
    user: root
    pswd: nebula
    # 填写 Nebula Graph 中需要写入数据的图空间名称。
    space: basketballplayer
    connection {
      timeout: 3000
      retry: 3
    }
    execution {
      retry: 3
    }
    error: {
      max: 32
      output: /tmp/errors
    }
    rate: {
      limit: 1024
      timeout: 1000
    }
  }
  # 处理点
  tags: [
    # 设置 Tag player 相关信息。
    # 如果需要将 rowkey 设置为数据源，请填写“rowkey”, 列族内的列请填写实际列名。
    {
      # Nebula Graph 中对应的 Tag 名称。
      name: player
      type: {
        # 指定数据源文件格式，设置为 HBase。
        source: hbase
        # 指定如何将点数据导入 Nebula Graph：Client 或 SST。
        sink: client
      }
      host:192.168.*.*
      port:2181
      table:"player"
      columnFamily:"cf"

      # 在 fields 里指定 player 表中的列名称，其对应的 value 会作为 Nebula Graph 中指定属性。
      # fields 和 nebula.fields 里的配置必须一一对应。
      # 如果需要指定多个列名称，用英文逗号（,）隔开。
      fields: [age,name]
      nebula.fields: [age,name]

      # 指定表中某一列数据为 Nebula Graph 中点 VID 的来源。
      # 例如 rowkey 作为 VID 的来源，请填写“rowkey”。
      vertex:{
          field:rowkey
      }

      # 单批次写入 Nebula Graph 的数据条数。
      batch: 256

      # Spark 分区数量
      partition: 32
    }
    # 设置 Tag team 相关信息。
    {
      name: team
      type: {
        source: hbase
        sink: client
      }
      host:192.168.*.*
      port:2181
      table:"team"
      columnFamily:"cf"
      fields: [name]
      nebula.fields: [name]
      vertex:{
          field:rowkey
      }
      batch: 256
      partition: 32
    }

  ]

  # 处理边数据
  edges: [
    # 设置 Edge type follow 相关信息
    {
      # Nebula Graph 中对应的 Edge type 名称。
      name: follow

      type: {
        # 指定数据源文件格式，设置为 HBase。
        source: hbase

        # 指定边数据导入 Nebula Graph 的方式，
        # 指定如何将点数据导入 Nebula Graph：Client 或 SST。
        sink: client
      }

      host:192.168.*.*
      port:2181
      table:"follow"
      columnFamily:"cf"

      # 在 fields 里指定 follow 表中的列名称，其对应的 value 会作为 Nebula Graph 中指定属性。
      # fields 和 nebula.fields 里的配置必须一一对应。
      # 如果需要指定多个列名称，用英文逗号（,）隔开。
      fields: [degree]
      nebula.fields: [degree]

      # 在 source 里，将 follow 表中某一列作为边的起始点数据源。示例使用 rowkey。
      # 在 target 里，将 follow 表中某一列作为边的目的点数据源。示例使用列 dst_player。
      source:{
          field:rowkey
      }

      target:{
          field:dst_player
      }

      # 单批次写入 Nebula Graph 的数据条数。
      batch: 256

      # Spark 分区数量
      partition: 32
    }

    # 设置 Edge type serve 相关信息
    {
      name: serve
      type: {
        source: hbase
        sink: client
      }
      host:192.168.*.*
      port:2181
      table:"serve"
      columnFamily:"cf"

      fields: [start_year,end_year]
      nebula.fields: [start_year,end_year]
      source:{
          field:rowkey
      }

      target:{
          field:teamid
      }

      batch: 256
      partition: 32
    }
  ]
}
```

### 步骤 3：向 Nebula Graph 导入数据

运行如下命令将 HBase 数据导入到 Nebula Graph 中。关于参数的说明，请参见[导入命令参数](../parameter-reference/ex-ug-para-import-command.md)。

```bash
${SPARK_HOME}/bin/spark-submit --master "local" --class com.vesoft.nebula.exchange.Exchange <nebula-exchange-{{exchange.release}}.jar_path> -c <hbase_application.conf_path>
```

!!! note

    JAR 包有两种获取方式：[自行编译](../ex-ug-compile.md)或者从 maven 仓库下载。

示例：

```bash
${SPARK_HOME}/bin/spark-submit  --master "local" --class com.vesoft.nebula.exchange.Exchange  /root/nebula-exchange/nebula-exchange/target/nebula-exchange-{{exchange.release}}.jar  -c /root/nebula-exchange/nebula-exchange/target/classes/hbase_application.conf
```

用户可以在返回信息中搜索`batchSuccess.<tag_name/edge_name>`，确认成功的数量。例如`batchSuccess.follow: 300`。

### 步骤 4：（可选）验证数据

用户可以在 Nebula Graph 客户端（例如 Nebula Graph Studio）中执行查询语句，确认数据是否已导入。例如：

```ngql
GO FROM "player100" OVER follow;
```

用户也可以使用命令 [`SHOW STATS`](../../3.ngql-guide/7.general-query-statements/6.show/14.show-stats.md) 查看统计数据。

### 步骤 5：（如有）在 Nebula Graph 中重建索引

导入数据后，用户可以在 Nebula Graph 中重新创建并重建索引。详情请参见[索引介绍](../../3.ngql-guide/14.native-index-statements/README.md)。