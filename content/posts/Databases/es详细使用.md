---
title: "ES详细使用"
date: 2019-06-19T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- databases
tags: 
- ES
---

```
Value Count - 类似sql的count函数，统计总数
Cardinality - 类似SQL的count(DISTINCT 字段)， 统计不重复的数据总数
Avg - 求平均值
Sum - 求和
Max - 求最大值
Min - 求最小值


Terms聚合 - 类似SQL的group by，根据字段唯一值分组
Histogram聚合 - 根据数值间隔分组，例如: 价格按100间隔分组，0、100、200、300等等
Date histogram聚合 - 根据时间间隔分组，例如：按月、按天、按小时分组
Range聚合 - 按数值范围分组，例如: 0-150一组，150-200一组，200-500一组。
```
```json
POST /sales/_search?size=0
{
    "aggs" : {
        "prices" : { // 聚合查询名字，随便取一个
            "histogram" : { // 聚合类型为：histogram
                "field" : "price", // 根据price字段分桶
                "interval" : 50 // 分桶的间隔为50，意思就是price字段值按50间隔分组
            }
        }
    }
}
```
```json
POST /sales/_search?size=0
{
    "aggs" : {
        "sales_over_time" : { // 聚合查询名字，随便取一个
            "date_histogram" : { // 聚合类型为: date_histogram
                "field" : "date", // 根据date字段分组
                "calendar_interval" : "month", // 分组间隔：month代表每月、支持minute（每分钟）、hour（每小时）、day（每天）、week（每周）、year（每年）
                "format" : "yyyy-MM-dd" // 设置返回结果中桶key的时间格式
            }
        }
    }
}
```
```json
GET /_search
{
    "aggs" : {
        "price_ranges" : {
            "range" : {
                "field" : "price",
                "keyed" : true,
                "ranges" : [
                    // 通过key参数，配置每一个分桶的名字
                    { "key" : "cheap", "to" : 100 },
                    { "key" : "average", "from" : 100, "to" : 200 },
                    { "key" : "expensive", "from" : 200 }
                ]
            }
        }
    }
}
```

# 聚合后排序

_count - 按文档数排序。对 terms 、 histogram 、 date_histogram 有效
_term - 按词项的字符串值的字母顺序排序。只在 terms 内使用
_key - 按每个桶的键值数值排序, 仅对 histogram 和 date_histogram 有效

```json
GET /cars/_search
{
    "size" : 0,
    "aggs" : {
        "colors" : { // 聚合查询名字，随便取一个
            "terms" : { // 聚合类型为: terms
              "field" : "color", 
              "order": { // 设置排序参数
                "_count" : "asc"  // 根据_count排序，asc升序，desc降序
              }
            }
        }
    }
}
```

```json
通常情况下，我们根据桶聚合分桶后，都会对桶内进行多个维度的指标聚合，所以我们也可以根据桶内指标聚合的结果进行排序
GET /cars/_search
{
    "size" : 0,
    "aggs" : {
        "colors" : { // 聚合查询名字
            "terms" : { // 聚合类型: terms，先分桶
              "field" : "color", // 分桶字段为color
              "order": { // 设置排序参数
                "avg_price" : "asc"  // 根据avg_price指标聚合结果，升序排序。
              }
            },
            "aggs": { // 嵌套聚合查询，设置桶内聚合指标
                "avg_price": { // 聚合查询名字，前面排序引用的就是这个名字
                    "avg": {"field": "price"} // 计算price字段平均值
                }
            }
        }
    }
}
```

# 获取当前es信息

```json
GET /
```

# 添加文档,如果index不存在则自动创建

```json
PUT twitter/_doc/1
{
  "user": "GB",
  "uid": 1,
  "city": "Beijing",
  "province": "Beijing",
  "country": "China"
}
```

# 获取Twitter的字段类型

```json
GET twitter/_mapping
```
# 自定义索引字段类型

