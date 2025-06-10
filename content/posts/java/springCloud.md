---
title: "SpringCloud"
date: 2019-12-01T11:18:26+08:00
author: ["loveyu"]
draft: false
categories: 
- java
tags: 
- java
- springCloud
- springCloudAlibaba
---

#  Spring Cloud Alibaba

## 介绍

Spring Cloud Alibaba 致力于提供微服务开发的一站式解决方案。此项目包含开发分布式应用微服务的必需组件，方便开发者通过 Spring Cloud 编程模型轻松使用这些组件来开发分布式应用服务。

依托 Spring Cloud Alibaba，您只需要添加一些注解和少量配置，就可以将 Spring Cloud 应用接入阿里微服务解决方案，通过阿里中间件来迅速搭建分布式应用系统。

## 主要功能

- **服务限流降级**：默认支持 WebServlet、WebFlux, OpenFeign、RestTemplate、Spring Cloud Gateway, Zuul, Dubbo 和 RocketMQ 限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控。
- **服务注册与发现**：适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持。
- **分布式配置管理**：支持分布式系统中的外部化配置，配置更改时自动刷新。
- **消息驱动能力**：基于 Spring Cloud Stream 为微服务应用构建消息驱动能力。
- **分布式事务**：使用 @GlobalTransactional 注解， 高效并且对业务零侵入地解决分布式事务问题。
- **阿里云对象存储**：阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- **分布式任务调度**：提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀分配到所有 Worker（schedulerx-client）上执行。
- **阿里云短信服务**：覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道

## 组件

