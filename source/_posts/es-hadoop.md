title: Elasticsearch & Hadoop
date: 2017-08-20 22:05:14
tags: [Elasticsearc, Hadoop]
---

最近有个在 [Hive](https://hive.apache.org/) 上面查询 [Elasticsearch](https://www.elastic.co/products/elasticsearch) 中数据的需求。

<!-- more -->

一开始计划是用 Elasticsearch 的 [snapshot/restore](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-snapshots.html) API的方式把数据备份到 `HDFS`，再从 `HDFS` 导数据到 `Hive` 中，如下图：

![es hdfs hive](/images/es_hdfs_hive.png)

为此用 [curator](https://github.com/elastic/curator) 准备了备份的 `action` 文件：

```yml
actions:
  1:
    description: "Daily snapshot indics"
    action: snapshot
    options:
      repository: mybackup
      name: backup_%Y-%m-%d
      wait_for_completion: True
      ignore_unavailable: True
    filters:
    - filtertype: age
      source: creation_date
      direction: old
      unit: days
      unit_count: 1
```

结果发现用 `snapshot` 的文件格式，并不能够很直接的解析出来。

![es snapshot](/images/es_snapshot.png)

`sqoop` 应该也还没有这个功能……

还好官方还有这个操作：https://www.elastic.co/products/hadoop
提供了最直接的整合方式。

按照文档的接入说明，总结下，接入 Hadoop 需要下面几步：
假设 Elasticsearch 中存在 index：test，type：test，mapping 为：

```json
{
  "test" : {
    "mappings" : {
      "test" : {
        "properties" : {
          "date" : {
            "type" : "date",
            "format" : "dateOptionalTime"
          },
          "host" : {
            "type" : "string"
          },
          "name" : {
            "type" : "string"
          }
        }
      }
    }
  }
}
```

### ES Hadoop

1. 下载 hadoop 对应的 es-hadoop jar 包，并上传到 hdfs 中；

2. 接着在 Hive 中创建一个 external table，如下：

```sql
ADD JAR hdfs:///lib/hive/udfs/elasticsearch-hadoop-5.5.1.jar;

CREATE EXTERNAL TABLE test (
    date date,
    name string,
    host int,
)
STORED BY 'org.elasticsearch.hadoop.hive.EsStorageHandler'
TBLPROPERTIES('es.resource' = 'test/test',
              'es.nodes' = 'elasticsearch:9200',
              'es.query'= '?q=*');
```

elasticsearch 和 hive 中的 data type，可以参考：
https://www.elastic.co/guide/en/elasticsearch/hadoop/current/hive.html#hive-type-conversion


3. 然后就可以在 Hive 中分析 Elasticsearch 的数据了。


### 一些 Tips

- 时间数据不兼容

Elasticsearch 中的 date 的格式，通常为 ISO 8601，通常这种格式，es-hadoop 是能够理解的，如果时间格式不为 ISO 8601，这种情况使用 hive 的时候，会报数据格式不兼容的错误，处理这样的情况，可以在 `TBLPROPERTIES` 中增加配置

```
'es.mapping.date.rich' = 'false'
```

- 数组中存在多个类型

如果遇到 elasticsearch 中的 array 中存储了不同类型的数据，在 hadoop 这样的数据类型严格，就会报数据类型的错误。目前 es_hadoop 还不支持 union 类型，为了避免这个错误，一是修正 Elasticsearch 的数据，二是通过：

```
`es.read.field.exclude` = 'fieldname1, fieldname2'
```

### 将 Elasticsearch 的数据归档到 Hive

接着上面的例子，上面创建了 external 表 test，接着可以创建一个用于数据归档的表：

```sql
CREATE TABLE archive_test (
    date date,
    name string,
    host int,
)
```

再执行：

```sql
INSERT INTO TABLE archive_test SELECT * FROM test;
```


### 参考链接

- https://www.elastic.co/guide/en/elasticsearch/hadoop/current/reference.html