```json
PUT test
{
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword"
      },
      "message": {
        "type": "text"
      }
    }
  }
}
```

# 在已有索引字段的基础上添加指定类型的字段

```json
PUT test/_mapping
{
  "properties": {
    "age": {
      "type": "long"
    }
  }
}
```

# 使用?refresh=true可以立即刷新该文档使其可以被马上搜索到

```json
PUT twitter/_doc/1?refresh=true
{
  "user": "GB",
  "uid": 1,
  "city": "Beijing",
  "province": "Beijing",
  "country": "China"
}
```

# ?refresh=true

> 这种操作会使Elasticsearch 变得非常慢,通过设置 refresh=wait_for。这样相当于一个同步的操作，它等待下一个 refresh 周期发生完后，才返回

```json
PUT twitter/_doc/1?refresh=wait_for
{
  "user": "GB",
  "uid": 1,
  "city": "Beijing",
  "province": "Beijing",
  "country": "China"
}

```
# 使用_doc如果id相同则会更改并把版本号加一,使用_create如果存在该id则会报错

```json
PUT twitter/_create/1
{
  "user": "GB",
  "uid": 1,
  "city": "Shenzhen",
  "province": "Guangdong",
  "country": "China"
}
```

# 通过设置?op_type=create参数可以达到和上边相同结果

```json
PUT twitter/_doc/1?op_type=create
{
  "user": "GB",
  "uid": 1,
  "city": "Beijing",
  "province": "Beijing",
  "country": "China"
}
```

# 不指定id,es自动生成id,只能在post请求下才可以

```json
POST twitter/_doc
{
  "user": "MB",
  "uid": 2,
  "city": "Shenzhen",
  "province": "Guangdong",
  "country": "China"
}
```

# 获取id为1的文档信息

```json
GET twitter/_doc/1
```

# 只获取文档id为1的source信息

```json
GET twitter/_source/1
```

# 获取ids为1和2的

```json
GET twitter/_mget
{
  "ids":["1","2"]
}
```

# 只修改id为1的user字段

```json
POST twitter/_update/1
{
  "doc":{
    "user":"abccde"
  }
}
```

# 在不知道id的情况下修改user为GB的用户信息

```json
POST twitter/_update_by_query
{
  "query": {
    "match": {
      "user": "GB"
    }
  },
  "script": {
    "source": "ctx._source.city = params.city;ctx._source.province = params.province;ctx._source.country = params.country",
    "lang": "painless",
    "params": {
      "city": "上海",
      "province": "上海",
      "country": "中国"
    }
  }
}
```

# 检查id为1的doc是否存在

```json
HEAD twitter/_doc/1
```

# 检查索引是否存在

```json
HEAD twitter
```

# 删除id为1的doc

```json
DELETE twitter/_doc/1
```

# 删除索引

```json
DELETE twitter
```

# 不知道id的情况下删除city为上海的用户

```json
POST twitter/_delete_by_query
{
  "query":{
    "match":{
      "city":"上海"
    }
  }
}
```

# 批量创建

> index可以为create,delete,update

