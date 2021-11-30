---
layout: post
title: Elasticsearch
categories: [中间件]
description: es
keywords: es,中间件
---

Elasticsearch是高度可伸缩的开源全文搜索和分析引擎。它允许我们快速实时地存储、搜索、分析大数据。

Elasticsearch使用Lucene作为内部引擎，但是在你使用它做全文搜索时，只需要使用统一开发好的API即可，而不需要了解其背后复杂的Lucene的运行原理。它的目的是通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单。

不过，Elasticsearch不仅仅是Lucene和全文搜索，我们还能这样去描述它：

- **分布式**的**实时文件存储**，每个字段都被索引并可被搜索
- 分布式的**实时分析搜索引擎**
- 可以扩展到上百台服务器，处理PB级结构化或非结构化数据

### 数据管理

ES提供近乎实时的数据操纵和搜索能力。默认情况下，从索引/更新/删除数据到在搜索结果中出现数据之前，可以预期延迟一秒钟（刷新间隔）。这是与其他SQL数据库的重要区别，SQL数据库中的数据在事务完成后立即可用。

#### 添加Document

接口： **`PUT /<index>/<type>/<ID>`**

现在我们往"customer"中创建一个ID为1 的document：

```shell
curl --location --request PUT 'localhost:9200/customer/_doc/1?pretty' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "John Doe"
}'
```

运行结果如下：

```json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1, //版本号，每修改一次+1
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

通过上面的命令，我们已经成功的添加了一个customer document在customer index中。

值得注意的是,Elasticsearch不需要在显式地创建一个索引之前,我们就可以创建索引文档。在前面的示例中,如果没有事先已经存在索引，Elasticsearch将自动创建索引。

其中，ID为可选项，假如我们没有指定ID，ES则会自动生成一个唯一的ID，如下：

```json
{
    "_index": "customer",
    "_type": "_doc",
    "_id": "BDOQbn0B5XETjuLvc9yy", //自动生成唯一ID
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 2,
    "_primary_term": 1
}
```

注意：我们这里使用了`POST`而不是`PUT`,如果使用PUT会提示错误：

```json
{
    "error": "Incorrect HTTP method for uri [/customer/_doc?pretty] and method [PUT], allowed: [POST]",
    "status": 405
}
```

#### **查询Document**

接口：**`GET /<index>/<type>/<ID>`**

查询索引Customer中ID为1的Document命令：

```shell
curl -X GET "localhost:9200/customer/_doc/1?pretty"
```

查询结果如下：

```json
{
    "_index": "customer",
    "_type": "_doc",
    "_id": "1",
    "_version": 2,
    "_seq_no": 1,
    "_primary_term": 1,
    "found": true,
    "_source": {
        "name": "John Doe" //存储的内容
    }
}
```

#### **替换Document**

接口：**`PUT /<index>/<type>/<ID>`**

添加Document时，假设ID已存在，如前面添加的ID为1，这将会覆盖原来的记录，如下：

```shell
curl --location --request PUT 'localhost:9200/customer/_doc/1?pretty' \
--header 'Content-Type: application/json' \
--data-raw '{
  "name": "zhang san"
}'
```

重新查询id=1，结果如下：

```json
{
    "_index": "customer",
    "_type": "_doc",
    "_id": "1",
    "_version": 3, //版本号自增
    "_seq_no": 3,
    "_primary_term": 1,
    "found": true,
    "_source": {
        "name": "zhang san"
    }
}
```

注意“_version”已自增。

#### **更新Document**

接口：**`POST /<index>/<type>/<ID>/_update`**

更新customer中ID为1的Document中的name为“Tom”，并且添加新的字段age

```shell
curl --location --request POST 'localhost:9200/customer/_doc/1/_update?pretty' \
--header 'Content-Type: application/json' \
--data-raw '{
  "doc": { "name": "Tom", "age": 20 }
}'
```

查询结果如下：

```json
{
    "_index": "customer",
    "_type": "_doc",
    "_id": "1",
    "_version": 4,
    "_seq_no": 4,
    "_primary_term": 1,
    "found": true,
    "_source": {
        "name": "Tom",
        "age": 20
    }
}
```

如果我更新的时候参数只有部分字段或者不传已经有的字段，结果会怎样呢？

```shell
curl --location --request POST 'localhost:9200/customer/_doc/1/_update?pretty' \
--header 'Content-Type: application/json' \
--data-raw '{
  "doc": { "address": "安徽", "sex": "男" } //之前的name和age没有传
}'
```

查询结果如下：

```json
{
    "_index": "customer",
    "_type": "_doc",
    "_id": "1",
    "_version": 5,
    "_seq_no": 5,
    "_primary_term": 1,
    "found": true,
    "_source": {
        "name": "Tom", //还存在
        "age": 20,	//还存在
        "address": "安徽",
        "sex": "男"
    }
}
```

看来**`_update`**接口会 保留已存在的字段，不会直接覆盖，这是和替换接口最大的区别。

#### **删除Document**

接口：**`DELETE /<index>/<type>/<ID>`**

删除customer中ID为2的Document

```shell
curl --location --request DELETE 'localhost:9200/customer/_doc/2?pretty'
```

假设文档不存在，则返回如下：

```json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "2",
  "_version" : 1,
  "result" : "not_found",	//文档不存在
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

