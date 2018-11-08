## 索引相关API
### 创建索引
```json
# create index
PUT /test_index
```
### 获取所有索引
```json
GET _cat/indices
```

### 删除索引
```json
DELETE /test_index
```

## 文档相关API

### 创建文档
- 指定id创建文档
```json
PUT /test_index/doc/1
{
    "username":"alfred",
    "age":1
}
```
**test_index:** 索引名称
**doc:** type，es6中type已经没有什么用了
**id:**文档id
如果索引不存在es会自动创建

- 不指定id创建文档
```json
POST /test_index/doc
{
  "username":"tome",
  "age":20
}
```
返回结果：
```json
{
  "_index": "test_index",
  "_type": "doc",
  "_id": "FQ-Q7mYBO5ZBGU1Sekpv",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```
注意：请求方式变了



### 查询文档API

- 查询文档

  - 指定要查询的文档id

    ```json
    GET test_index/doc/1
    ```

    响应结果

    ```json
    {
      "_index": "test_index",
      "_type": "doc",
      "_id": "1",
      "_version": 1,
      "found": true,
      "_source": {
        "username": "alfred",
        "age": 1
      }
    }
    ```

    `_source`存储了文档的完整原始数据

  - 搜索所有的文档，用`_search`，如下

    ```json
    GET /test_index/_search
    {
      "query": {
        "term": {
          "_id": {
            "value": "1"
          }
        }
      }
    } 
    ```

    查询语句，Json格式，放在http body中发送到es

    响应结果：

    ```json
    {
      "took": 9,
      "timed_out": false,
      "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
      },
      "hits": {
        "total": 1,
        "max_score": 1,
        "hits": [
          {
            "_index": "test_index",
            "_type": "doc",
            "_id": "1",
            "_score": 1,
            "_source": {
              "username": "alfred",
              "age": 1
            }
          }
        ]
      }
    }
    ```

    响应结果解析如下：

    **took：** 查询耗时，单位ms

    **total：** 符合条件的总文档数

    **hits：** 返回文档详情数据数组，默认前10个文档

    **_index：** 索引名

    **_id：** 文档id

    **_source：** 文档详情

- 批量创建文档API

  - es允许一次创建多个文档，从而减少网络传输开销，提升写入速率。请求地址是`_bulk`,请求如下

    ```json
    POST _bulk
    {"index":{"_index":"test_index","_type":"doc","_id":"3"}}
    {"username":"alfred","age":10}
    {"delete":{"_index":"test_index","_type":"doc","_id":"1"}}
    {"update":{"_id":"2","_index":"test_index","_type":"doc"}}
    {"doc":{"age":20}}
    ```

    响应结果：

    ```json
    {
      "took": 192,
      "errors": true,
      "items": [
        {
          "index": {
            "_index": "test_index",
            "_type": "doc",
            "_id": "3",
            "_version": 1,
            "result": "created",
            "_shards": {
              "total": 2,
              "successful": 2,
              "failed": 0
            },
            "_seq_no": 0,
            "_primary_term": 1,
            "status": 201
          }
        },
        {
          "delete": {
            "_index": "test_index",
            "_type": "doc",
            "_id": "1",
            "_version": 2,
            "result": "deleted",
            "_shards": {
              "total": 2,
              "successful": 2,
              "failed": 0
            },
            "_seq_no": 1,
            "_primary_term": 1,
            "status": 200
          }
        },
        {
          "update": {
            "_index": "test_index",
            "_type": "doc",
            "_id": "2",
            "status": 404,
            "error": {
              "type": "document_missing_exception",
              "reason": "[doc][2]: document missing",
              "index_uuid": "0AatPvgyS22KJJR30zr39w",
              "shard": "2",
              "index": "test_index"
            }
          }
        }
      ]
    }
    ```

    解析响应结果：

    **took：** 查询耗时 单位是ms

    **items：** 每个bulk操作的返回结果

- es允许一次查询多个文档

  - 请求地址是`_mget`，如下：

    ```json
    GET /_mget
    {
      "docs":[
        {
          "_index":"test_index",
          "_type":"doc",
          "_id":"1"
        },
        {
        "_index":"test_index",
        "_type":"doc",
        "_id":"3"
        }
        ]
    }
    ```

    响应结果：

    ```json
    {
      "docs": [
        {
          "_index": "test_index",
          "_type": "doc",
          "_id": "1",
          "found": false
        },
        {
          "_index": "test_index",
          "_type": "doc",
          "_id": "3",
          "_version": 1,
          "found": true,
          "_source": {
            "username": "alfred",
            "age": 10
          }
        }
      ]
    }
    ```