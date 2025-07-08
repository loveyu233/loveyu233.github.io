---
title: "Go Rabbitmq"
date: 2023-06-23T10:01:17+08:00
author: ["loveyu"]
draft: false
categories: 
- go
- mq
tags: 
- rabbitmq
---

# RabbitMQ模式

1. 简单模式（Simple Mode）： 简单模式是最基本的模式，也是最简单的消息传递模式。它包括一个生产者将消息发送到一个队列中，然后一个消费者从同一个队列接收消息并进行处理。
2. 工作队列模式（Work Queue Mode）： 工作队列模式也被称为任务队列模式或消息队列模式。它包括一个生产者将任务发送到一个队列中，多个消费者从同一个队列接收任务并进行处理。每个任务只能由一个消费者处理，确保任务的负载均衡。
3. 发布/订阅模式（Publish/Subscribe Mode）： 发布/订阅模式用于将消息广播给多个消费者。生产者将消息发布到交换机（Exchange），然后交换机将消息传递到所有绑定的队列中，每个队列都有自己的消费者来接收和处理消息。
4. 路由模式（Direct Exchange Mode）： 路由模式是基于消息的路由键（Routing Key）进行消息传递的一种模式。生产者将消息发送到交换机，并指定一个特定的路由键，交换机将消息传递给所有与该路由键匹配的队列，每个队列都有自己的消费者来接收和处理消息。
5. 主题模式（Topic Exchange Mode）： 主题模式是路由模式的扩展，它支持使用通配符的路由键进行消息传递。生产者将消息发送到交换机，并指定一个匹配规则的路由键模式，交换机根据匹配规则将消息传递给符合条件的队列，每个队列都有自己的消费者来接收和处理消息。
6. 领域特定语言模式（Distributed Domain-Specific Language Mode）： 领域特定语言模式（也称为流派特定语言模式）是一种用于实现复杂分布式系统的高级模式。它基于领域特定语言（DSL）来定义和描述消息的流动和处理过程，以实现更精确的消息传递和处理策略。





# 直连模式（Direct）

>- 创建连接和信道。
>- 声明交换机和队列。
>- 将队列绑定到交换机，并指定路由键。
>- 发布消息到交换机，附带指定的路由键。

```go
// 创建连接和信道
conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
ch, err := conn.Channel()

// 声明交换机和队列
q, err := ch.QueueDeclare("my_queue", false, false, false, false, nil)

// 将队列绑定到交换机，并指定路由键
err = ch.QueueBind(q.Name, "my_routing_key", "my_exchange", false, nil)

// 发布消息到交换机，附带指定的路由键
msg := amqp.Publishing{
    Body: []byte("Hello, RabbitMQ!"),
}
err = ch.Publish("my_exchange", "my_routing_key", false, false, msg)

```



# 主题模式（Topic）

>- 创建连接和信道。
>- 声明交换机和队列。
>- 将队列绑定到交换机，并使用通配符形式的路由键进行匹配。
>- 发布消息到交换机，附带指定的路由键。

```go
// 创建连接和信道
conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
ch, err := conn.Channel()

// 声明交换机和队列
q, err := ch.QueueDeclare("my_queue", false, false, false, false, nil)

// 将队列绑定到交换机，并使用通配符形式的路由键进行匹配
err = ch.QueueBind(q.Name, "my_topic.#", "my_exchange", false, nil)

// 发布消息到交换机，附带指定的路由键
msg := amqp.Publishing{
    Body: []byte("Hello, RabbitMQ!"),
}
err = ch.Publish("my_exchange", "my_topic.key", false, false, msg)
```



# 发布/订阅模式（Publish/Subscribe）

>- 创建连接和信道。
>- 声明交换机。
>- 声明多个队列。
>- 将队列绑定到交换机。
>- 发布消息到交换机。

```go
// 创建连接和信道
conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
ch, err := conn.Channel()

// 声明交换机
err = ch.ExchangeDeclare("my_exchange", "fanout", false, false, false, false, nil)

// 声明多个队列
q1, err := ch.QueueDeclare("queue1", false, false, false, false, nil)
q2, err := ch.QueueDeclare("queue2", false, false, false, false, nil)

// 将队列绑定到交换机
err = ch.QueueBind(q1.Name, "", "my_exchange", false, nil)
err = ch.QueueBind(q2.Name, "", "my_exchange", false, nil)

// 发布消息到交换机
msg := amqp.Publishing{
    Body: []byte("Hello, RabbitMQ!"),
}
err = ch.Publish("my_exchange", "", false, false, msg)
```



# 工作队列模式（Work Queue）

>- 创建连接和信道。
>- 声明队列。
>- 发布消息到队列。

