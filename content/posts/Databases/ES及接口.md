---
title: "ES接口"
date: 2023-08-11T11:18:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
- es
---

# ES及接口

> v1.0.0

Base URLs:

* <a href="http://127.0.0.1:9200">测试环境: http://127.0.0.1:9200</a>

# 索引

## PUT 创建索引

PUT /test1

> 返回示例

> 200 Response

```json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "string"
}
```

### 返回结果

|状态码|状态码含义|说明|数据模型|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|成功|Inline|

### 返回数据结构

状态码 **200**

|名称|类型|必选|约束|中文名|说明|
|---|---|---|---|---|---|
|» acknowledged|boolean|true|none|响应结果|none|
|» shards_acknowledged|boolean|true|none|分片结果|none|
|» index|string|true|none|索引名称|none|

## DELETE 删除索引

DELETE /test1

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

|状态码|状态码含义|说明|数据模型|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|成功|Inline|

### 返回数据结构

## GET 查询索引

GET /test

> 返回示例

> 200 Response

```json
{
  "test": {
    "aliases": {},
    "mappings": {},
    "settings": {
      "index": {
        "routing": {
          "allocation": null
        },
        "number_of_shards": "string",
        "provided_name": "string",
        "creation_date": "string",
        "number_of_replicas": "string",
        "uuid": "string",
        "version": {
          "created": null
        }
      }
    }
  }
}
```

### 返回结果

|状态码|状态码含义|说明|数据模型|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|成功|Inline|

### 返回数据结构

状态码 **200**

|名称|类型|必选|约束|中文名|说明|
|---|---|---|---|---|---|
|» test|object|true|none|索引名|none|
|»» aliases|object|true|none|别名|none|
|»» mappings|object|true|none|映射|none|
|»» settings|object|true|none|设置|none|
|»»» index|object|true|none|索引|none|
|»»»» routing|object|true|none||none|
|»»»»» allocation|object|true|none||none|
|»»»»»» include|object|true|none||none|
|»»»»»»» _tier_preference|string|true|none||none|
|»»»» number_of_shards|string|true|none||none|
|»»»» provided_name|string|true|none||none|
|»»»» creation_date|string|true|none||none|
|»»»» number_of_replicas|string|true|none||none|
|»»»» uuid|string|true|none||none|
|»»»» version|object|true|none||none|
|»»»»» created|string|true|none||none|

## GET 查看全部索引

GET /_cat/indices

> 返回示例

> 成功

### 返回结果

|状态码|状态码含义|说明|数据模型|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|成功|Inline|

### 返回数据结构

## PUT 索引映射(相当于指定字段的类型,keyword为不可分割类型,查询时必须全部满足才可以查询到结果,index为false的字段不能单独查询)

PUT /user/_mapping

> Body 请求参数

```json
{
  "properties": {
    "name": {
      "type": "text",
      "index": true
    },
    "sex": {
      "type": "keyword",
      "index": true
    },
    "tel": {
      "type": "keyword",
      "index": false
    }
  }
}
```

### 请求参数

|名称|位置|类型|必选|说明|
|---|---|---|---|---|
|body|body|object| 否 |none|

> 返回示例

> 成功

```json
{
  "acknowledged": true
}
```

### 返回结果

|状态码|状态码含义|说明|数据模型|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|成功|Inline|

### 返回数据结构

状态码 **200**

|名称|类型|必选|约束|中文名|说明|
|---|---|---|---|---|---|
|» acknowledged|boolean|true|none||none|

## GET 查看指定索引的映射规则

GET /user/_mapping

> 返回示例

> 成功

```json
{
  "user": {
    "mappings": {
      "properties": {
        "name": {
          "type": "text"
        },
        "sex": {
          "type": "keyword"
        },
        "tel": {
          "type": "keyword",
          "index": false
        }
      }
    }
  }
}
```

### 返回结果

|状态码|状态码含义|说明|数据模型|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|成功|Inline|

### 返回数据结构

