---
title: "springBoot整合rabbitMQ"
date: 2019-10-12T11:28:07+08:00
author: ["loveyu"]
draft: false
categories: 
- java
tags: 
- java
- spring
- springBoot
- rabbitMQ
---

# SpringBoot 整合 rabbitMQ

>mac安装brew install rabbitmq

># application.yml设置
>
>```yaml
>spring:
>rabbitmq:
>host: 47.115.50.193
>username: admin
>password: admin
>```

>## 创建交换机，消息队列，并进行绑定

```java
@Configuration
public class RabbitMQConfig {
    private static final String TOPIC_NAME = "top_name";
    private static final String QUEUE_NAME = "queue_name";
    //交换机
    @Bean("queueEx")
    public Exchange queueEx(){
        return ExchangeBuilder.topicExchange(TOPIC_NAME).durable(true).build();
    }
    //消息队列
    @Bean("queue")
    public Queue queue(){
        return QueueBuilder.durable(QUEUE_NAME).build();
    }
    //绑定
    @Bean
    public Binding queueExchange(@Qualifier("queueEx") Exchange exchange,@Qualifier("queue") Queue queue){
        return BindingBuilder.bind(queue).to(exchange).with("a.#").noargs();
    }
}
```

>```java
>@Autowired
>private RabbitTemplate rabbitTemplate;
>```
>
>## 生产者
>
>```java
>public void q(){
>rabbitTemplate.convertAndSend("top_name","a.qwe","rabbitmqboot");
>}
>```
>
>## 消费者
>
>```java
>@RabbitListener(queues = "queue_name") //设置要监听的消息队列
>public void listenerq(Message message,Channel channel){
>System.out.println(new String(message.getBody()));
>}
>```



>## 消息可靠投递
>
>### Application.yml配置
>
>```yaml
>rabbitmq:
>publisher-confirm-type: correlated
>```
>
>### Java代码配置
>
>```java
>package com.xiaoyu.config;
>
>import org.springframework.amqp.core.ReturnedMessage;
>import org.springframework.amqp.rabbit.connection.CorrelationData;
>import org.springframework.amqp.rabbit.core.RabbitTemplate;
>import org.springframework.beans.factory.annotation.Autowired;
>import org.springframework.stereotype.Component;
>
>import javax.annotation.PostConstruct;
>
>@Component
>public class PubConfirmAndReturn implements RabbitTemplate.ReturnsCallback,RabbitTemplate.ConfirmCallback{
>
>@Autowired
>RabbitTemplate rabbitTemplate;
>
>@PostConstruct
>public void init(){
>   rabbitTemplate.setReturnsCallback(this);
>   rabbitTemplate.setConfirmCallback(this);
>}
>
>//confirm确定模式
>@Override
>public void confirm(CorrelationData correlationData, boolean b, String s) {
>   if (b) {
>       System.out.println("消息发送到exchange成功");
>   } else {
>       System.out.println("失败原因："+ s);
>   }
>}
>
>//return退回模式
>@Override
>public void returnedMessage(ReturnedMessage returnedMessage) {
>   System.out.println("消息送exchange到queue失败，详情为：");
>   System.out.println("消息主体 message : "+new String(returnedMessage.getMessage().getBody()));
>   System.out.println("描述："+ returnedMessage.toString());
>   System.out.println("消息使用的交换器 exchange : "+ returnedMessage.getExchange());
>   System.out.println("消息使用的路由键 routing : "+ returnedMessage.getRoutingKey());
>}
>} 
>```
>
>### rabbitmq 整个消息投递的路径为：
>producer—>rabbitmq broker—>exchange—>queue—>consumer
>消息从 producer 到 exchange 则会返回一个 confirmCallback 确认模式。
>消息从 exchange–>queue 投递失败则会返回一个 returnCallback退回模式 。



>## ack
>
>```java
>@RabbitListener(queues = "queue_name")
>public void listener(Message message, Channel channel){
>long deliveryTag = message.getMessageProperties().getDeliveryTag();
>try {
>System.out.println(new String(message.getBody()));
>System.out.println("处理业务逻辑");
>Thread.sleep(2000);
>int a = 1/0;
>//        手动签收
>channel.basicAck(deliveryTag,true);
>} catch (Exception e) {
>//  第三个参数：requeue：重回队列，如果设置为true，则消息重新回到queue，broker会重新发送消息给消费端
>try {
> channel.basicNack(deliveryTag,true,true);
>} catch (Exception ioException) {
> ioException.printStackTrace();
>}
>}
>}
>```
>
>#### 当消息在被消费时，如果在不手动签收则会停滞，如果在处理业务逻辑时出错，则可以设置是否重回队列



>## ttl过期
>
>### 队列过期
>
>```java
>@Bean("queuettl")
>public Queue queueTTL(){
>//        ttl:队列的过期时间
>return QueueBuilder.durable("queue_ttl").ttl(1000000).build();
>}
>@Bean("exchangettl")
>public Exchange exchangeTTL(){
>return ExchangeBuilder.topicExchange("exchange_ttl").durable(true).build();
>}
>@Bean
>public Binding ttl(@Qualifier("queuettl") Queue queue,@Qualifier("exchangettl") Exchange exchange){
>return BindingBuilder.bind(queue).to(exchange).with("q.#").noargs();
>}
>```
>
>##### 队列过期是指：所有发送到这个队列的消息如果没有在规定的时间消费就会被处理掉（删除改消息）
>
>#### 消息过期
>
>```java
>public void ttl(){
>//        消息后处理对象，设置一些消息的参数信息
>MessagePostProcessor messagePostProcessor = new MessagePostProcessor() {
>@Override
>public Message postProcessMessage(Message message) throws AmqpException {
> //                设置message的信息
> message.getMessageProperties().setExpiration("10000");
> //                返回该消息
> return message;
>}
>};
>//        如果同时设置了队列过期时间和消息单独过期时间，以时间短的为准。如下列情况是到十秒同时过期
>//        队列过期会将队列内的消息全部移除
>//        消息过期后，只有消息在队列的顶端才会判断其是否过期，过期则移除
>//        队列过期
>rabbitTemplate.convertAndSend("exchange_ttl","q.aa","ttl");
>//        消息单独过期
>rabbitTemplate.convertAndSend("exchange_ttl","q.aa","ttl",messagePostProcessor);
>}
>```
>
>##### 消息过期是指：发送的这一条消息会被设置过期时间，正常发送的消息则不会过期。