```json
POST _bulk
{ "index" : { "_index" : "twitter", "_id": 1} }
{"user":"双榆树-张三","message":"今儿天气不错啊，出去转转去","uid":2,"age":20,"city":"北京","province":"北京","country":"中国","address":"中国北京市海淀区","location":{"lat":"39.970718","lon":"116.325747"}}
{ "index" : { "_index" : "twitter", "_id": 2 }}
{"user":"东城区-老刘","message":"出发，下一站云南！","uid":3,"age":30,"city":"北京","province":"北京","country":"中国","address":"中国北京市东城区台基厂三条3号","location":{"lat":"39.904313","lon":"116.412754"}}
{ "index" : { "_index" : "twitter", "_id": 3} }
{"user":"东城区-李四","message":"happy birthday!","uid":4,"age":30,"city":"北京","province":"北京","country":"中国","address":"中国北京市东城区","location":{"lat":"39.893801","lon":"116.408986"}}
{ "index" : { "_index" : "twitter", "_id": 4} }
{"user":"朝阳区-老贾","message":"123,gogogo","uid":5,"age":35,"city":"北京","province":"北京","country":"中国","address":"中国北京市朝阳区建国门","location":{"lat":"39.718256","lon":"116.367910"}}
{ "index" : { "_index" : "twitter", "_id": 5} }
{"user":"朝阳区-老王","message":"Happy BirthDay My Friend!","uid":6,"age":50,"city":"北京","province":"北京","country":"中国","address":"中国北京市朝阳区国贸","location":{"lat":"39.918256","lon":"116.467910"}}
{ "index" : { "_index" : "twitter", "_id": 6} }
{"user":"虹桥-老吴","message":"好友来了都今天我生日，好友来了,什么 birthday happy 就成!","uid":7,"age":90,"city":"上海","province":"上海","country":"中国","address":"中国上海市闵行区","location":{"lat":"31.175927","lon":"121.383328"}}

POST _bulk
{ "delete" : { "_index" : "twitter", "_id": 1 }}

POST _bulk
{ "update" : { "_index" : "twitter", "_id": 2 }}
{"doc": { "city": "长沙"}}
```

# 超找全部doc

```json
POST twitter/_search
```

# 获取twitter有多少条doc

```json
GET twitter/_count
```

# 索引统计

```json
GET twitter/_stats
```

# 同时获得多个索引的统计数据

```json
GET twitter1,twitter2,twitter3/_stats
```

# 使用通配符来匹配多个索引

```json
GET twitter*/_stats
```

# 分页

```json
GET twitter/_search
{
  "size":"2",
  "from":"2"
}


GET users/_search
{
  "size": 2,
  "sort":{
    "age":"desc"
  }
}

GET users/_search
{
  "size": 2,
  "sort":{
    "age":"desc"
  },
  "search_after":[12]
}


POST books/_search?scroll=10m
{
  "query": {
    "match_all": {}
  },
  "sort": { "price": "desc" }, 
  "size": 2
}
POST /_search/scroll                                                    
{
  "scroll" : "5m",   
  "scroll_id" : "FGluY2x1ZGVfY29udGV4dF9......==" 
}
```
# ?filter_path 过滤返回数据,指定需要的数据集

```json
GET twitter/_search?filter_path=hits.hits._source
{
  "size":"2",
  "from":"2"
}
```
# fields 只要查有指定字段的

```json
GET twitter/_search
{
  "fields":["user","city"]
}
```

# _source:false 不展示_source资源

```json
GET twitter/_search
{
  "fields":["user","city"],
  "_source":"false"
}
```

# includes包含和excludes不包含

```json
GET twitter/_search
{
  "_source": {
    "includes": [
      "user*",
      "location*"
    ],
    "excludes": [
      "*.lat"
    ]
  },
  "query": {
    "match_all": {}
  }
}
```

# 想要的 field 可能在 _source 里根本没有，可以使用 script field 来生成这些 field

```json
GET twitter/_search
{
  "_source":[],
  "script_fields": {
    "years_to_100": {
      "script": {
        "lang": "painless",
        "source": "100-params._source['age']"
      }
    },
    "year_of_birth":{
      "script": "2023 - params._source['age']"
    }
  }
}
```

# 查找message中包含 出 的

```json
GET twitter/_search
{
  "query":{
    "match":{
      "message":"出"
    }
  }
}
```

# 查找message中包含 出 的且分数最低为1.4

```json
GET twitter/_search
{
  "query":{
    "match":{
      "message":"出"
    }
  },
  "min_score":1.4
}
```

# or匹配任意一个字符即可,and需要完整匹配全部