```go
// 创建连接和信道
conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
ch, err := conn.Channel()

// 声明队列
q, err := ch.QueueDeclare("my_queue", false, false, false, false, nil)

// 发布消息到队列
msg := amqp.Publishing{
    Body: []byte("Hello, RabbitMQ!"),
}
err = ch.Publish("", q.Name, false, false, msg)
```



# 可选参数

```go
	/*
  x-message-ttl：设置消息的过期时间（Time to Live），单位为毫秒。超过该时间的消息将被自动删除。
  x-expires：设置队列的过期时间，单位为毫秒。如果在指定的时间内没有连接使用该队列，队列将被自动删除。
  x-max-length：设置队列的最大长度限制。一旦达到该限制，后续的消息将会被拒绝或被丢弃。
  x-max-length-bytes：设置队列的最大占用空间限制。一旦达到该限制，后续的消息将会被拒绝或被丢弃。
  x-dead-letter-exchange：设置队列的死信交换机，用于处理被拒绝、过期或达到最大重试次数的消息。
  x-dead-letter-routing-key：设置队列的死信路由键，用于指定被发送到死信交换机的消息的路由键。
  x-overflow：设置队列溢出行为。可以是 drop-head（删除最早的消息）或 reject-publish（拒绝发布新消息）。
	*/
	amqp.Table{}
```



# 死信队列代码

>设置``"x-message-ttl":             "2000",``无效,会导致数据无法发送成功
>
>在发送数据时设置``Expiration:  "1000",``有效,会在过期后把消息路由到私信队列中

```go
package main

import (
	"fmt"
	"github.com/streadway/amqp"
)

func main() {
	dial, _ := amqp.Dial("amqp://guest:guest@localhost:5672")
	ch, _ := dial.Channel()
	// 设置死信交换机和死信路由键
	args := amqp.Table{
		"x-dead-letter-exchange":    "dead_letter_exchange",
		"x-dead-letter-routing-key": "dead_letter_key",
		//"x-message-ttl":             "2000",
	}

	// 声明普通队列，并将死信属性添加到其参数中
	normalQueue, _ := ch.QueueDeclare(
		"normal_queue",
		true,
		false,
		false,
		false,
		args,
	)

	// 声明死信队列
	deadLetterQueue, _ := ch.QueueDeclare(
		"dead_letter_queue",
		true,
		false,
		false,
		false,
		nil,
	)

	// 定义死信交换机
	ch.ExchangeDeclare(
		"dead_letter_exchange",
		"direct",
		true,
		false,
		false,
		false,
		nil,
	)

	// 将死信队列绑定到死信交换机上
	ch.QueueBind(
		"dead_letter_queue",
		"dead_letter_key", // 死信路由键
		"dead_letter_exchange",
		false,
		nil,
	)
	err := publishMessage(ch, normalQueue.Name, "测试数据")
	fmt.Println(err)
	deadMessageCount, _ := ch.QueueInspect(deadLetterQueue.Name)
	normalMessageCount, _ := ch.QueueInspect(normalQueue.Name)
	fmt.Println("dead: ", deadMessageCount.Messages)
	fmt.Println("normal: ", normalMessageCount.Messages)

	//consume, _ := ch.Consume("normal_queue", "", false, false, false, false, nil)
	//for d := range consume {
	//	fmt.Println(string(d.Body))
	//	d.Reject(false)
	//}
	//fmt.Println("dead: ", deadLetterQueue.Messages)

}
func publishMessage(ch *amqp.Channel, queueName, message string) error {
	// 将消息发送到普通队列
	err := ch.Publish(
		"",
		queueName,
		false,
		false,
		amqp.Publishing{
			Expiration:  "1000",
			ContentType: "text/plain",
			Body:        []byte(message),
		},
	)
	if err != nil {
		return fmt.Errorf("无法发送消息到队列: %v", err)
	}

	fmt.Println("已发布测试消息到队列:", queueName)
	return nil
}

```





# 交换机和路由键区别

1. 交换机（Exchange）： 交换机是消息的分发中心，它接收来自生产者的消息并根据路由键的匹配规则将消息发送给与之绑定的队列。交换机有不同的类型，常见的类型包括 direct、topic、fanout 和 headers。

   - direct：直连交换机，根据消息的路由键与绑定的队列的绑定键进行精确匹配。
   - topic：主题交换机，使用通配符形式的路由键与绑定键进行模糊匹配。
   - fanout：发布/订阅交换机，无需匹配，将消息广播给所有与之绑定的队列。
   - headers：头交换机，根据消息的头部属性与绑定的队列的头部属性进行匹配。

   交换机通过声明创建，并通过名称指定。生产者发布消息时，可以指定发送到哪个交换机。

