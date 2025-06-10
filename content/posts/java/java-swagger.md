---
title: "Java Swagger"
date: 2019-07-04T21:11:25+08:00
author: ["loveyu"]
draft: false
categories: 
- java
tags: 
- java
- swagger
---

>访问地址


http://localhost:8080/swagger-ui/



```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```



```java
@EnableOpenApi
@Configuration
public class Swagger3Config {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.OAS_30)
                .pathMapping("/")
                // 定义是否开启swagger，false为关闭，可以通过变量控制
                .enable(true)
                // 将api的元信息设置为包含在json ResourceListing响应中。
                .apiInfo(apiInfo())
                // 接口调试地址
                .host("http://localhost:8080")
                // 选择哪些接口作为swagger的doc发布
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.xiaoyu.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    /** API 页面上半部分展示信息 */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Swagger3Demo")
                .description("一次简单的Demo")
                .contact(new Contact("loveYu233", "xxxxxxx", "1958387321@qq.com"))
                .version("1.0")
                .build();
    }
}
```



```java
@RestController
@Api(tags = "1.User测试类接口")
public class Test {

    @ApiOperation("get测试返回UserJson字符串")
    @GetMapping("/getUser")
    public String t1() {
        User user = new User("1", "zhangsan", "qwer");
        return JSON.toJSON(user).toString();
    }

    @ApiOperation("post测试返回UserJson字符串")
    @PostMapping("/postUser")
    public String t2() {
        User user = new User("1", "zhangsan", "qwer");
        return JSON.toJSON(user).toString();
    }
}
```