- **[Sentinel](https://github.com/alibaba/Sentinel)**：把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

- **[Nacos](https://github.com/alibaba/Nacos)**：一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

- **[RocketMQ](https://rocketmq.apache.org/)**：一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。

- **[Dubbo](https://github.com/apache/dubbo)**：Apache Dubbo™ 是一款高性能 Java RPC 框架。

- **[Seata](https://github.com/seata/seata)**：阿里巴巴开源产品，一个易于使用的高性能微服务分布式事务解决方案。

- **[Alibaba Cloud OSS](https://www.aliyun.com/product/oss)**: 阿里云对象存储服务（Object Storage Service，简称 OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。您可以在任何应用、任何时间、任何地点存储和访问任意类型的数据。

- **[Alibaba Cloud SchedulerX](https://help.aliyun.com/document_detail/43136.html)**: 阿里中间件团队开发的一款分布式任务调度产品，提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。

- **[Alibaba Cloud SMS](https://www.aliyun.com/product/sms)**: 覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。

## 如何使用

### 如何引入依赖

如果需要使用已发布的版本，在 `dependencyManagement` 中添加如下配置。

```xml
	<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2.2.5.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

然后在 `dependencies` 中添加自己所需使用的依赖即可使用。

## 演示 Demo

为了演示如何使用，Spring Cloud Alibaba 项目包含了一个子模块`spring-cloud-alibaba-examples`。此模块中提供了演示用的 example ，您可以阅读对应的 example 工程下的 readme 文档，根据里面的步骤来体验。

Example 列表：

[Sentinel Example](https://github.com/alibaba/spring-cloud-alibaba/tree/master/spring-cloud-alibaba-examples/sentinel-example/sentinel-core-example/readme-zh.md)

[Nacos Config Example](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-examples/nacos-example/nacos-config-example/readme-zh.md)

[Nacos Discovery Example](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-examples/nacos-example/nacos-discovery-example/readme-zh.md)

[RocketMQ Example](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-examples/rocketmq-example/readme-zh.md)

[Seata Example](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-examples/seata-example/readme-zh.md)

[Alibaba Cloud OSS Example](https://github.com/alibaba/aliyun-spring-boot/tree/master/aliyun-spring-boot-samples/aliyun-oss-spring-boot-sample)

[Alibaba Cloud SMS Example](https://github.com/alibaba/aliyun-spring-boot/tree/master/aliyun-spring-boot-samples/aliyun-sms-spring-boot-sample)

[Alibaba Cloud SchedulerX Example](https://github.com/alibaba/aliyun-spring-boot)

## 版本管理规范

项目的版本号格式为 x.x.x 的形式，其中 x 的数值类型为数字，从 0 开始取值，且不限于 0~9 这个范围。项目处于孵化器阶段时，第一位版本号固定使用 0，即版本号为 0.x.x 的格式。

由于 Spring Boot 1 和 Spring Boot 2 在 Actuator 模块的接口和注解有很大的变更，且 spring-cloud-commons 从 1.x.x 版本升级到 2.0.0 版本也有较大的变更，因此我们采取跟 SpringBoot 版本号一致的版本:

- 1.5.x 版本适用于 Spring Boot 1.5.x
- 2.0.x 版本适用于 Spring Boot 2.0.x
- 2.1.x 版本适用于 Spring Boot 2.1.x
- 2.2.x 版本适用于 Spring Boot 2.2.x
- 2021.x 版本适用于 Spring Boot 2.4.x

nacos开启：sh docker-startup.sh -m standalone

spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://120.26.11.225:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=hzy1214..

java -jar -Dservice.port=8080 /Users/huangzhenyu/Downloads/sentinel-dashboard-1.8.1.jar

# nacos       

## mysql版本高错误：

### 在nacos安装目录下创建/plugins/mysql文件夹，在其中放入8.0jar包

>启动
>
>sh startup.sh -m standalone

## 基本使用

### 服务方：

1.pom.xml

```xml
<dependencies>
    <!--SpringCloud ailibaba nacos -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <!-- SpringBoot整合Web组件 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <!--日常通用jar包配置-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

2.yml

```yml
server:
  port: 9001

spring:
  application:
    name: cloudalibaba-provider-payment


  cloud:
    nacos:
      discovery:
        server-addr: 120.26.11.225:8848

management:
  endpoints:
    web:
      exposure:
        include: '*'
```

3.启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class Cloudalibaba_provider_payment9001 {
    public static void main(String[] args) {
        SpringApplication.run(Cloudalibaba_provider_payment9001.class,args);
    }
}
```

3.service

```java
@RestController
public class PaymentController {

    @Value("${server.port}")
    private String serverPort;

    @GetMapping("/payment/nacos/{id}")
    String getPayment(@PathVariable("id") Integer id) {
        return "nacos registry, server port : " + serverPort + ", id : " + id;
    }
}
```

### 消费方：

#### 使用restTemplate

```xml
<dependencies>
    <!--SpringCloud ailibaba nacos -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
    <dependency>
        <groupId>org.example</groupId>
        <artifactId>cloud-api-commons</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
    <!-- SpringBoot整合Web组件 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <!--日常通用jar包配置-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

2.yml

```yml
server:
  port: 83

spring:
  application:
    name: cloudalibaba-consumer-order-nacos

  cloud:
    nacos:
      discovery:
        server-addr: 120.26.11.225:8848

service-url:
  nacos-user-service: http://cloudalibaba-provider-payment
```

3.启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class Cloudalibaba_consumer_order_nacos83 {
    public static void main(String[] args) {
        SpringApplication.run(Cloudalibaba_consumer_order_nacos83.class,args);
    }
}
```

4.config

```java
@Configuration
public class RestTemplateConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

5.controller

```java
@RestController
public class OrderController {

    @Resource
    private RestTemplate restTemplate;

    @Value("${service-url.nacos-user-service}")
    private String serviceURL;

    @GetMapping("/consumer/paymenr/nacos/{id}")
    String nacosPayment(@PathVariable("id") Integer id) {
        return restTemplate.getForObject(serviceURL+"/payment/nacos/"+id,String.class);
    }

}
```

#### 使用OpenFeign

pom.xml

```
<dependency>
<groupId>com.alibaba.cloud</groupId>
<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>

<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

```java
@FeignClient(name = "SpringCloudAlibabaPro")
public interface DemoFeign {
    @GetMapping("/nacos/{id}")
    String t1 (@PathVariable("id") Integer id);
}
```

```java
@RestController
public class DemoController {

    @Autowired
    DemoFeign demoFeign;

    @GetMapping("/get/{id}")
    public String t1 (@PathVariable("id") Integer id) {
        return demoFeign.t1(id);
    }
}
```

>配置日志信息

```java
//@Configuration     添加注解会将作用所有的服务提供方
//如果是只想针对某一个服务就不需要加

@Bean
Logger.Level feign() {
    /**
     * NONE:性能最佳适用于生产环境，不记录任何日志(默认值)
     * BASIC:适用于成产环境的追踪问题，仅记录请求方式，URL，响应码以及执行时间
     * HEADERS:记录请求和响应头信息
     * FULL:适用于测试环境，记录请求和响应的头信息，body和元数据
     */
    return Logger.Level.FULL;
}


logging:
  level:
    com.xiaoyu.service: debug
```

>配置超时时间

```java
/**
 * 全剧配置超时时间针对所有的服务
 * 只针对某一个服务的话在yml配置
 * @return
 */
@Bean
public Request.Options options () {
    return new Request.Options(1000,1000);
}


#针对某一个服务的超时时间设置
#全局和局部都配置的话生效的是局部配置
feign:
  client:
    config:
      SpringCloudAlibabaPro:
        #默认2s
        connectTimeout: 3000
#       默认5s
        readTimeout: 3000
```



### 配置中心：

<img src="../../../img/apringCloudNacos.png" alt="acl" style="zoom:100%;" />

```xml
<dependencies>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    </dependency>
    <!--nacos-discovery-->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <!--web + actuator-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <!--一般基础配置-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

2.yml

application.yml

```yml
spring:
  profiles:
    active: dev
```

bootstrap.yml

```yml
server:
  port: 3377

spring:
  application:
    name: nacos-config

  cloud:
    nacos:
      discovery:
        server-addr: 120.26.11.225:8848
      config:
        server-addr: 120.26.11.225:8848
        file-extension: yaml
        max-retry: 10
        namespace: 123456
        group: DEV-GROUP

# ${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}
# 结果是：nacos-config-dev.yaml
```

3.启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class Cloudalibaba_config_nacos3377 {
    public static void main(String[] args) {
        SpringApplication.run(Cloudalibaba_config_nacos3377.class,args);
    }
}
```

4.controller

```java
@RestController
@RefreshScope
public class ConfigController {
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config/info")
    String getConfigInfo() {
        return configInfo;
    }
}
```

<img src="../../../img/springCloudGroup.png" style="zoom:100%;" />

## nacos集群搭建

查看当前启动多少台nacos

ps -ef|grep nacos|grep -v grep|wc -l



# sentinel

分布式系统的流量防卫兵

<img src="../../../img/sentinel.png" alt="acl" style="zoom:100%;" />

## 流控规则

<img src="../../../img/liukong.png" alt="acl" style="zoom:100%;" />

流控页面提示信息为：Blocked by Sentinel (flow limiting)

###  资源名称 ：

#### rest路径例如：/testA

###  QPS ：

#### 每秒的并发量

###  线程数 ：

#### 同时访问的线程量

###  单机阈值 ：

#### 每秒的并发量/同时访问的线程量的最大值

###  直接 ：

#### 默认只针对当前的资源

###  关联 ：

#### 当资源名的资源达到设置的规则后，访问关联的资源会被限流

###  链路 ：

#### 以调用链路为单位做限流处理，例如：A->B->C 这个链路的总体流量只按入口A的请求量来计算

###  快速失败 ：

#### 就是超过这个阈值就直接显示错误页面

###  WarmUp ：

#### 预热，设置预热时间，刚开始的访问量是设置阈值/3，当达到预热时间后会恢复到设置的阈值

###  排队等待 ：

#### 大于并发量可以进入但是要排队等待

## 服务熔断降级规则

<img src="../../../img/rongduan.png" alt="acl" style="zoom:100%;" />

服务降级后提示信息为：Blocked by Sentinel (flow limiting)

###  慢调用比例 ：

#### 当 统计时间内 的 达到最小请求数 且每条 请求数的处理时间达到最大RT的比例阈值 会进行 服务熔断 ，服务熔断的时间为设置的 熔断时长 

<img src="../../../img/cuowubilie.png" alt="acl" style="zoom:100%;" />

###  异常比例 ：

#### 当在 统计时长内 的 最小请求数 的 请求出现错误的比例 达到设置 比例阈值 就会 进行服务熔断 ，时间为 熔断时长 

<img src="../../../img/yichangbilie.png" alt="acl" style="zoom:100%;" />



###  异常数 ：

#### 当在 统计时长内 的 最小请求数 的 请求出现错误的比例 达到设置 异常数 就会进行 服务熔断 ，时间为 熔断时长 

## 热点规则

<img src="../../../img/redian.png" alt="acl" style="zoom:100%;" />

热点达到阈值后出现页面设置,相当于ex_testD是testD的兜底方法

```java
    @GetMapping("/testD")
    @SentinelResource(value = "testD",blockHandler = "ex_testD")
    public String testD(@RequestParam(value = "p1",required = false) String p1,
                        @RequestParam(value = "p2",required = false) String p2) {
        return "testD";
    }

    public String ex_testD(String p1, String p2, BlockException blockException) {
        return "ex_testD";
    }
```



###  资源名 ：

#### rest路径

###  参数索引 ：

#### 例如：localhost:/user/zhangsan?username=aaa&password=hahhah,参数索引0测试username依次类推

###  单机阈值 ：

#### 访问最大值

###  统计窗口时长 ：

#### 规定在多少秒内达到单机阈值生效

###  参数类型 ：

#### 基本的八大数据类型，int，string，double等

###  =参数值 ：

#### 参数索引的值

###  限定阈值 ：

#### 参数索引的值为设置的参数值时的限流阈值

## 系统保护规则

<img src="../../../img/xitongbaohu.png" alt="acl" style="zoom:100%;" />

###  LOAD ：

#### 自适应，仅对linux/unix-like机器生效，设定的参考值为CPU核数*2.5

###  CPU ：

#### 当系统CPU使用率达到阈值生效

###  平均RT ：

#### 当单台机器上所有入口流量的平均RT达到阈值生效单位毫秒

###  并发线程数 ：

#### 当单台机器上所有入口流量的并发线程数达到阈值生效

###  入口QPS ：

#### 当单台机器上所有入口流量的QPS达到阈值生效

## @SentinelResource

```java
    /*
    blockHandler:服务配置规格异常的兜底
    fallback负责业务逻辑报错的兜底
    exceptionsToIgnore当匹配这个错误的时候不会调用兜底方法
    
    如果方法不在同一个类，则使用以下参数进行配置
     blockHandlerClass = ,
     fallbackClass = 
     */
@SentinelResource(value = "fallback",
        fallback = "handlerFallback",
        blockHandler = "blockHandler",
        exceptionsToIgnore = {RuntimeException.class})
```

## sentinel持久化

- resource：资源名称

- limitApp：来源应用

- grade：阈值类型
  - 0：表示线程数
  - 1：表示QPS

- count：单机阈值

- strategy：流控模式
  - 0：表示直接
  - 1：表示关联
  - 2：表示链路

- controlBehavior：流控效果
  - 0：表示快速失败
  - 1：表示Warm Up
  - 2：表示排队等待

- clusterMode：是否集群
  - true
  - false

<img src="../../../img/sentinelzhijiuhua.png" alt="acl" style="zoom:100%;" />

<img src="../../../img/sentinelyaml.png" alt="acl" style="zoom:100%;" />



# springCloudSeata

> 下载seata

> 在seata目录下创建config.txt

```bash
nsport.type=TCP
transport.server=NIO
transport.heartbeat=true
transport.enableClientBatchSendRequest=true
transport.threadFactory.bossThreadPrefix=NettyBoss
transport.threadFactory.workerThreadPrefix=NettyServerNIOWorker
transport.threadFactory.serverExecutorThreadPrefix=NettyServerBizHandler
transport.threadFactory.shareBossWorker=false
transport.threadFactory.clientSelectorThreadPrefix=NettyClientSelector
transport.threadFactory.clientSelectorThreadSize=1
transport.threadFactory.clientWorkerThreadPrefix=NettyClientWorkerThread
transport.threadFactory.bossThreadSize=1
transport.threadFactory.workerThreadSize=default
transport.shutdown.wait=3
service.vgroupMapping.my_test_tx_group=default
service.default.grouplist=127.0.0.1:8091
service.enableDegrade=false
service.disableGlobalTransaction=false
client.rm.asyncCommitBufferLimit=10000
client.rm.lock.retryInterval=10
client.rm.lock.retryTimes=30
client.rm.lock.retryPolicyBranchRollbackOnConflict=true
client.rm.reportRetryCount=5
client.rm.tableMetaCheckEnable=false
client.rm.tableMetaCheckerInterval=60000
client.rm.sqlParserType=druid
client.rm.reportSuccessEnable=false
client.rm.sagaBranchRegisterEnable=false
client.rm.tccActionInterceptorOrder=-2147482648
client.tm.commitRetryCount=5
client.tm.rollbackRetryCount=5
client.tm.defaultGlobalTransactionTimeout=60000
client.tm.degradeCheck=false
client.tm.degradeCheckAllowTimes=10
client.tm.degradeCheckPeriod=2000
client.tm.interceptorOrder=-2147482648
store.lock.mode=file
store.session.mode=file
store.publicKey=
store.file.dir=file_store/data
store.file.maxBranchSessionSize=16384
store.file.maxGlobalSessionSize=512
store.file.fileWriteBufferCacheSize=16384
store.file.flushDiskMode=async
store.file.sessionReloadReadSize=100
store.db.datasource=druid
store.db.dbType=mysql
store.db.driverClassName=com.mysql.jdbc.Driver
store.db.url=jdbc:mysql://localhost:3306/seata?characterEncoding=utf8&useSSL=false&serverTimezone=UTC&rewriteBatchedStatements=true
store.db.user=root
store.db.password=Hzyy1214
store.db.minConn=5
store.db.maxConn=30
store.db.globalTable=global_table
store.db.branchTable=branch_table
store.db.queryLimit=100
store.db.lockTable=lock_table
store.db.maxWait=5000
store.redis.mode=single
store.redis.single.host=127.0.0.1
store.redis.single.port=6379
store.redis.sentinel.masterName=
store.redis.sentinel.sentinelHosts=
store.redis.maxConn=10
store.redis.minConn=1
store.redis.maxTotal=100
store.redis.database=0
store.redis.password=
store.redis.queryLimit=100
server.recovery.committingRetryPeriod=1000
server.recovery.asynCommittingRetryPeriod=1000
server.recovery.rollbackingRetryPeriod=1000
server.recovery.timeoutRetryPeriod=1000
server.maxCommitRetryTimeout=-1
server.maxRollbackRetryTimeout=-1
server.rollbackRetryTimeoutUnlockEnable=false
server.distributedLockExpireTime=10000
client.undo.dataValidation=true
client.undo.logSerialization=jackson
client.undo.onlyCareUpdateColumns=true
server.undo.logSaveDays=7
server.undo.logDeletePeriod=86400000
client.undo.logTable=undo_log
client.undo.compress.enable=true
client.undo.compress.type=zip
client.undo.compress.threshold=64k
log.exceptionRate=100
transport.serialization=seata
transport.compressor=none
metrics.enabled=false
metrics.registryType=compact
metrics.exporterList=prometheus
metrics.exporterPrometheusPort=9898
```

>上述配置解读：
>
>service.vgroupMapping.my_test_tx_group=default	配置信息到nacos的分组默认default
>
>service.default.grouplist=127.0.0.1:8091	seata的ip端口
>
>```
>配置数据库
>store.db.datasource=druid
>store.db.dbType=mysql
>store.db.driverClassName=com.mysql.jdbc.Driver
>store.db.url=jdbc:mysql://localhost:3306/seata?characterEncoding=utf8&useSSL=false&serverTimezone=UTC&rewriteBatchedStatements=true
>store.db.user=root
>store.db.password=Hzyy1214
>```
>
>
>
>```
>数据库的表默认名称即可，如果数据库表修改这里也要修改
>store.db.globalTable=global_table
>store.db.branchTable=branch_table
>store.db.queryLimit=100
>store.db.lockTable=lock_table
>```
>
>



>在conf目录创建nacos-config.sh脚本用于提交config.txt的配置信息到nacos

```shell
opyright 1999-2019 Seata.io Group.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at、
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

while getopts ":h:p:g:t:u:w:" opt
do
  case $opt in
  h)
    host=$OPTARG
    ;;
  p)
    port=$OPTARG
    ;;
  g)
    group=$OPTARG
    ;;
  t)
    tenant=$OPTARG
    ;;
  u)
    username=$OPTARG
    ;;
  w)
    password=$OPTARG
    ;;
  ?)
    echo " USAGE OPTION: $0 [-h host] [-p port] [-g group] [-t tenant] [-u username] [-w password] "
    exit 1
    ;;
  esac
done

urlencode() {
  for ((i=0; i < ${#1}; i++))
  do
    char="${1:$i:1}"
    case $char in
    [a-zA-Z0-9.~_-]) printf $char ;;
    *) printf '%%%02X' "'$char" ;;
    esac
  done
}

if [[ -z ${host} ]]; then
    host=localhost
fi
if [[ -z ${port} ]]; then
    port=8848
fi
if [[ -z ${group} ]]; then
    group="SEATA_GROUP"
fi
if [[ -z ${tenant} ]]; then
    tenant=""
fi
if [[ -z ${username} ]]; then
    username=""
fi
if [[ -z ${password} ]]; then
    password=""
fi

nacosAddr=$host:$port
contentType="content-type:application/json;charset=UTF-8"

echo "set nacosAddr=$nacosAddr"
echo "set group=$group"

failCount=0
tempLog=$(mktemp -u)
function addConfig() {
  curl -X POST -H "${contentType}" "http://$nacosAddr/nacos/v1/cs/configs?dataId=$(urlencode $1)&group=$group&content=$(urlencode $2)&tenant=$tenant&username=$username&password=$password" >"${tempLog}" 2>/dev/null
  if [[ -z $(cat "${tempLog}") ]]; then
    echo " Please check the cluster status. "
    exit 1
  fi
  if [[ $(cat "${tempLog}") =~ "true" ]]; then
    echo "Set $1=$2 successfully "
  else
    echo "Set $1=$2 failure "
    (( failCount++ ))
  fi
}

count=0
for line in $(cat $(dirname "$PWD")/config.txt | sed s/[[:space:]]//g); do
  (( count++ ))
	key=${line%%=*}
    value=${line#*=}
	addConfig "${key}" "${value}"
done

echo "========================================================================="
echo " Complete initialization parameters,  total-count:$count ,  failure-count:$failCount "
echo "========================================================================="

if [[ ${failCount} -eq 0 ]]; then
	echo " Init nacos config finished, please start seata-server. "
else
	echo " init nacos config fail. "
fi
```



> 修改配置文件file.config
>
> 在 file.config中 store.db.driverClassName 默认是 com.mysql.jdbc.Driver。 mysql8.0是 com.mysql.cj.jdbc.Drive

>执行命令提交配置信息
>
>```shell
>sh nacos-config.sh ip
>```

>启动seata
>
>```shell
>sh seata-server.sh -h nacos的ip地址
>```



## 是什么？

- 1+3

- 1：transactionID XID 全局唯一的事物ID

- 3：3组件概念：
  - TC (Transaction Coordinator) - 事务协调者
    - 维护全局和分支事务的状态，驱动全局事务提交或回滚。
  - TM (Transaction Manager) - 事务管理器
    - 定义全局事务的范围：开始全局事务、提交或回滚全局事务。
  - RM (Resource Manager) - 资源管理器
    - 管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

## 处理过程：

- TM想TC申请开启一个全局事务，全局事务创建成功并生成 一个全局唯一的XID 
- XID在微服务调用链路的上下文中传播
- RM向TC注册分支事务，将其纳入XID对应全局事务的管辖
- TM向TC发起针对XID的全局提交或回滚决议
- TC调度XID下管辖的全部分支事务完成提交或回滚请求



| 分布式事务模式 | 介绍                                                         | 技术栈                              |
| -------------- | ------------------------------------------------------------ | ----------------------------------- |
| **AT 模式**    | 无侵入的分布式事务解决方案，适用于不希望对业务进行改造的场景，几乎0学习成本（sql都由框架托管统一执行，会存在**脏写**问题） | seata、shardingsphere               |
| **TCC 模式**   | 高性能分布式事务解决方案，适用于核心系统等对性能有很高要求的场景（第一阶段会产生行锁，事务执行太久会**锁行很久**） | seata、service-comb                 |
| **Saga 模式**  | 长事务解决方案，适用于业务流程长且需要保证事务最终一致性的业务系统（第一阶段就操作DB，会存在**脏读**问题） | seata、shardingsphere、service-comb |
| **XA模式**     | 分布式强一致性的解决方案，但**性能低**而使用较少。           | seata、shardingsphere               |

启动命令：sh seata-server.sh -p 9999 -h 120.26.11.225

后台启动：nohup  sh seata-server.sh -p 9999 -h 120.26.11.225 -m file  >nohup.out 2>1 &

需要修改conf文件夹的file.conf改为db和配置数据库连接信息，具体建数据库信息如下，registry.conf修改为nacos配置nacos信息

创建数据库和表，如图

![数据库](../../../../../%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/%E6%96%87%E6%9C%AC%E7%AC%94%E8%AE%B0/otherImg/redis/%E6%95%B0%E6%8D%AE%E5%BA%93.png)

sql语句：

seata数据库：

```sql
-- -------------------------------- The script used when storeMode is 'db' --------------------------------
-- the table to store GlobalSession data
CREATE DATABASE seate;
USE seate;
CREATE TABLE IF NOT EXISTS `global_table`
(
    `xid`                       VARCHAR(128) NOT NULL,
    `transaction_id`            BIGINT,
    `status`                    TINYINT      NOT NULL,
    `application_id`            VARCHAR(32),
    `transaction_service_group` VARCHAR(32),
    `transaction_name`          VARCHAR(128),
    `timeout`                   INT,
    `begin_time`                BIGINT,
    `application_data`          VARCHAR(2000),
    `gmt_create`                DATETIME,
    `gmt_modified`              DATETIME,
    PRIMARY KEY (`xid`),
    KEY `idx_gmt_modified_status` (`gmt_modified`, `status`),
    KEY `idx_transaction_id` (`transaction_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- the table to store BranchSession data
CREATE TABLE IF NOT EXISTS `branch_table`
(
    `branch_id`         BIGINT       NOT NULL,
    `xid`               VARCHAR(128) NOT NULL,
    `transaction_id`    BIGINT,
    `resource_group_id` VARCHAR(32),
    `resource_id`       VARCHAR(256),
    `branch_type`       VARCHAR(8),
    `status`            TINYINT,
    `client_id`         VARCHAR(64),
    `application_data`  VARCHAR(2000),
    `gmt_create`        DATETIME(6),
    `gmt_modified`      DATETIME(6),
    PRIMARY KEY (`branch_id`),
    KEY `idx_xid` (`xid`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- the table to store lock data
CREATE TABLE IF NOT EXISTS `lock_table`
(
    `row_key`        VARCHAR(128) NOT NULL,
    `xid`            VARCHAR(128),
    `transaction_id` BIGINT,
    `branch_id`      BIGINT       NOT NULL,
    `resource_id`    VARCHAR(256),
    `table_name`     VARCHAR(32),
    `pk`             VARCHAR(36),
    `gmt_create`     DATETIME,
    `gmt_modified`   DATETIME,
    PRIMARY KEY (`row_key`),
    KEY `idx_branch_id` (`branch_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;
```

seata_account/order/storage数据库：

```sql
CREATE DATABASE seata_order;
USE seata_order;
CREATE TABLE t_order(
    id BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY ,
    user_id BIGINT(11) DEFAULT NULL COMMENT '用户id',
    product_id BIGINT(11) DEFAULT NULL COMMENT '产品id',
    count INT(11) DEFAULT NULL COMMENT '数量',
    money DECIMAL(11,0) DEFAULT NULL COMMENT '金额',
    status INT(1) DEFAULT NULL COMMENT '订单状态：0创建中，1已完结'
)ENGINE=InnoDB AUTO_INCREMENT=7 CHARSET=utf8;

CREATE DATABASE seata_storage;
USE seata_storage;
CREATE TABLE t_storage(
    id BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY ,
    product_id BIGINT(11) DEFAULT NULL COMMENT '产品id',
    total INT(11) DEFAULT NULL COMMENT '总库存',
    used INT(11) DEFAULT NULL COMMENT '已用库存',
    residue INT(11) DEFAULT NULL COMMENT '剩余库存'
)ENGINE=InnoDB AUTO_INCREMENT=7 CHARSET=utf8;
INSERT INTO t_storage(id, product_id, total, used, residue) VALUES(1,1,100,0,100);

CREATE DATABASE seata_account;
USE seata_account;
CREATE TABLE t_account(
    id BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY ,
    user_id BIGINT(11) DEFAULT NULL COMMENT '用户id',
    total DECIMAL(10,0) DEFAULT NULL COMMENT '总额度',
    used DECIMAL(10,0) DEFAULT NULL COMMENT '已用额度',
    residue DECIMAL(10,0) DEFAULT 0 COMMENT '剩余可用额度'
)ENGINE=InnoDB AUTO_INCREMENT=7 CHARSET=utf8;
INSERT INTO t_account(id, user_id, total, used, residue) VALUES(1,1,1000,0,1000);
```

## pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <!--feign-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
    <!--web-actuator-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--mysql-druid-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.23</version>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.1.10</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.0.0</version>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.0.5</version>
    </dependency>
</dependencies>
```

## application.yml

```yml
seata:
  enabled: true
  enable-auto-data-source-proxy: true
  tx-service-group: my_test_tx_group
  registry:
    type: nacos
    nacos:
      application: seata-server
      server-addr: 127.0.0.1:8848
      username: nacos
      password: nacos
  config:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8848
      group: SEATA_GROUP
      username: nacos
      password: nacos
      namespace: 0af6e97b-a684-4647-b696-7c6d42aecce7
  service:
    vgroupMapping:
      my_test_tx_group: default
    disable-global-transaction: false
  client:
    rm:
      report-success-enable: false
spring:
  application:
    name: Seata
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/seata_order?characterEncoding=utf8&useSSL=false&serverTimezone=UTC&rewriteBatchedStatements=true
    username: root
    password: Hzyy1214

server:
  port: 8999

```

## dataSource

```java
package com.xiaoyu.config;

import com.zaxxer.hikari.HikariDataSource;
import io.seata.rm.datasource.DataSourceProxy;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

import javax.sql.DataSource;


@Configuration
public class DataSourceProxyConfig {

    /**
     * 需要将 DataSourceProxy 设置为主数据源，否则事务无法回滚
     */
    @Primary
    @Bean("dataSource")
    public DataSource dataSource(@Value("${spring.datasource.url}")String url, @Value("${spring.datasource.driver-class-name}")String driverClassName,
                                 @Value("${spring.datasource.username}")String username, @Value("${spring.datasource.password}")String password) {
        HikariDataSource hikariDataSource = new HikariDataSource();
        hikariDataSource.setJdbcUrl(url);
        hikariDataSource.setDriverClassName(driverClassName);
        hikariDataSource.setUsername(username);
        hikariDataSource.setPassword(password);
        return new DataSourceProxy(hikariDataSource);
    }
}
```

## 回滚

```java
@Service
public class OrderService {
  //出现异常自动回滚无异常自动提交
    @GlobalTransactional(name = "my_test_tx_group", rollbackFor = Exception.class)
    public void t1() {
        //业务逻辑
    }
  
  
  //捕捉异常手动回滚
    public void t2() {
        GlobalTransaction globalTransaction = GlobalTransactionContext.getCurrentOrCreate();
        try {
          //开始事物
            globalTransaction.begin(3000, "my_test_tx_group");
          
            //业务逻辑
          
          //没有异常提交事务
            globalTransaction.commit();
        } catch (Exception e) {
            try {
              //出现异常回滚事务
                globalTransaction.rollback();
            } catch (TransactionException ex) {
                ex.printStackTrace();
            }
        }
    }
}
```

## other

>使用openfeign调用其他服务时，参数>=两个会报错，解决办法：
>
>使用@RequestParam("id")
>
>```java
>@FeignClient(value = "Seata-Storage")
>public interface StorageService {
>@GetMapping("/update/storage")
>String updateStorage (@RequestParam("id") Integer id,@RequestParam("account") Integer account);
>}
>```