```json
GET twitter/_search
{
  "query": {
    "match": {
      "user": {
        "query": "朝阳区-老贾",
        "operator": "or"
      }
    }
  }
}
```

# 高亮显示

>默认为<em></em>

```json
GET twitter/_search
{
  "query": {
    "match": {
      "address": "北京"
    }
  },
  "highlight": {
    "fields": {
      "address": {}
    }
  }
}
```

# 自定义高亮的标签

```json
GET twitter/_search
{
  "query":{
    "match": {
      "address": "北京"
    }
  },
  "highlight":{
    "fields":{
      "address":{}
    },
    "pre_tags":["<red>"],
    "post_tags": ["</red>"]
  }
}
```

# 同时查找多个id

```json
GET twitter/_search
{
  "query":{
    "ids": {
      "values": ["1","2"]
    }
  }
}
```

# 权重

> 查找字段user,address,message中包含 朝阳 且address分数进行三倍权重,"type": "best_fields"分数按照最高字段的结算

```json
GET twitter/_search
{
  "query":{
    "multi_match": {
      "query": "朝阳",
      "fields": ["user","address^3","message"],
      "type": "best_fields"
    }
  }
}
```

# prefix超找前缀为指定字符的

```json
GET twitter/_search
{
  "query":{
    "prefix": {
      "user": {
        "value": "朝"
      }
    }
  }
}

```
# keyword 指定精准匹配

```json
GET twitter/_search
{
  "query":{
    "term": {
      "user.keyword": {
        "value": "朝阳区-老贾"
      }
    }
  }
}
```

# terms 精准匹配多个值

```json
GET twitter/_search
{
  "query":{
    "terms": {
      "user.keyword": [
        "朝阳区-老贾",
        "东城区-老刘"
      ]
    }
  }
}
```

# 复合查询

> must:必须匹配,must_not:不得匹配,should:满不满足都行,filter:必须匹配于must区别:分数在过滤子句中是不相关的

>match 查询：这是Elasticsearch中最基本的查询类型。它根据给定的字段和查询文本进行匹配。它可以是全局的（默认）或局部的（通过在字段名后面添加 ^ 来指定）。
match_all 查询：这个查询将匹配所有文档，不论其内容如何。它通常用于在不知道具体查询条件或想要匹配所有文档的情况下使用。
match_phrase 查询：这个查询用于在给定字段中执行精确短语匹配。它确保查询文本作为一个整体出现在匹配的字段中，并且词序与原始查询文本中的词序相同。
match_phrase_prefix 查询：这个查询用于执行前缀短语匹配。它查找在给定字段中以查询文本开头的短语。
multi_match 查询：这个查询允许在一个字段或多个字段上执行多个查询。它返回与任何查询匹配的文档。可以通过指定 type 参数来选择要使用的匹配类型（例如 best_fields、most_fields、cross_fields 等）。
```json
GET twitter/_search
{
  "query":{
    "bool": {
      "must": [
        {"match": {
          "user":"朝阳区-老贾"
        }}
      ]
    }
  }
}

// minimum_should_match: 最少需要满足几个should
GET twitter/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "city": "北京"
          }
        },
        {
          "match": {
            "city": "武汉"
          }
        }
      ],
      "minimum_should_match": 2
    }
  }
}
```

# sort 排序

```json
GET twitter/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 30,
        "lte": 40
      }
    }
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    }
  ]
}

```

# es执行sql语句

```json
GET /_sql?
{
  "query": """
    SELECT * FROM twitter 
    WHERE age = 30
  """
}

```