我们先添加一条数据，然后再删除，删除结果如下：

```json
{
    "_index": "customer",
    "_type": "_doc",
    "_id": "2",
    "_version": 3,
    "result": "deleted",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 8,
    "_primary_term": 1
}
```

再次查询id=2，数据不存在，结果如下：

```json
{
    "_index": "customer",
    "_type": "_doc",
    "_id": "2",
    "found": false
}
```

#### **批量处理（demo）**

接口：**`POST <index>/<type>/_bulk`**

在ES中，除了上面针对单个Document增、删、改、查之外，ES还提供了一个强大的API`_bulk`，它具备了批量操作的能力。

#####  批量添加

批量添加2个Document

```shell
curl -X POST "localhost:9200/customer/_doc/_bulk?pretty" -H 'Content-Type: application/json' -d'
{"index":{"_id":"11"}}
{"name": "Milton" }
{"index":{"_id":"22"}}
{"name": "Cherish" }
'
```

结果如下：

```json
{
  "took" : 272,
  "errors" : false,
  "items" : [
    {
      "index" : {
        "_index" : "customer",
        "_type" : "_doc",
        "_id" : "11",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 0,
        "_primary_term" : 3,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "customer",
        "_type" : "_doc",
        "_id" : "22",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 1,
        "_primary_term" : 3,
        "status" : 201
      }
    }
  ]
}
```

通过上面可知，已经新增的两条Document。

##### 批量更新和删除

```shell
curl -X POST "localhost:9200/customer/_doc/_bulk?pretty" -H 'Content-Type: application/json' -d'
{"update":{"_id":"11"}} //更新
{"doc": { "name": "Milton Love Cherish" } }
{"delete":{"_id":"22"}} //删除
'
```

运行结果如下：

```json
{
  "took" : 269,
  "errors" : false,
  "items" : [
    {
      "update" : {
        "_index" : "customer",
        "_type" : "_doc",
        "_id" : "11",
        "_version" : 2,
        "result" : "updated",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 1,
        "_primary_term" : 3,
        "status" : 200
      }
    },
    {
      "delete" : {
        "_index" : "customer",
        "_type" : "_doc",
        "_id" : "22",
        "_version" : 2,
        "result" : "deleted",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 2,
        "_primary_term" : 3,
        "status" : 200
      }
    }
  ]
}
```

#### 批量创建/更新 index

```json
POST _bulk
{ "index" : { "_index" : "teacher", "_type" : "_doc", "_id" : "1" } }
{ "name" : "Milton" }
{ "index" : { "_index" : "teacher", "_type" : "_doc", "_id" : "2" } }
{ "name" : "Cherish" }
{ "index" : { "_index" : "teacher", "_type" : "_doc", "_id" : "3" } }
{ "name" : "Evan" }
```

#### 批量创建 create

```json
POST _bulk
{ "create" : { "_index" : "teacher", "_type" : "_doc", "_id" : "4" } }
{ "name" : "yangp" }
{ "create" : { "_index" : "teacher", "_type" : "_doc", "_id" : "5" } }
{ "name" : "yangf" }
```

> index 和 create 都可以增加文档。使用index时，如果记录已存在，则会进行更新，如果不存在，则会新增；但是使用create时，如果记录已存在，则会创建失败。

#### 批量更新 update

```json
POST /teacher/_doc/_bulk
{ "update" : { "_id" : "4" } }
{"doc":{ "name" : "yangp_update" }}
{ "update" : { "_id" : "5" } }
{"doc":{ "name" : "yangf_update" }}
```

#### 批量删除 delete

```json
POST /teacher/_doc/_bulk
{"delete":{"_id":"4"}}
{"delete":{"_id":"5"}}
```

#### **批量获取 `_mget`**

获取索引teacher中，id为1,2的document

```shell
GET /teacher/_doc/_mget
{
    "ids":["1","2"]
}
```

```shell
GET /teacher/_mget
{
    "docs":[
        {
            "_type":"_doc",
            "_id":"1"
        },
        {
            "_type":"_doc",
            "_id":"2"
        }
        ]
}
```

查询方式有多种，这里只展示2种。

#### 匹配删除`_delete_by_query`

从索引teacher中删除name为“Evan”的document

```shell
POST /teacher/_delete_by_query
{
    "query":{
        "match": {
           "name": "Evan"
        }
    }
}
```

#### **匹配更新`_update_by_query`**

从索引teacher中，更新name包含“Milton”的文档，设置其gener=“Boy”,age=100

```shell
POST /teacher/_update_by_query
{
    "query":{
        "match": {
           "name": "Milton"
        }
    },
    "script":{
        "source":"ctx._source.gener=params.gener;ctx._source.age=params.age",
        "params":{
            "gener":"Boy"
,
           "age":100
        }
    }
}
```