2. 路由键（Routing Key）： 路由键是消息的标识符，用于将消息从交换机路由到相应的队列。具体的路由规则取决于交换机的类型。

   - 在 direct 类型的交换机中，路由键必须与队列的绑定键完全匹配。
   - 在 topic 类型的交换机中，路由键可以使用通配符形式进行匹配，例如使用 `*` 表示单个词，使用 `#` 表示零个或多个词。
   - 在 fanout 和 headers 类型的交换机中，路由键无需匹配，因为消息会被广播给所有绑定的队列。

   生产者发布消息时，需要指定消息的路由键，以便交换机将其发送到相应的队列。



# 交换机的类型

1. 直连交换机（Direct Exchange）： 直连交换机根据消息的路由键（Routing Key）将消息发送到相匹配的队列上。路由键与队列的绑定规则一一对应，只有当消息的路由键与队列的绑定规则完全匹配时，消息才会被路由到该队列。这是一种一对一的路由方式。
2. 主题交换机（Topic Exchange）： 主题交换机使用通配符匹配消息的路由键和队列的绑定规则，从而将消息发送到一个或多个匹配的队列上。在主题交换机中，路由键可以使用通配符 `*` 和 `#`，其中 `*` 匹配一个单词，`#` 匹配零个或多个单词。这是一种灵活且强大的路由方式。
3. 扇形交换机（Fanout Exchange）： 扇形交换机将消息广播到所有与之绑定的队列上，忽略消息的路由键。当消息发送到扇形交换机时，它会被复制并发送到所有绑定的队列，实现一对多的消息广播。
4. 头部交换机（Headers Exchange）： 头部交换机根据消息的头部属性（Headers）进行匹配和路由。在绑定队列时，可以指定一组键值对，并通过匹配消息的头部属性与之进行匹配，从而将消息发送到相应的队列中。



# QueueInspect和queueDeclarePassive区别

1. 功能：
   - `QueueInspect` 方法用于获取队列的详细信息，包括队列的名称、持久化标志、是否排他、是否自动删除、绑定的交换机等。
   - `QueueDeclarePassive` 方法用于检查队列是否已存在，如果队列存在则返回成功，否则返回失败。
2. 调用方式：
   - `QueueInspect` 方法是通过管理插件的 HTTP API 进行调用的，需要发送 HTTP 请求到 RabbitMQ 的管理接口上，并提供正确的权限和参数。
   - `QueueDeclarePassive` 方法是通过 AMQP 协议直接与 RabbitMQ 服务器通信进行调用的，可以在代码中使用 RabbitMQ 客户端库直接调用该方法。
3. 返回结果：
   - `QueueInspect` 方法返回包含队列详细信息的结构体或 JSON 格式数据，可以获取队列的各种属性。
   - `QueueDeclarePassive` 方法返回一个成功/失败的响应，用于判断队列是否存在。
4. 应用场景：
   - `QueueInspect` 方法适用于需要获取队列详细信息的场景，如监控、管理工具等。
   - `QueueDeclarePassive` 方法适用于在代码中检查队列是否已存在的场景，可以避免重复声明队列。

`QueueInspect` 方法用于获取队列的详细信息，而 `QueueDeclarePassive` 方法用于检查队列是否已存在。根据具体的需求和使用场景选择合适的方法。



# 头部交换机

```go
package main

import (
    "fmt"
    "log"

    "github.com/streadway/amqp"
)

func main() {
    // 建立 RabbitMQ 连接
    conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
    if err != nil {
        log.Fatal(err)
    }
    defer conn.Close()

    // 创建一个信道
    channel, err := conn.Channel()
    if err != nil {
        log.Fatal(err)
    }
    defer channel.Close()

    // 声明头部交换机
    err = channel.ExchangeDeclare(
        "my_headers_exchange", // 交换机名称
        "headers",             // 交换机类型
        true,                  // 是否持久化
        false,                 // 是否自动删除
        false,                 // 是否是内部交换机
        false,                 // 是否等待服务器确认
        amqp.Table{},          // 额外参数
    )
    if err != nil {
        log.Fatal(err)
    }

    // 创建一个队列
    queue, err := channel.QueueDeclare(
        "my_headers_queue", // 队列名称
        true,               // 是否持久化
        false,              // 是否自动删除
        false,              // 是否排他队列
        false,              // 是否等待服务器确认
        nil,                // 额外参数
    )
    if err != nil {
        log.Fatal(err)
    }

    // 绑定队列到头部交换机，并指定匹配的头部属性
    err = channel.QueueBind(
        queue.Name,                       // 队列名称
        "",                               // 路由键，对于头部交换机不需要指定
        "my_headers_exchange",            // 交换机名称
        false,                            // 是否不等待服务器确认
      amqp.Table{"x-match": "all",      // 头部属性的匹配规则，这里表示所有的头部属性都要匹配上,any:只要至少有一个指定的键值对匹配，就会路由到队列。
                   "key1": "value1",      // 设置头部属性
                   "key2": "value2"},
    )
    if err != nil {
        log.Fatal(err)
    }

    // 发布一条包含头部属性的消息
    headers := amqp.Table{"key1": "value1", "key2": "value2"}
    err = channel.Publish(
        "my_headers_exchange", // 交换机名称
        "",                    // 路由键，对于头部交换机不需要指定
        false,                 // 是否不等待服务器确认
        false,                 // 是否立即发送
        amqp.Publishing{
            Headers: headers,              // 设置头部属性
            ContentType: "text/plain",
            Body:        []byte("Hello, RabbitMQ!"),
        },
    )
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println("消息已发送到头部交换机")
}

```



