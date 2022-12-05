---
title: "Java RabbitMQ"
date: 2019-09-24T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- java
tags: 
- java
- rabbitMQ
---

# 安装：

>1.	rpm -Uvh https://download.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
>
>2.	yum install erlang
>
>3.	yum install socat
>
>4.	rpm -Uvh https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.0/rabbitmq-server-3.8.0-1.el7.noarch.rpm
>
>5.	rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
>
>6.	yum install rabbitmq-server-3.6.8-1.el7.noarch.rpm
>
>7.	service rabbitmq-server start
>
>8.	service rabbitmq-server status
>
>9.	rabbitmq-service start 开始服务
>
>  Rabbitmq-service stop  停止服务
>
>  rabbitmq-server -detached 后台运行

>- ## hello world简单模式
>
> - #### 一对一，一个生产者对应一个消费者。
>
> - #### 简单模式不需要创建交换机，但是有交换机，是使用的默认交换机。

>### 生产者

```java
package com.xiaoyu.test1;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class Pro_HelloWorld {
    public static void main(String[] args) throws IOException, TimeoutException {
        //配置连接信息
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("47.115.50.193");	//ip
        factory.setPort(5672);	//端口号
        factory.setVirtualHost("/");	//虚拟机
        factory.setUsername("admin");	//账户
        factory.setPassword("admin");	//密码
        //配置管道
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        //队列创建
        channel.queueDeclare("helloworld", true, false, false, null);
      /*	参数：
                queue：队列名称，如果没有这个队列则创建这个队列，有则不会创建。
                durable：是否持久化，当mq重启之后是否还有消息。
                exclusive：是否独占，只能有一个消费者监听这队列，当connection关闭时，是否删除队列。
                autoDelete：是否自动删除，让没有consumer时，自动删除掉。
                arguments：参数。
      */
        //发送消息
        String body = "hell rabbitMQ";
        channel.basicPublish("", "helloworld", null, body.getBytes());
      /*	参数：
      exchange：交换机名称，简单模式下交换机会使用默认的 ‘’ ’‘。
			routingKey：路由名称，使用默认交换机的话路由名称要和队列名称一致。
			props：配置信息。
			body：发送的消息数据。
      */
        //关闭
        channel.close();
        connection.close();
    }
}
```

>### 消费者

```java
package com.xiaoyu.test1;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class Con_HelloWorld {
    public static void main(String[] args) throws IOException, TimeoutException {
        //创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("47.115.50.193");
        factory.setPort(5672);
        factory.setVirtualHost("/");
        factory.setUsername("admin");
        factory.setPassword("admin");
        //创建连接
        Connection connection = factory.newConnection();
        //创建管道
        Channel channel = connection.createChannel();
        //配置信息
        channel.queueDeclare("hello_world1", true, false, false, null);
        //消息接收到时自动执行该方法
        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("consumerTag标识:" + consumerTag);
                System.out.println("Exchange管道:" + envelope.getExchange());
                System.out.println("RoutingKey路由key:" + envelope.getRoutingKey());
                System.out.println("properties属性:" + properties);
                System.out.println("body消息:" + new String(body));
            }
        };
        //设置接受那个管道
        channel.basicConsume("hello_world1", true, consumer);
    }
}
```

>- ## Work queues工作队列模式
>
> - #### 一对多，一个生产者多个消费者。
>
> - #### 工作模式不需要创建交换机，但是有交换机，是使用的默认交换机。

>生产者

```java
package com.xiaoyu.test1;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class Pro_HelloWorld {
    public static void main(String[] args) throws IOException, TimeoutException {
        //配置连接信息
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("47.115.50.193");
        factory.setPort(5672);
        factory.setVirtualHost("/");
        factory.setUsername("admin");
        factory.setPassword("admin");
        //配置管道
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        //队列声明
        //channel.queueDeclare("helloworld", true, false, false, null);
        //发送消息
        String body = "hell rabbitMQ";
        channel.basicPublish("", "helloworld", null, body.getBytes());
        //关闭
        channel.close();
        connection.close();
    }
}
```

>消费者

```java
package com.xiaoyu.test1;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class Con_HelloWorld {
    public static void main(String[] args) throws IOException, TimeoutException {
        //创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("47.115.50.193");
        factory.setPort(5672);
        factory.setVirtualHost("/");
        factory.setUsername("admin");
        factory.setPassword("admin");
        //创建连接
        Connection connection = factory.newConnection();
        //创建管道
        Channel channel = connection.createChannel();
        //配置信息
        channel.queueDeclare("hello_world1", true, false, false, null);
        //消息接收到时自动执行该方法
        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("consumerTag标识:" + consumerTag);
                System.out.println("Exchange管道:" + envelope.getExchange());
                System.out.println("RoutingKey路由key:" + envelope.getRoutingKey());
                System.out.println("properties属性:" + properties);
                System.out.println("body消息:" + new String(body));
            }
        };
        //设置接受那个管道
        channel.basicConsume("hello_world1", true, consumer);
    }
}
```

