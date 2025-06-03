---
title: "ES安装使用"
date: 2019-06-09T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- databases
tags: 
- ES
---

# 安装使用

## 安装ElasticSearch：

https://www.elastic.co/cn/downloads/elasticsearch

>解决跨域问题

在config的elasticsearch.yml中添加

```yml
http.cors.enabled: "true"
http.cors.allow-origin: "*"
```

## 安装elasticsearch-head插件：

https://github.com/mobz/elasticsearch-head

- `git clone git://github.com/mobz/elasticsearch-head.git`
- `cd elasticsearch-head`
- `npm install`
- `npm run start`
- `open` http://localhost:9100/

## 安装kibana

https://www.elastic.co/cn/downloads/kibana

>### 设置中文

在config文件的配置yml中添加

```yml
i18n.locale: "zh-CN"
```

## 下载ik分词器

https://github.com/medcl/elasticsearch-analysis-ik/releases

解压到es的plugins文件中



> ## Elasticsearch：面向文档



# 使用rest指令操作es

## 中文分词使用ik

>### 下载解压到es的插件目录即可

>## 最小粒度

>```react
>GET _analyze
>{
>"analyzer": "ik_smart",
>"text": "中国共产党"
>}
>```
>
>
>
>```json
>{
>"tokens" : [
>{
> "token" : "中国共产党",
> "start_offset" : 0,
> "end_offset" : 5,
> "type" : "CN_WORD",
> "position" : 0
>}
>]
>}
>```
>
>

>## 最细粒度

>```react
>GET _analyze
>{
>"analyzer": "ik_max_word",
>"text": "中国共产党"
>}
>```
>
>
>
>```json
>{
>"tokens" : [
>{
> "token" : "中国共产党",
> "start_offset" : 0,
> "end_offset" : 5,
> "type" : "CN_WORD",
> "position" : 0
>},
>{
> "token" : "中国",
> "start_offset" : 0,
> "end_offset" : 2,
> "type" : "CN_WORD",
> "position" : 1
>},
>{
> "token" : "国共",
> "start_offset" : 1,
> "end_offset" : 3,
> "type" : "CN_WORD",
> "position" : 2
>},
>{
> "token" : "共产党",
> "start_offset" : 2,
> "end_offset" : 5,
> "type" : "CN_WORD",
> "position" : 3
>},
>{
> "token" : "共产",
> "start_offset" : 2,
> "end_offset" : 4,
> "type" : "CN_WORD",
> "position" : 4
>},
>{
> "token" : "党",
> "start_offset" : 4,
> "end_offset" : 5,
> "type" : "CN_CHAR",
> "position" : 5
>}
>]
>}
>```
>
>



## 自定义分词

> ### 在ik的config里面创建.dic结尾的文件，里面写入需要的词，在.xml里面添加这个.dic文件，重启es即可

## 英文分词使用es自带的

keyword：当作一个整体

standard：会被拆分

```
GET _analyze
{
  "analyzer": "keyword",
  "text": "狂神说java name"
}
```

		{
	  "tokens" : [
	    {
	      "token" : "狂神说java name",
	      "start_offset" : 0,
	      "end_offset" : 12,
	      "type" : "word",
	      "position" : 0
	    }
	  ]
	}

```
GET _analyze
{
  "analyzer": "standard",
  "text": "狂神说java name"
}
```

```
{
  "tokens" : [
    {
      "token" : "狂",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<IDEOGRAPHIC>",
      "position" : 0
    },
    {
      "token" : "神",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "<IDEOGRAPHIC>",
      "position" : 1
    },
    {
      "token" : "说",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "<IDEOGRAPHIC>",
      "position" : 2
    },
    {
      "token" : "java",
      "start_offset" : 3,
      "end_offset" : 7,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "name",
      "start_offset" : 8,
      "end_offset" : 12,
      "type" : "<ALPHANUM>",
      "position" : 4
    }
  ]
}
```



## 高亮

```react
GET /hzyy/user/_search
{
  "query": {
    "match": {
      "name": "狂神"
    }
  },
  "highlight": {
    "pre_tags": "<p class='key' style='color:red'>",
    "post_tags": "</p>", 
    "fields": {
      "name": {}
    }
  }
}

			结果：
      ...
        "highlight" : {
          "name" : [
            "<p class='key' style='color:red'>狂</p><p class='key' style='color:red'>神</p>说java"
          ]
        }
				...
        "highlight" : {
          "name" : [
            "<p class='key' style='color:red'>狂</p><p class='key' style='color:red'>神</p>说VUE"
          ]
        }
```





## 创建索引

> put /索引名/类型/文档id

```react
PUT /test1/type1/1	
{
  "name": "zhangsan",
  "age": 18
}
```