# TTL

> TTL（Time-to-Live）是一种用于设置消息的生存时间的功能。在 RabbitMQ 中，通过设置消息的 TTL，可以指定消息在被消费或者被丢弃之前允许存在的时间长度。

在 RabbitMQ 中，可以通过以下几种方式来实现消息的 TTL：

1. 消息级别的 TTL：在发布消息时，可以通过设置消息的 `expiration` 属性来指定消息的 TTL。该属性的值是一个表示毫秒数的字符串，指定了消息从发送后经过多长时间后过期。一旦消息的 TTL 时间到期，消息将会被立即丢弃或者死信处理。
2. 队列级别的 TTL：在声明队列时，可以使用 `x-message-ttl` 参数来设置队列中所有消息的默认 TTL。该参数的值是一个表示毫秒数的整数，用于指定消息在队列中的存活时间。
3. 无限期的消息：如果不想为消息设置 TTL 或者希望让消息永久存在直到被消费或者手动删除，可以将 TTL 设置为 0。这样的消息不会因为时间而过期。

> 需要注意的是，TTL 只能保证消息的最长存在时间，但并不能保证消息一定会在 TTL 时间内被消费。如果消息在 TTL 时间内没有被消费，那么它将会被丢弃或者进入死信队列，具体取决于您的设置和配置。



# 死信队列

> 死信队列（Dead Letter Queue）是一种用于处理无法被消费或处理的消息的特殊队列。当消息满足一定的条件时，例如消息过期、被拒绝、达到最大重试次数等，它将被路由到死信队列中，而不是直接丢弃或者无限地重新投递。

死信队列的使用可以提供以下几个优势：

1. 错误处理：当消息处理失败或出现错误时，将消息发送到死信队列，方便后续进行错误处理和调查。
2. 重试机制：通过配置死信队列以及相关参数（例如消息的 TTL 和重试次数限制），可以实现自动的消息重试机制，减少手动处理的工作量。
3. 延迟处理：通过设置死信队列中消息的 TTL，可以实现延迟处理机制。消息在一定时间后进入死信队列，随后再从死信队列中取出进行处理。

使用死信队列的步骤如下：

1. 创建死信队列：首先要创建一个专门用于接收死信的队列，通常是一个普通的队列。
2. 设置死信属性：在声明普通队列时，可以设置 `x-dead-letter-exchange` 和 `x-dead-letter-routing-key` 属性。这些属性指示当消息成为死信时应该发送到的交换机和路由键。
3. 定义死信规则：通过设置消息的 TTL、最大重试次数等属性，来定义什么样的消息会成为死信。
4. 绑定死信队列：将普通队列与死信队列进行绑定，以便将死信路由到死信队列中。



# Nack、Ack 和 Reject

>主要区别在于 `Nack` 可以选择拒绝多个消息并重新投递，而 `Ack` 仅确认消息已处理并删除，`Reject` 则是简化的拒绝并删除当前消息。

1. `Nack`（Negative Acknowledgment）：通过调用 `Nack` 方法可以将消息拒绝并重新投递到队列中。它有两个参数：`multiple` 和 `requeue`。`multiple` 表示是否拒绝多个消息，如果设置为 `true`，那么当前消息及其之前的所有未确认消息都会被拒绝。`requeue` 表示被拒绝的消息是否重新进入队列，如果设置为 `true`，那么被拒绝的消息将重新投递给其他消费者。
2. `Ack`（Acknowledgment）：通过调用 `Ack` 方法可以明确地确认消息已经成功处理，并将其从队列中删除。这意味着 RabbitMQ 可以安全地假设该消息已被完全处理，从而不再重新发送给其他消费者。
3. `Reject`：通过调用 `Reject` 方法可以拒绝消息，并将其从队列中删除或转发到死信队列。与 `Nack` 方法不同，`Reject` 方法没有参数来指定是否拒绝多个消息，它总是只拒绝当前消息。

