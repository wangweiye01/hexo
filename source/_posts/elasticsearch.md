---
title: elasticsearch全文搜索
date: 2020-03-21 16:02:02
tags:
---

## 什么是Elasticsearch

Elasticsearch是一个开源的，分布式，高可用的数据库，是目前[全文搜索](https://baike.baidu.com/item/%E5%85%A8%E6%96%87%E6%90%9C%E7%B4%A2%E5%BC%95%E6%93%8E)的首选。它可以快速的存储、搜索、分析海量数据。

Elastic是基于开源库Lucene开发的，提供了REST API的操作接口，简单易用

## 优秀案例

GitHub使用ElasticSearch做PB级搜索；包括但不限于维基百科、携程等成功案例；

## 为什么ES全文搜索查询速度快

ES检索速度极快的很重要原理就是使用了`倒排索引`。什么是倒排索引？通常我们理解的索引就是通过`key`找到对应的`value`，所以通俗来讲倒排索引就是通过`value`找到对应的`key`

![termindex.png](http://s1.wailian.download/2020/05/19/termindex.png)

理解上图可以了解倒排索引的基本原理。首先ES索引时会先将文本进行分词，然后记录分词和文档的对应关系。当查询时，对查询条件同样进行分词，然后根据分词匹配对应的文档。

## 基本概念

### Node和Cluster

Elastic 本质上是一个分布式数据库，允许多台服务器协同工作，每台服务器可以运行多个Elastic实例

单个 Elastic 实例称为一个节点（node）。一组节点构成一个集群（cluster）

### Index

索引是具有某种相似特征文档的集合。类似于MySql中的Database

### Type

es6.x建议在一个index中保持一个type

### Document

文档是可以被索引的基本信息单元。文档用JSON表示。类似于MySql中的一条数据(Row)

### Field

属性，类似于MySql中的某个字段(Column)

## 安装Elasticsearch(6.3.2)

自己安装，此处选择6.3.2版本


## Kibana安装

Kibana 是为 Elasticsearch设计的开源分析和可视化平台。你可以使用 Kibana 来搜索，查看存储在 Elasticsearch 索引中的数据并与之交互。你可以很容易实现高级的数据分析和可视化，以图标的形式展现出来。

注意与安装的Elasticsearch版本号(6.3.2)一致

## 操作数据

```
# 新增数据1
PUT /person/_doc/1
{
  "first_name": "John",
  "last_name": "Smith",
  "age": 25,
  "about": "I love to go rock climbing",
  "interests": [
    "sports",
    "music"
  ]
}

# 新增数据2
PUT /person/_doc/2
{
  "first_name": "Eric",
  "last_name": "Smith",
  "age": 23,
  "about": "I love basketball",
  "interests": [
    "sports",
    "reading"
  ]
}

# 获得1
GET /person/_doc/1
# 获得2
GET /person/_doc/2

# bool搜索
POST /person/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "first_name": "Eric"
          }
        }
      ]
    }
  }
}

# bool搜索
POST /person/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "last_name": "Smith"
          }
        },
        {
          "match": {
            "about": "basketball"
          }
        }
      ]
    }
  }
}
```

## ik分词器

[ik分词器](https://github.com/medcl/elasticsearch-analysis-ik)是最流行的中文分词器，点击可查看ik和ES的版本对照关系以及安装方式

ik提供了两个分词算法`ik_smart`和`ik_max_word`

通过以下两个图，可以看出两个算法的区别。ik_smart为智能切分，ik_max_word为最细粒度划分

ik_smart

![smart.png](http://s1.wailian.download/2020/05/08/smart.png)

ik_max_word

![WX20200508-145012-max-word.png](http://s1.wailian.download/2020/05/08/WX20200508-145012-max-word.png)

根据两个算法的区别，一般遵循以下原则：

 ***索引时，为了提供索引的覆盖范围，通常会采用ik_max_word分析器，会以最细粒度分词索引，搜索时为了提高搜索准确度，会采用ik_smart分析器，会以粗粒度分词***

 ## 快照创建于恢复

 使用无论哪个存储数据的软件，定期备份你的数据都是很重要的，es提供了快照机制。你可以使用`snapshot`API。这个会拿到你集群里当前的状态和数据然后保存到一个共享仓库里。这个备份过程是"智能"的。你的第一个快照会是一个数据的完整拷贝，但是所有后续的快照会保留的是已存快照和新数据之间的差异。随着你不时的对数据进行快照，备份也在增量的添加和删除。这意味着后续备份会相当快速，因为它们只传输很小的数据量。

 ### 创建仓库

 ```
 PUT _snapshot/my_backup ①
{
    "type": "fs", ②
    "settings": {
        "location": "/mount/backups/my_backup" ③
    }
}
 ```
 
 ① 给我们的仓库取一个名字，在本例它叫 my_backup。
 ② 我们指定仓库的类型应该是一个共享文件系统。
 ③ 最后，我们提供一个已挂载的设备作为目的地址。注意：共享文件系统路径必须确保集群所有节点都可以访问到。

 ### 创建快照

 一个仓库可以包含多个快照。每个快照跟一系列索引相关（比如所有索引，一部分索引，或者单个索引）。当创建快照的时候，你指定你感兴趣的索引然后给快照取一个唯一的名字。

#### 快照所有打开的索引

 让我们从最基础的快照命令开始：

 `PUT _snapshot/my_backup/snapshot_1`

 这个会备份所有打开的索引到 my_backup 仓库下一个命名为 snapshot_1 的快照里。这个调用会立刻返回，然后快照会在后台运行。如果你想阻塞调用直到快照完成可以添加`wait_for_completion`标记实现

`PUT _snapshot/my_backup/snapshot_1?wait_for_completion=true`

#### 快照指定索引

```
PUT _snapshot/my_backup/snapshot_2
{
    "indices": "index_1,index_2"
}
```

这个快照命令现在只会备份`index1`和`index2`了。

### 获取快照信息

获取某个仓库下所有快照信息

`GET _snapshot/my_backup/_all`

获取某个仓库下单个快照信息

`GET _snapshot/my_backup/snapshot_2`

响应

```json
{
    "snapshots": [
        {
            "snapshot": "snapshot_202005080945",
            "version_id": 2030599,
            "version": "2.3.5",
            "indices": [
                "pipe"
            ],
            "state": "SUCCESS",
            "start_time": "2020-05-08T01:45:57.540Z",
            "start_time_in_millis": 1588902357540,
            "end_time": "2020-05-08T01:45:57.586Z",
            "end_time_in_millis": 1588902357586,
            "duration_in_millis": 46,
            "failures": [],
            "shards": {
                "total": 1,
                "failed": 0,
                "successful": 1
            }
        }
    ]
}
```

### 删除快照

`DELETE _snapshot/my_backup/snapshot_2`

### 使用快照恢复数据

快照的目的就是为了备份，为了恢复。一旦你备份过了数据，恢复它就简单了：只要在你希望恢复回集群的快照 ID后面加上`_restore`即可

`POST _snapshot/my_backup/snapshot_1/_restore`

默认行为是把这个快照里存有的所有索引都恢复。如果`snapshot_1包括五个索引，这五个都会被恢复到我们集群里。和`snapshot`API 一样，我们也可以选择希望恢复具体哪个索引。

还有附加的选项用来重命名索引。这个选项允许你通过模式匹配索引名称，然后通过恢复进程提供一个新名称。如果你想在不替换现有数据的前提下，恢复老数据来验证内容，或者做其他处理，这个选项很有用。让我们从快照里恢复单个索引并提供一个替换的名称：

```
POST /_snapshot/my_backup/snapshot_1/_restore
{
    "indices": "index_1", ①
    "rename_pattern": "index_(.+)", ②
    "rename_replacement": "restored_index_$1" ③
}
```

① 只恢复`index_1`索引，忽略快照中存在的其余索引。
② 查找所提供的模式能匹配上的正在恢复的索引。
③ 然后把它们重命名成替代的模式。

这个会恢复`index_1`到你及群里，但是重命名成了`restored_index_1`