状态码 **200**

|名称|类型|必选|约束|中文名|说明|
|---|---|---|---|---|---|
|» user|object|true|none||none|
|»» mappings|object|true|none||none|
|»»» properties|object|true|none||none|
|»»»» name|object|true|none||none|
|»»»»» type|string|true|none||none|
|»»»» sex|object|true|none||none|
|»»»»» type|string|true|none||none|
|»»»» tel|object|true|none||none|
|»»»»» type|string|true|none||none|
|»»»»» index|boolean|true|none||none|

# 文档

## POST 创建文档

POST /shopping/_doc

> Body 请求参数

```json
{
  "title": "小米手机",
  "category": "小米",
  "image": "http://abc.com/a.jpg",
  "price": 2999.99
}
```

### 请求参数

|名称|位置|类型|必选|说明|
|---|---|---|---|---|
|body|body|object| 否 |none|

> 返回示例

> 成功

```json
{
  "_index": "shopping",
  "_id": "PAGtQ4kBKLdwdVo2W1at",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```

### 返回结果

|状态码|状态码含义|说明|数据模型|
|---|---|---|---|
|201|[Created](https://tools.ietf.org/html/rfc7231#section-6.3.2)|成功|Inline|

### 返回数据结构

状态码 **201**

|名称|类型|必选|约束|中文名|说明|
|---|---|---|---|---|---|
|» _index|string|true|none||none|
|» _id|string|true|none||none|
|» _version|integer|true|none||none|
|» result|string|true|none||none|
|» _shards|object|true|none||none|
|»» total|integer|true|none||none|
|»» successful|integer|true|none||none|
|»» failed|integer|true|none||none|
|» _seq_no|integer|true|none||none|
|» _primary_term|integer|true|none||none|

## GET 获取指定字段的平均值

GET /shopping/_search

> Body 请求参数

```json
{
  "aggs": {
    "suibianqide": {
      "avg": {
        "field": "price"
      }
    }
  },
  "size": 0
}
```

### 请求参数

|名称|位置|类型|必选|说明|
|---|---|---|---|---|
|body|body|object| 否 |none|

> 返回示例

> 成功

```json
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 7,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "suibianqide": {
      "value": 3142.8471330915177
    }
  }
}
```

### 返回结果

|状态码|状态码含义|说明|数据模型|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|成功|Inline|

### 返回数据结构

状态码 **200**

|名称|类型|必选|约束|中文名|说明|
|---|---|---|---|---|---|
|» took|integer|true|none||none|
|» timed_out|boolean|true|none||none|
|» _shards|object|true|none||none|
|»» total|integer|true|none||none|
|»» successful|integer|true|none||none|
|»» skipped|integer|true|none||none|
|»» failed|integer|true|none||none|
|» hits|object|true|none||none|
|»» total|object|true|none||none|
|»»» value|integer|true|none||none|
|»»» relation|string|true|none||none|
|»» max_score|null|true|none||none|
|»» hits|[string]|true|none||none|
|» aggregations|object|true|none||none|
|»» suibianqide|object|true|none||none|
|»»» value|number|true|none||none|

## GET 根据文档id查询信息

GET /shopping/_doc/PAGtQ4kBKLdwdVo2W1at

> 返回示例

> 成功

```json
{
  "_index": "shopping",
  "_id": "PAGtQ4kBKLdwdVo2W1at",
  "_version": 1,
  "_seq_no": 0,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "title": "小米手机",
    "category": "小米",
    "image": "http://abc.com/a.jpg",
    "price": 2999.99
  }
}
```

### 返回结果

|状态码|状态码含义|说明|数据模型|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|成功|Inline|

### 返回数据结构

状态码 **200**

|名称|类型|必选|约束|中文名|说明|
|---|---|---|---|---|---|
|» _index|string|true|none||none|
|» _id|string|true|none||none|
|» _version|integer|true|none||none|
|» _seq_no|integer|true|none||none|
|» _primary_term|integer|true|none||none|
|» found|boolean|true|none||none|
|» _source|object|true|none||none|
|»» title|string|true|none||none|
|»» category|string|true|none||none|
|»» image|string|true|none||none|
|»» price|number|true|none||none|

## DELETE 根据文档id删除该文档

DELETE /shopping/_doc/PAGtQ4kBKLdwdVo2W1at

> Body 请求参数

```json
{}
```

### 请求参数

|名称|位置|类型|必选|说明|
|---|---|---|---|---|
|body|body|object| 否 |none|

> 返回示例

> 成功

```json
{
  "_index": "shopping",
  "_id": "PAGtQ4kBKLdwdVo2W1at",
  "_version": 2,
  "result": "deleted",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 1,
  "_primary_term": 1
}
```

### 返回结果

|状态码|状态码含义|说明|数据模型|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|成功|Inline|

### 返回数据结构

状态码 **200**

|名称|类型|必选|约束|中文名|说明|
|---|---|---|---|---|---|
|» _index|string|true|none||none|
|» _id|string|true|none||none|
|» _version|integer|true|none||none|
|» result|string|true|none||none|
|» _shards|object|true|none||none|
|»» total|integer|true|none||none|
|»» successful|integer|true|none||none|
|»» failed|integer|true|none||none|
|» _seq_no|integer|true|none||none|
|» _primary_term|integer|true|none||none|

## POST 修改文档(和创建文档一样,如要创建文档的id已存在就会覆盖掉原有的文档)

POST /shopping/_doc/PAGtQ4kBKLdwdVo2W1at

> Body 请求参数

```json
{
  "title": "小米手机123",
  "category": "小米",
  "image": "http://abc.com/a.jpg",
  "price": 2999.99
}
```

### 请求参数

|名称|位置|类型|必选|说明|
|---|---|---|---|---|
|body|body|object| 否 |none|

> 返回示例

> 成功

```json
{
  "_index": "shopping",
  "_id": "PAGtQ4kBKLdwdVo2W1at",
  "_version": 3,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 5,
  "_primary_term": 1
}
```

### 返回结果

|状态码|状态码含义|说明|数据模型|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|成功|Inline|

### 返回数据结构

状态码 **200**

|名称|类型|必选|约束|中文名|说明|
|---|---|---|---|---|---|
|» _index|string|true|none||none|
|» _id|string|true|none||none|
|» _version|integer|true|none||none|
|» result|string|true|none||none|
|» _shards|object|true|none||none|
|»» total|integer|true|none||none|
|»» successful|integer|true|none||none|
|»» failed|integer|true|none||none|
|» _seq_no|integer|true|none||none|
|» _primary_term|integer|true|none||none|

## POST 修改文档局部数据

POST /shopping/_update/PQGyQ4kBKLdwdVo2G1Zr

> Body 请求参数

```json
{
  "doc": {
    "price": 3999.99
  }
}
```

### 请求参数

|名称|位置|类型|必选|说明|
|---|---|---|---|---|
|body|body|object| 否 |none|

> 返回示例

> 成功

```json
{
  "_index": "shopping",
  "_id": "PAGtQ4kBKLdwdVo2W1at",
  "_version": 7,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 10,
  "_primary_term": 1
}
```

### 返回结果

|状态码|状态码含义|说明|数据模型|
|---|---|---|---|
|201|[Created](https://tools.ietf.org/html/rfc7231#section-6.3.2)|成功|Inline|

### 返回数据结构

状态码 **201**

|名称|类型|必选|约束|中文名|说明|
|---|---|---|---|---|---|
|» _index|string|true|none||none|
|» _id|string|true|none||none|
|» _version|integer|true|none||none|
|» result|string|true|none||none|
|» _shards|object|true|none||none|
|»» total|integer|true|none||none|
|»» successful|integer|true|none||none|
|»» failed|integer|true|none||none|
|» _seq_no|integer|true|none||none|
|» _primary_term|integer|true|none||none|

# 数据模型