>## dlx死信
>
>### 造成死信的原因：
>
>* 过期时间
>
> * ```java
>   //.ttl(10000)    
>   public void one_dlx(){
>     rabbitTemplate.convertAndSend("test_exchange_dlx","test_dlx.xx","one_del....");
>   }
>   ```
>
>* 长度限制
>
>   * ```java
>     //.maxLength(10)	设置最大为10条消息
>     public void two_dlx(){
>       for (int i = 0; i < 20; i++) {
>         rabbitTemplate.convertAndSend("test_exchange_dlx","test_dlx.xx","two_del....");
>       }
>     }
>     ```
>
>* 消息拒收
>
>   * ```java
>     public void three_dlx(Message message, Channel channel){
>       System.out.println(new String(message.getBody()));
>       try {
>         channel.basicNack(message.getMessageProperties().getDeliveryTag(),true,false);
>       } catch (IOException e) {
>         e.printStackTrace();
>       }
>     }
>     ```
>
>#### 死信配置
>
>```java
>//    1:声明正常队列和交换机进行绑定
>@Bean("test_queue_dlx")
>public Queue test_queue_dlx(){
> return QueueBuilder.durable("test_queue_dlx").deadLetterExchange("exchange_dlx").deadLetterRoutingKey("dlx.suibian").ttl(10000).maxLength(10).build();
>}
>@Bean("test_exchange_dlx")
>public Exchange test_exchange_dlx(){
> return ExchangeBuilder.topicExchange("test_exchange_dlx").durable(true).build();
>}
>@Bean
>public Binding test_binding_dlx(@Qualifier("test_queue_dlx")Queue queue,@Qualifier("test_exchange_dlx")Exchange exchange){
> return BindingBuilder.bind(queue).to(exchange).with("test_dlx.#").noargs();
>}
>//    2.声明死信队列和交换机进行绑定
>@Bean("queue_dlx")
>public Queue queue_dlx(){
> return QueueBuilder.durable("queue_dlx").build();
>}
>@Bean("exchange_dlx")
>public Exchange exchange_dlx(){
> return ExchangeBuilder.topicExchange("exchange_dlx").durable(true).build();
>}
>@Bean
>public Binding binding_dlx(@Qualifier("queue_dlx")Queue queue,@Qualifier("exchange_dlx")Exchange exchange){
> return BindingBuilder.bind(queue).to(exchange).with("dlx.#").noargs();
>}
>```
>
>##### 死信队列可以理解为是被绑定正常队列的回收站，死信队列和正常队列一样，一样可以消费其中的消息



>## ttl+dlx延迟队列
>
>#### 配置
>
>```java
>@Bean("test_queue_ttl_dlx")
>public Queue test_queue_ttl_dlx(){
>return QueueBuilder.durable("test_queue_ttl_dlx").deadLetterExchange("exchange_ttl_dlx").deadLetterRoutingKey("ttl_dlx.suibian").ttl(10000).build();
>}
>@Bean("test_exchange_ttl_dlx")
>public Exchange test_exchange_ttl_dlx(){
>return ExchangeBuilder.topicExchange("test_exchange_ttl_dlx").durable(true).build();
>}
>@Bean
>public Binding test_binding_ttl_dlx(@Qualifier("test_queue_ttl_dlx")Queue queue,@Qualifier("test_exchange_ttl_dlx")Exchange exchange){
>return BindingBuilder.bind(queue).to(exchange).with("test_ttl_dlx.#").noargs();
>}
>
>@Bean("queue_ttl_dlx")
>public Queue queue_ttl_ttl_dlx(){
>return QueueBuilder.durable("queue_ttl_dlx").build();
>}
>@Bean("exchange_ttl_dlx")
>public Exchange exchange_ttl_dlx(){
>return ExchangeBuilder.topicExchange("exchange_ttl_dlx").durable(true).build();
>}
>@Bean
>public Binding binding_ttl_dlx(@Qualifier("queue_ttl_dlx")Queue queue,@Qualifier("exchange_ttl_dlx")Exchange exchange){
>return BindingBuilder.bind(queue).to(exchange).with("ttl_dlx.#").noargs();
>}
>```
>
>### 生产者
>
>```java
>@GetMapping("/ttl_dlx")
>@ResponseBody
>public void ttl_dlx(){
>for (int i = 0; i < 10; i++) {
>rabbitTemplate.convertAndSend("test_exchange_ttl_dlx","test_ttl_dlx.xx","ttl_dlx...");
>}
>}
>```
>
>### 消费者
>
>```java
>@RabbitListener(queues = "queue_ttl_dlx")
>public void ttl_dlx_li(Message message,Channel channel){
>System.out.println(new String(message.getBody()));
>try {
>channel.basicAck(message.getMessageProperties().getDeliveryTag(),true);
>} catch (IOException e) {
>e.printStackTrace();
>}
>}
>```
>
>#### 延迟队列：设置队列为过期队列，当改消息过期后到绑定的死信队列里面，从而实现消息的延迟
