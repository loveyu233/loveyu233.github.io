---
title: "springBoot操作ES"
date: 2019-06-24T09:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- java
tags: 
- ES
- springBoot
---

# SpringBoot操作ES

## 导入依赖：ES依赖和阿里json依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>

<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>fastjson</artifactId>
  <version>1.2.75</version>
</dependency>
```

## 配置bean

>不配置也可以，使用默认的

```java
@Configuration()
public class EsConfig {
    @Bean
    public RestHighLevelClient restHighLevelClient() {
        return new RestHighLevelClient(
                RestClient.builder(
                        new HttpHost("localhost", 9200, "http")));
    }
}
```

## 实现增删改查

```java
@Autowired
private RestHighLevelClient client;
```

### 创建索引

```java
@Test
void t1() throws IOException {
  CreateIndexRequest request = new CreateIndexRequest("hzy_index");
  CreateIndexResponse response = client.indices().create(request, RequestOptions.DEFAULT);
  System.out.println(response);
}
```



### 判断索引是否存在

```java
@Test
void t2() throws IOException {
    GetIndexRequest request = new GetIndexRequest("hzy_index");
    boolean exists = client.indices().exists(request, RequestOptions.DEFAULT);
    System.out.println(exists);
}
```



### 删除索引

```java
@Test
void t3() throws IOException {
    DeleteIndexRequest request = new DeleteIndexRequest("hzy_index");
    AcknowledgedResponse response = client.indices().delete(request, RequestOptions.DEFAULT);
    System.out.println(response.isAcknowledged());
}
```



### 添加文档

```java
@Test
void t4() throws IOException {
    User user = new User("狂神说", 12);
    IndexRequest request = new IndexRequest("hzy_index");
    request.id("1").timeout("1s").source(JSON.toJSONString(user), XContentType.JSON);
    IndexResponse response = client.index(request, RequestOptions.DEFAULT);
    System.out.println("返回消息：" + response.toString());
    System.out.println("状态：" + response.status());
}
```



### 获取文档

```java
@Test
void t5() throws IOException {
  GetRequest request = new GetRequest("hzy_index", "1");
  //        不获取返回的 _source的上下文
  request.fetchSourceContext(new FetchSourceContext(false));
  request.storedFields("_none_");
  //        获取文档是否存在
  boolean exists = client.exists(request, RequestOptions.DEFAULT);
  System.out.println(exists);
}
```




### 获取文档信息

```java
@Test
void t6() throws IOException {
    GetRequest request = new GetRequest("hzy_index", "1");
    GetResponse response = client.get(request, RequestOptions.DEFAULT);
    System.out.println(response.getSourceAsString());
    System.out.println(response);
}
```



### 更新文档

```java
@Test
void t7() throws IOException {
    UpdateRequest request = new UpdateRequest("hzy_index", "1");
    User user = new User("狂神说", 22);
    request.timeout("1s").doc(JSON.toJSONString(user), XContentType.JSON);
    UpdateResponse response = client.update(request, RequestOptions.DEFAULT);
    System.out.println(response);
    System.out.println(response.status());
}
```



### 删除文档

```java
@Test
void t8() throws IOException {
    DeleteRequest request = new DeleteRequest("hzy_index", "1");
    request.timeout("1s");
    DeleteResponse response = client.delete(request, RequestOptions.DEFAULT);
    System.out.println(response);
    System.out.println(response.status());
}
```



### 批量插入

```java
@Test
void t9() throws IOException {
    BulkRequest bulkRequest = new BulkRequest();
    ArrayList<User> users = new ArrayList<>();
    users.add(new User("a", 1));
    users.add(new User("b", 2));
    users.add(new User("c", 3));
    users.add(new User("d", 4));
    users.add(new User("e", 5));
    users.add(new User("apple", 6));
    users.add(new User("app", 7));
    users.add(new User("ai", 8));
    bulkRequest.timeout("6s");
    for (int i = 0; i < users.size(); i++) {
        bulkRequest.add(new IndexRequest("hzy_index")
                .id("" + i + 1)
                .source(JSON.toJSONString(users.get(i)), XContentType.JSON));
    }
    BulkResponse response = client.bulk(bulkRequest, RequestOptions.DEFAULT);
    System.out.println(response);
    System.out.println(response.hasFailures()); //是否失败
}
```



### 搜索和高亮

```java
@Test
void t10() throws IOException {
    SearchRequest request = new SearchRequest("biandata");
    SearchSourceBuilder builder = new SearchSourceBuilder();
    MatchQueryBuilder query = QueryBuilders.matchQuery("title", "白蛇");
    builder.query(query).timeout(new TimeValue(10, TimeUnit.SECONDS));

    HighlightBuilder highlightBuilder = new HighlightBuilder();
    highlightBuilder.field("title")
            .requireFieldMatch(false)
            .preTags("<p style='color:red'>")
            .postTags("</p>");
    builder.highlighter(highlightBuilder);

    request.source(builder);
    SearchResponse search = client.search(request, RequestOptions.DEFAULT);
    System.out.println(JSON.toJSONString(search.getHits()));
    System.out.println("=========================");
    for (SearchHit hit : search.getHits().getHits()) {
        Map<String, HighlightField> highlightFields = hit.getHighlightFields();
        HighlightField name = highlightFields.get("title");
        Map<String, Object> sourceAsMap = hit.getSourceAsMap();
        if (name != null) {
            Text[] fragments = name.fragments();
            for (Text fragment : fragments) {
                sourceAsMap.put("title",fragment);
            }
        }
        System.out.println(sourceAsMap);
    }
}
```