```json
{
  "_index" : "test1",
  "_type" : "type1",
  "_id" : "1",
  "_version" : 1,
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

| _index | _type | _id  | ▲_score | name     | age  |      |
| :----- | :---- | :--- | :------ | :------- | :--- | :--- |
| test1  | type1 | 1    | 1       | zhangsan | 18   |      |



## 创建索引规则

只有表没有数据

```json
PUT /test2
{
  "mappings": {
    "properties": {
      "name": {
      "type": "text"
      },
      "age": {
        "type": "long"
      },
      "birthday": {
        "type": "date"
      }
    }
  }
}
```



## 获取信息

模糊查询

```react
GET /hzyy/user/_search
{
  "query": {
    "match": {
      "name": "狂神"
    }
  },
  "_source": ["name","desc","age"]		//显示指定字段
}
```



排序

```react
GET /hzyy/user/_search
{
  "query": {
    "match": {
      "name": "狂神"
    }
  },
  "sort": [
    {
      "age": {
        "order": "asc"	//asc：升序，desc：降序
      }
    }
  ]
}
```



分页

```react
GET /hzyy/user/_search
{
  "query": {
    "match": {
      "name": "狂神"
    }
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    }
  ],
  "from": 0,	//从第几条开始
  "size": 1		//返回几条数据
}
```



## bool查询

must：and	条件全部符合

should：or	条件符合一个

must_not：not	条件不符合

"gte": 13,	gt:> gte:>=
 "lte": 20	lt:< lte:<=

```react
GET /hzyy/user/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "狂神"
          }
        },
        {
          "match": {
            "age": "12"
          }
        }
      ]
    }
  }
}

GET /hzyy/user/_search
{
  "query": {
    "bool": {
      "must_not": [
        {
          "match": {
            "age": "12"
          }
        }
      ]
    }
  }
}

GET /hzyy/user/_search
{
  "query": {
    "match": {
      "name": "狂神"
    }
  },
  "sort": [
    {
      "age": {
        "order": "asc"
      }
    }
  ]
}

GET /hzyy/user/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "狂神"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "age": {
              "gte": 13,
              "lte": 20
            }
          }
        }
      ]
    }
  }
}
```





GET /hzyy/user/_search?q=name:hzy

```react
GET test2
```

```json
{
  "test2" : {
    "aliases" : { },
    "mappings" : {
      "properties" : {
        "age" : {
          "type" : "long"
        },
        "birthday" : {
          "type" : "date"
        },
        "name" : {
          "type" : "text"
        }
      }
    },
    "settings" : {
      "index" : {
        "routing" : {
          "allocation" : {
            "include" : {
              "_tier_preference" : "data_content"
            }
          }
        },
        "number_of_shards" : "1",
        "provided_name" : "test2",
        "creation_date" : "1628610218722",
        "number_of_replicas" : "1",
        "uuid" : "MKEsFvDtTRef74KV8bQe_A",
        "version" : {
          "created" : "7130199"
        }
      }
    }
  }
}
```



类型不写默认为_doc，不指定字段类型，es会自动配置字段类型

```react
PUT /test3/_doc/1
{
  "name": "zhangsan",
  "age": 12,
  "birth": "2000-01-01"
}
```

```json
GET test3

keyword ： 不可分割类型

{
  "test3" : {
    "aliases" : { },
    "mappings" : {
      "properties" : {
        "age" : {
          "type" : "long"
        },
        "birth" : {
          "type" : "date"
        },
        "name" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    },
    "settings" : {
      "index" : {
        "routing" : {
          "allocation" : {
            "include" : {
              "_tier_preference" : "data_content"
            }
          }
        },
        "number_of_shards" : "1",
        "provided_name" : "test3",
        "creation_date" : "1628610458534",
        "number_of_replicas" : "1",
        "uuid" : "-QeDzMqLQ_W1oerXvp42ZA",
        "version" : {
          "created" : "7130199"
        }
      }
    }
  }
}
```



## 获取es系统信息

```react
GET _cat/indices?v
```



## 修改

```react
POST /test3/_doc/1/_update
{
  "doc": {
    "name": "hzyy"
  }
}
```



## 删除

```react
DELETE /test2
```



## es不允许修改索引库

### 但是可以添加新的字段

```react
PUT /索引库名/_mapping
{
	"properties": {
		"新的字段名": {
			"type": "类型"
		}
	}
}
```



## 文档操作

### 增加文档

```react
POST /docu1/_doc/1
{
  "字段一": "值一",
  "字段二": "值二",
  "字段三": {
    "子属性一": "值三",
    "子属性二": "值四"
  }
}

GET /docu1/_doc/1	查询

DELETE /docu1/_doc/1 删除
```

```json
{
  "_index" : "docu1",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "字段一" : "值一",
    "字段二" : "值二",
    "字段三" : {
      "子属性一" : "值三",
      "子属性二" : "值四"
    }
  }
}
```



## 修改文档

方式一：全量修改，会删除旧文档，添加新文档

id存在就修改，不存在就是添加

```react
PUT /索引库名/_doc/id
{
	"字段1": "值1",
	“字段二": "值2"，
	...
}
```



方式二：增量修改，修改指定字段值

```react
POST /索引库名/_update/id
{
	"doc": {
		"字段名": "新的值"
	}
}
```



# 