>1. **Term Query（词项查询）**：用于精确匹配某个字段的值，不进行分词，只能匹配完整的词项。例如，搜索一个确切的数字、日期、布尔值或一个确切的词语。
>2. **Match Query（匹配查询）**：根据指定字段进行分词后进行搜索，匹配查询会对搜索词进行分析，然后使用分析器将搜索词转换为一组词项，然后匹配这些词项。这种查询通常用于全文搜索。
>3. **Wildcard Query（通配符查询）**：使用通配符进行模糊匹配。通配符可以是 *（匹配任意多个字符）或 ?（匹配一个字符）。
>4. **Exists Query（存在查询）**：用于查找某个字段存在非空值的文档。
>5. **Terms Query（词项列表查询）**：类似于Term Query，但可以同时匹配多个值。
>6. **Org Match Query（原始匹配查询）**：在Elasticsearch 7.x 中新增的功能，与Match Query类似，但会保留搜索词的原始形式，而不进行分词。

# 查询索引信息

## 索引统计信息

>一般用来获取该索引的存储大小

### 查询全部索引统计信息

```shell
get /_cat/indices?v
```

### 查询一个或多个索引统计信息

>多个索引中间用逗号隔开

```shell
GET /a,b/_stats
```

## 索引映射信息

>一般用来获取索引的字段信息和字段个数

### 查询一个或多个索引的映射信息

```shell
GET /a,b/_mapping
```







# 数组操作

## array元素是string

```shell
# 创建一条数据
POST test_index/test_type/1
{
  "tags":["tag1", "tag2", "tag3"]  
}

# 查看数据
GET test_index/test_type/1


# 给 _id=1 的tags增加一个 tag5
POST test_index/test_type/1/_update
{
   "script" : {
       "source": "ctx._source.tags.add(params.tag)",
       "params" : {
          "tag" : "tag5"
       }
   }
}

# _id=1 的tags移除 tag5
POST test_index/test_type/1/_update
{
   "script" : {
       "source": "ctx._source.tags.remove(ctx._source.tags.indexOf(params.tag))",
       "params" : {
          "tag" : "tag5"
       }
   }
}

# 查询 tag1 in tags 移除tag2
POST test_index/test_type/_update_by_query
{
  "query": {
    "term": {
      "tags": "tag1"
    }
  },
  "script": {
    "source": "ctx._source.tags.remove(ctx._source.tags.indexOf(params.tag))",
    "params": {
      "tag": "tag2"
    }
  }
}
```

## Array里边包含对象

```shell
# 添加一条数据
POST /blog-test/doc/AXiHn9l3CFuw25Pb9kYM
{
  "title": "Invest Money",
  "body": "Please start investing money as soon...",
  "tags": [
    "money",
    "invest"
  ],
  "published_on": "18 Oct 2017",
  "comments": [
    {
      "name": "William",
      "age": 34,
      "rating": 8,
      "comment": "Nice article..",
      "commented_on": "30 Nov 2017"
    },
    {
      "name": "John",
      "age": 38,
      "rating": 9,
      "comment": "I started investing after reading this.",
      "commented_on": "25 Nov 2017"
    },
    {
      "name": "Smith",
      "age": 33,
      "rating": 7,
      "comment": "Very good post",
      "commented_on": "20 Nov 2017"
    }
  ]
}

# 数组添加一条数据
POST /blog-test/doc/AXiHn9l3CFuw25Pb9kYM/_update
{
  "script": {
    "source": "ctx._source.comments.add(params.new_comment)",
    "params": {
      "new_comment": {
        "name": "xiang",
        "age": 25,
        "rating": 18,
        "comment": "very very good article...",
        "commented_on": "3 Nov 2018"
      }
    }
  }
}



# 数组移除一条数据
POST /blog-test/doc/AXiHn9l3CFuw25Pb9kYM/_update
{
  "script": {
    "lang": "painless",
    "source": "ctx._source.comments.removeIf(it -> it.name == 'John');"
  }
}

# 数组更新一条数据
POST /blog-test/doc/AXiHn9l3CFuw25Pb9kYM/_update
{
  "script": {
    "source": "for(e in ctx._source.comments){if (e.name == 'Smith') {e.age = 25; e.comment= 'very very good article...';}}"
  }
}
```

