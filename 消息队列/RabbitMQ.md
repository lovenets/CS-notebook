# 基本概念

![RMQ基本概念](img\RMQ基本概念.png)

1. Message

   消息，消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。

2. ==Publisher==
    消息的生产者，也是一个向交换器发布消息的客户端应用程序。

3. ==Exchange==
    交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。在 RabbitMQ 中，消息并不是直接发送到队列中，而是先进入交换器，然后再由交换器决定发不发、发给哪些队列。

4. ==Binding==
    绑定，用于消息队列和交换器之间的关联，就是将交换器和队列进行绑定。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。

5. ==Queue==
    消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。

6. Connection
    网络连接，比如一个 TCP 连接。

7. Channel
    信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的 TCP 连接内的虚拟连接，AMQP 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁 TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条 TCP 连接。

8. ==Consumer==
    消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。

9. Virtual Host
    虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 / 。

10. Broker
     表示消息队列服务器实体。

# 消息路由

AMQP 中增加了 Exchange 和 Binding 的角色。生产者把消息发布到 Exchange 上，消息最终到达队列并被消费者接收，而 Binding 决定交换器的消息应该发送到哪些队列。交换器 -> 队列就可以看作一条路由。

# 交换器类型

## 1.direct

消息中的路由键（routing key）如果和 Binding 中的 binding key 一致， 交换器就将消息发到对应的队列中。路由键与队列名完全匹配，如果一个队列绑定到交换机要求路由键为“dog”，则只转发 routing key 标记为“dog”的消息，不会转发“dog.puppy”，也不会转发“dog.guard”等等。它是完全匹配、==单播==的模式。

![Direct Exchange](img\Direct Exchange.png)

## 2.fanout

每个发到 fanout 类型交换器的消息都会分到所有绑定的队列上去。fanout 交换器不处理路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。fanout 类型转发消息是最快的，用于==广播==。

![Fanout Exchange](img\Fanout Exchange.png)

## 3.topic

topic 交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。它将路由键和绑定键的字符串切分成单词，这些单词之间用点隔开。它同样也会识别两个通配符：符号“#”和符号“*”。#匹配0个或多个单词，星号匹配一个单词。这种交换器用于典型的 pub/sub 场景。

![Topic Exchange](img\Topic Exchange.png)

# Spring Boot 和 RabbitMQ 简单整合

## 1.创建队列

在 RabbitMQ 配置类中创建几个队列：

```java
@Configuration
public class RabbitFanoutConfig {
    @Bean
    public Queue queue1(){
        return new Queue("queue1");
    }

    @Bean
    public Queue queue2(){
        return new Queue("queue2");
    }
}

```

## 2.编写 receiver

```java
@Component
@RabbitListener(queues = "queue1") 
public class MyReceiver1 {
    @RabbitHandler
    public void receive(String message){
        System.out.println("Receiver1: " + message);
    }
}
```

（1）`@RabbitListener`这个注解用来指定 receiver 要接受哪个队列中的消息，可以用数组指定多个队列。

（2）`@RabbitHandler`这个注解用在接收消息的方法上，方法的参数就是一个 String。

## 3.根据情况配置 Exchange 和 sender

在实际开发中要根据实际情况配置 Exchange。

### （1）单播：把消息发送到指定的接收者

如果是单播的情况，可以直接使用 RabbitMQ 默认的 Exchange 而无需自己配置，所以直接实现 sender 的逻辑即可：

```java
@RestController
public class MQController {

    @Autowired
    private  RabbitTemplate template;
    
    // 单播
    @GetMapping("/receiver1")
    public void receive1(){
        template.convertAndSend("queue1" , "This message is sent to receiver1.");
    }

    @GetMapping("/receiver2")
    public void receive2(){
        template.convertAndSend("queue2" , "This message is sent to receiver2.");
    }
}

```

`RabbitTemplate`这个类用来发送消息，核心方法是`convertAndSend`，在单播的模式下有两个参数，第一个参数可以理解为队列的名称，指定消息要发送到哪个队列；第二个参数是消息。

### （2）广播

在广播的情况下需要先在配置类中配置一个 Fanout Exchange：

```java
    // 以下是将 Exchange 和 队列进行绑定
    // 任何发送到 Fanout Exchange 的消息都会被转发到与该 Exchange 绑定的所有队列上
    @Bean
    public Binding bindingExchangeQueue1(Queue queue1,FanoutExchange fanoutExchange){
        return BindingBuilder.bind(queue1).to(fanoutExchange);
    }

    @Bean
    public Binding bindingExchangeQueue2(Queue queue2,FanoutExchange fanoutExchange){
        return BindingBuilder.bind(queue2).to(fanoutExchange);
    }
```

实现 sender：

```java
@RestController
public class MQController {

    @Autowired
    private  RabbitTemplate template;

    @Autowired
    private  FanoutExchange fanoutExchange;
    
    // 广播
    @GetMapping("/all")
    public void receiveAll(){
        template.convertAndSend(fanoutExchange.getName(), "", "This message is sent to all receivers.");
    }
}
```

广播的模式下`convertAndSend`有三个参数，第一个是 Fanout Exchange 的名称；第二个是队列的名称，由于是广播，所以直接写 ""；第三个是消息。

### （3）订阅 topic

在这种模式下，receiver 只会接收到与自己订阅的 topic 相关的消息。

配置 Topic Exchange 时，在绑定这一步中`with`方法的参数就可以理解为 topic，队列只会接受指定 topic 相关的消息。如果使用通配符的话，通配符和普通文本之间要用“.”隔开。

```java
    // Topic Exchange 配置
    @Bean
    public TopicExchange topicExchange(){
        return new TopicExchange("topicExchange");
    }
    // queue1 只接受 topic 以“topic1.”开头的消息
    @Bean
    public Binding topic1ExchangeQueue1(Queue queue1,TopicExchange topicExchange){
        return BindingBuilder.bind(queue1).to(topicExchange).with("topic1.#");
    }
    // queue2 只接受 topic 以“topic2.”开头的消息
    @Bean
    public Binding topic2ExchangeQueue2(Queue queue2,TopicExchange topicExchange){
        return BindingBuilder.bind(queue2).to(topicExchange).with("topic2.#");
    }
```

```java
@RestController
public class MQController {

    @Autowired
    private  RabbitTemplate template;

    @Autowired
    private TopicExchange topicExchange;
    
    // 订阅 topic
    @GetMapping("/topic1")
    public void receiveTopic1(){
        template.convertAndSend(topicExchange.getName(), "topic1.ABC",
                "This message whose topic is topic1.ABC is sent to receiver1.");
        template.convertAndSend(topicExchange.getName(), "topic1.DEF",
                "This message whose topic is topic1.DEF is sent to receiver1.");
    }

    @GetMapping("/topic2")
    public void receiveTopic2(){
        template.convertAndSend(topicExchange.getName(), "topic2.ABC",
                "This message whose topic is topic2.ABC is sent to receiver2.");
        template.convertAndSend(topicExchange.getName(), "topic2.DEF",
                "This message whose topic is topic2.DEF is sent to receiver2.");
    }
}
```

`convertAndSend`的第二个参数就是 topic。