>- ## Publish/Subscribe发布订阅模式
>
> - #### 一次性向多个消费者发送消息。
>
> - #### 创建多个队列，分别向队列发送消息，消费者再分别进行消费。

>### 生产者

```java
package com.xiaoyu.test1;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class Pro_PUSU {
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("47.115.2.107");
        factory.setPort(5672);
        factory.setVirtualHost("/");
        factory.setUsername("guest");
        factory.setPassword("guest");

        Connection connection = factory.newConnection();

        Channel channel = connection.createChannel();

        String exchangeName = "test_fanout";
        //创建交换机
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.FANOUT, true, false, false, null);
      /*	参数：
      	exchange：交换机名称
				type：交换机类型
						DIRECT：定向
						FANOUT：扇形（广播）
						TOPIC：通配符的方式
						HEADERS：参数匹配
				durable：是否持久化
				autoDelete：自动删除
				internal：内部使用，一般false
				arguments：参数
      */
        //创建队列
        channel.queueDeclare("test_fanout_qu1", true, false, false, null);
        channel.queueDeclare("test_fanout_qu2", true, false, false, null);
        //绑定队列和交换机
        channel.queueBind("test_fanout_qu1", exchangeName, "");
        channel.queueBind("test_fanout_qu2", exchangeName, "");
      /*	参数：
      	queue：队列名称。
				exchange：交换机名称。
				routingKey：路由键，绑定规则，如果交换机的类型为fanout，routingKey设置为空字符串。
      */
        //发送消息
        String body = "日志信息：张三调用find all日志级别，info。。。";
        channel.basicPublish(exchangeName, "", null, body.getBytes());
        channel.close();
        connection.close();
    }
}
```

>### 消费者

```java
/*
	分别创建多个队列，生产者会同时向这些队列进行发送消息，队列接受消息并进行消费。
*/
```

>- ##  Routing路由模式
>
> - #### 发送消息到路由key，路由key再进行判断分别发送给对应的队列。

>### 生产者

```java
package com.xiaoyu.test1;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class Pro_Routing {
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setPort(5672);
        factory.setVirtualHost("/");
        factory.setUsername("guest");
        factory.setPassword("guest");

        Connection connection = factory.newConnection();

        Channel channel = connection.createChannel();

        String exchangeName = "test_direct";
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.DIRECT, true, false, false, null);

        channel.queueDeclare("test_direct_qu1", true, false, false, null);
        channel.queueDeclare("test_direct_qu2", true, false, false, null);


        channel.queueBind("test_direct_qu1", exchangeName, "error");
        channel.queueBind("test_direct_qu2", exchangeName, "info");
        channel.queueBind("test_direct_qu2", exchangeName, "error");
        channel.queueBind("test_direct_qu2", exchangeName, "warning");

        String body = "日志信息：张三调用find all日志级别，info。。。";
        channel.basicPublish(exchangeName, "warning", null, body.getBytes());

        channel.close();
        connection.close();
    }
}
```

>### 消费者

```java
/*
	分别创建对应队列，路由key会对消息进行判断分发到对应的消息队列。
*/
```

>- ## Topics通配模式
>
> - #### 和路由模式一样，只是要比路由模式更加方便，可以自己指定通配符来进行判断。

>### 生产者

```java
public void pro2() throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("47.115.2.107");
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();
        String exchangeName = "hzyy";
        channel.exchangeDeclare(exchangeName,BuiltinExchangeType.TOPIC,true,false,false,null);
        channel.queueDeclare("con1",true,false,false,null);
        channel.queueDeclare("con2",true,false,false,null);
        channel.queueBind("con1",exchangeName,"*.a");
//        channel.queueBind("con1",exchangeName,"b.#");
        channel.queueBind("con2",exchangeName,"#.a");
//        channel.queueBind("con2",exchangeName,"#.a");
        channel.basicPublish(exchangeName,"a.a",null,"a.a".getBytes());
        channel.basicPublish(exchangeName,"b.qqqqqq",null,"b.qqqqqq".getBytes());
        channel.basicPublish(exchangeName,"a.aaaaaa",null,"a.b".getBytes());
        channel.basicPublish(exchangeName,"b.q",null,"b.c".getBytes());
        channel.close();
        connection.close();
    }
```

>### 消费者

```java
/*
	分别创建对应队列，路由key会对消息进行判断分发到对应的消息队列。
*/
```

>##### Exchange 交换机
>
>​	
>
>- Fanout 广播
>
> 		将消息交给所有绑定到交换机的队列
> 			路由键为null
>
>
>- Direct 定向
>
> 		把消息交给符合指定routing Key（路由key）的队列
> 			指定路由键
>
>
>- Topic 通配符
>
> 		把消息交给符合routing pattern（路由模式）的队列
>
>
>交换机只负责转发消息，不具备存储消息的能力，因此如果没有任何队列与Exchange绑定，或者没有符合路由规则的队列，那么消息会丢失
