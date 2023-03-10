---
layout: post
title: Spring Boot 消息队列 ActiveMQ 入门
category: CSS
tags: [css]
---

## Spring Boot 消息队列 ActiveMQ 入门

摘要: 原创出处 http://www.iocoder.cn/Spring-Boot/ActiveMQ/

------

------

> 本文在提供完整代码示例，可见 https://github.com/YunaiV/SpringBoot-Labs 的 [lab-32](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32) 目录。
>
> 原创不易，给点个 [Star](https://github.com/YunaiV/SpringBoot-Labs/stargazers) 嘿，一起冲鸭！

# 1. 概述

如果胖友还没了解过分布式消息队列 [ActiveMQ](https://activemq.apache.org/) ，建议先阅读下艿艿写的 [《芋道 ActiveMQ 极简入门》](http://www.iocoder.cn/ActiveMQ/install/?self) 文章。虽然这篇文章标题是安装部署，实际可以理解成《一文带你快速入门 ActiveMQ》，哈哈哈。

考虑这是 ActiveMQ 如何在 Spring Boot 整合与使用的文章，所以还是简单介绍下 ActiveMQ 是什么？

> FROM [《JMS 消息服务器 ActiveMQ》](https://www.oschina.net/p/activemq)
>
> ActiveMQ 是 Apache 出品，最流行的，能力强劲的开源消息总线。
>
> ActiveMQ 是一个完全支持 JMS1.1 和 J2EE1.4 规范的 JMS Provider 实现，尽管 JMS 规范出台已经是很久的事情了,但是 JMS 在当今的 J2EE 应用中间仍然扮演着特殊的地位。
>
> 主要特点：
>
> 1. 多种语言和协议编写客户端。语言: Java, C, C++, C#, Ruby, Perl, Python, PHP。应用协议: OpenWire,Stomp REST,WS Notification,XMPP,AMQP
> 2. 完全支持JMS1.1 和 J2EE1.4 规范 (持久化,XA消息,事务)
> 3. 对 Spring 的支持,ActiveMQ 可以很容易内嵌到使用 Spring 的系统里面去,而且也支持 Spring2.0 的特性
> 4. 通过了常见 J2EE 服务器(如 Geronimo,JBoss 4, GlassFish,WebLogic)的测试,其中通过 JCA 1.5 resource adaptors 的配置,可以让ActiveMQ可以自动的部署到任何兼容J2EE 1.4 商业服务器上
> 5. 支持多种传送协议:in-VM,TCP,SSL,NIO,UDP,JGroups,JXTA
> 6. 支持通过 JDBC 和 journal 提供高速的消息持久化
> 7. 从设计上保证了高性能的集群,客户端-服务器,点对点
> 8. 支持 Ajax
> 9. 支持与 Axis 的整合
> 10. 可以很容易得调用内嵌 JMS provider,进行测试

在本文中，我们会比 [《芋道 ActiveMQ 极简入门》](http://www.iocoder.cn/ActiveMQ/install/?self) 提供更多的生产者 Producer 和消费者 Consumer 的使用示例。例如说：

- Producer 同步与异步发送消息的方式。
- Producer 发送**顺序**消息，Consumer **顺序**消费消息。
- Producer 发送**定时**消息。
- Producer 发送**事务**消息。TODO
- Consumer **广播**和**集群**消费消息。

# 2. Spring-JMS

在 Spring 体系中，提供了 [Spring-JMS](https://github.com/spring-projects/spring-framework/tree/master/spring-jms) 组件，实现对 [JMS](https://mvnrepository.com/artifact/javax.jms/javax.jms-api) 规范的集成。我们来看看 Spring 文档对 Spring JMS 的描述：

> FROM [《Spring 文档 —— JMS (Java Message Service)》](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/integration.html#jms)
>
> Spring provides a JMS integration framework that simplifies the use of the JMS API in much the same way as Spring’s integration does for the JDBC API.
> Spring 提供了一个 JMS 的集成框架，简化了 JMS API 的使用，就像 Spring 对 JDBC API 的集成一样。
>
> JMS can be roughly divided into two areas of functionality, namely the production and consumption of messages.
>
> - The [JmsTemplate](https://github.com/spring-projects/spring-framework/blob/master/spring-jms/src/main/java/org/springframework/jms/core/JmsTemplate.java) class is used for message production and synchronous message reception.
> - For asynchronous reception similar to Java EE’s message-driven bean style, Spring provides a number of message-listener containers that you can use to create Message-Driven POJOs (MDPs). Spring also provides a declarative way to create message listeners.
>
> JMS 可以大致分为两块功能，即消息的发送和消费。
>
> - JmsTemplate 类，用于消息的发送和消息的同步接收。
> - 对于类似 Java EE 的消息驱动 Bean 形式的异步接收，Spring 提供了大量用于创建消息驱动 POJOs（MDPs）的消息监听器。Spring 还提供了一种创建消息侦听器的**声明式**方法。

- 英文跟艿艿一样不好的胖友，可以看看[《【译】Spring Framework Reference —— JMS 部分》](https://my.oschina.net/landas/blog/858784) 。
- 因为 ActiveMQ 提供了对 JMS 规范的支持，自然 Spring-JMS 可以访问 ActiveMQ 消息队列，更加方便的实现消息的发送与消费。

在 [Spring-Boot](https://spring.io/projects/spring-boot) 项目中，提供了 ActiveMQ 的自动化配置，所以我们仅需引入 [`spring-boot-starter-activemq`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-activemq) 依赖，即可愉快的使用。

# 3. 快速入门

> 示例代码对应仓库：[lab-32-activemq-demo](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo) 。

本小节，我们先来对 Spring-JMS 做一个快速入门，实现 Producer 同步与异步发送消息到 Queue 中，同时创建一个 Consumer 消费消息。

考虑到一个应用既可以使用生产者 Producer ，又可以使用消费者 Consumer ，所以示例就做成一个 [lab-32-activemq-demo](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo) 项目。

## 3.1 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo/pom.xml) 文件中，引入相关依赖。



```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>lab-32-activemq-demo</artifactId>

    <dependencies>
        <!-- 实现对 ActiveMQ 的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-activemq</artifactId>
        </dependency>

        <!-- 方便等会写单元测试 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>

显示详细信息
```



- 具体每个依赖的作用，胖友自己认真看下艿艿添加的所有注释噢。

## 3.2 应用配置文件

在 [`resources`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo/src/main/resources) 目录下，创建 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo/src/main/resources/application.yaml) 配置文件。配置如下：



```
spring:
  # ActiveMQ 配置项，对应 ActiveMQProperties 配置类
  activemq:
    broker-url: tcp://127.0.0.1:61616 # Activemq Broker 的地址
    user: admin # 账号
    password: admin # 密码
    packages:
      trust-all: true # 可信任的反序列化包
```



- 在 `spring.activemq` 配置项，设置 Kafka 的配置，对应 [ActiveMQProperties](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jms/activemq/ActiveMQProperties.java) 配置类。
- Spring Boot 提供的 [ActiveMQAutoConfiguration](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jms/activemq/ActiveMQAutoConfiguration.java) 自动化配置类，实现 ActiveMQ 的自动配置，创建相应的 Producer 和 Consumer 。
- `spring.activemq.packages.trust-all` 配置项，配置可信赖所有的 `package` 包。因为 ActiveMQ 在反序列化 POJO 的消息时，考虑到安全性，如果非可信赖的 Java 类，会抛出 `"This class is not trusted to be serialized"` 的异常。😈 想要尝试下效果的胖友，可以选择去掉这个配置，很酸爽。

## 3.3 Application

创建 [`Application.java`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/Application.java) 类，配置 `@SpringBootApplication` 注解即可。代码如下：



```
// Application.java

@SpringBootApplication
@EnableAsync // 开启异步
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```



- 我们额外添加了 `@EnableAsync` 注解，因为我们稍后要使用 Spring 提供的异步调用的功能。不了解这块的胖友，可以看看艿艿写的 [《芋道 Spring Boot 异步任务入门》](http://www.iocoder.cn/Spring-Boot/Async-Job/?self) 文章。

## 3.4 Demo01Message

在 [`cn.iocoder.springboot.lab32.activemqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/message) 包下，创建 [Demo01Message](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/message/Demo01Message.java) 消息类，提供给当前示例使用。代码如下：



```
// Demo01Message.java

public class Demo01Message implements Serializable {

    public static final String QUEUE = "QUEUE_DEMO_01";

    /**
     * 编号
     */
    private Integer id;

    // ... 省略 set/get/toString 方法

}

显示详细信息
```



- 注意，要实现 Java Serializable 序列化接口。因为 JMS 规范要求 POJO 消息类，需要实现 Serializable 接口。
- 在消息类里，我们枚举了 Queue 的名字。

我们无需中去 ActiveMQ 或者 Spring-JMS **主动**声明这个 ActiveMQ Queue 。因为 Consumer 订阅该 Queue 时，会**自动**进行创建。又或者 Producer 在发送消息到 Queue 中时，也会**自动**进行创建。

## 3.5 Demo01Producer

在 [`cn.iocoder.springboot.lab32.activemqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/producer) 包下，创建 [Demo01Producer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/producer/Demo01Producer.java) 类，它会使用 Spring-JMS 封装提供的 JmsMessagingTemplate ，实现发送消息。代码如下：



```
// Demo01Producer.java

@Component
public class Demo01Producer {

    @Autowired
    private JmsMessagingTemplate jmsTemplate;

    public void syncSend(Integer id) {
        // 创建 Demo01Message 消息
        Demo01Message message = new Demo01Message();
        message.setId(id);
        // 同步发送消息
        jmsTemplate.convertAndSend(Demo01Message.QUEUE, message);
    }

    @Async
    public ListenableFuture<Void> asyncSend(Integer id) {
        try {
            // 发送消息
            this.syncSend(id);
            // 返回成功的 Future
            return AsyncResult.forValue(null);
        } catch (Throwable ex) {
            // 返回异常的 Future
            return AsyncResult.forExecutionException(ex);
        }
    }

}

显示详细信息
```



- ```
  jmsTemplate
  ```



属性，是



JmsMessagingTemplate



对象，而不是



JmsTemplate



。

- JmsTemplate 是 JMS API 的封装，简化消息的发送与接收。
- JmsMessagingTemplate 是将 JmsTemplate 集成到 [Spring-Messaging](https://github.com/spring-projects/spring-framework/tree/master/spring-messaging/src/main/java/org/springframework/messaging) 体系中，其内部调用的还是 JmsTemplate 的方法。

- `#syncSend(Integer id)` 方法，调用 JmsMessagingTemplate 的同步发送消息方法。

- `#asyncSend(Integer id)` 方法，通过 `@Async` 注解，声明异步调用该方法，从而实现异步消息到 ActiveMQ 中。因为 JmsMessagingTemplate 并未像 [KafkaTemplate](https://github.com/spring-projects/spring-kafka/blob/master/spring-kafka/src/main/java/org/springframework/kafka/core/KafkaTemplate.java) 或 [RocketMQTemplate](https://github.com/apache/rocketmq-spring/blob/master/rocketmq-spring-boot/src/main/java/org/apache/rocketmq/spring/core/RocketMQTemplate.java) 直接提供了异步发送消息的方法，所以我们需要结合 Spring 异步调用来实现。

## 3.6 Demo01Consumer

在 [`cn.iocoder.springboot.lab32.activemqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/consumer) 包下，创建 [Demo01Consumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/consumer/Demo01Consumer.java) 类，消费消息。代码如下：



```
// Demo01Consumer.java

@Component
public class Demo01Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @JmsListener(destination = Demo01Message.QUEUE)
    public void onMessage(Demo01Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }

//    @JmsListener(destination = Demo01Message.QUEUE)
//    public void onMessage(javax.jms.Message message) {
//        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
//    }

}

显示详细信息
```



## 3.7 简单测试

创建 [Demo01ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo/src/test/java/cn/iocoder/springboot/lab32/activemqdemo/producer/Demo01ProducerTest.java) 测试类，编写单元测试方法，调用 Demo01Producer 两个发送消息的方式。代码如下：



```
// Demo01ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo01ProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private Demo01Producer producer;

    @Test
    public void testSyncSend() throws InterruptedException {
        // 发送消息
        int id = (int) (System.currentTimeMillis() / 1000);
        producer.syncSend(id);
        logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

    @Test
    public void testAsyncSend() throws InterruptedException {
        int id = (int) (System.currentTimeMillis() / 1000);
        producer.asyncSend(id).addCallback(new ListenableFutureCallback<Void>() {

            @Override
            public void onFailure(Throwable e) {
                logger.info("[testASyncSend][发送编号：[{}] 发送异常]]", id, e);
            }

            @Override
            public void onSuccess(Void aVoid) {
                logger.info("[testASyncSend][发送编号：[{}] 发送成功，发送成功]", id);
            }

        });
        logger.info("[testASyncSend][发送编号：[{}] 调用完成]", id);

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}

显示详细信息
```



- 比较简单，胖友自己看下两个单元测试方法。

我们来执行 `#testSyncSend()` 方法，测试同步发送消息。控制台输出如下：



```
# Producer 同步发送消息成功。
2019-12-15 00:19:18.736  INFO 87164 --- [           main] c.i.s.l.r.producer.Demo01ProducerTest    : [testSyncSend][发送编号：[1575908358] 发送成功]

# Demo01Consumer 成功消费了该消息
2019-12-15 00:19:18.751  INFO 87164 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo01Consumer        : [onMessage][线程编号:17 消息内容：Demo01Message{id=1575908358}]
```



- 同步发送的消息，成功被消费。

我们再来执行 `#tesSyncSendDefault()` 方法，测试另一个同步发送消息。控制台输出如下：



```
# Producer 同步发送消息成功。
2019-12-15 11:50:38.857  INFO 74430 --- [           main] c.i.s.l.a.producer.Demo01ProducerTest    : [testSyncSend][发送编号：[1576381838] 发送成功]

# Demo01Consumer 成功消费了该消息
2019-12-15 11:50:38.860  INFO 74430 --- [enerContainer-1] c.i.s.l.a.consumer.Demo01Consumer        : [onMessage][线程编号:18 消息内容：Demo01Message{id=1576381838}]
```



- 同步发送的消息，成功也被消费。

我们最后来执行 `#testAsyncSend()` 方法，测试异步发送消息。控制台输出如下：



```
# Producer 异步发送消息的调用完成。
2019-12-15 11:52:43.156  INFO 74582 --- [           main] c.i.s.l.a.producer.Demo01ProducerTest    : [testASyncSend][发送编号：[1576381963] 调用完成]

# Producer 异步发送消息成功。【回调】
2019-12-15 11:52:43.214  INFO 74582 --- [         task-1] c.i.s.l.a.producer.Demo01ProducerTest    : [testASyncSend][发送编号：[1576381963] 发送成功，发送成功]

# Demo01Consumer 成功消费了该消息
2019-12-15 11:52:43.218  INFO 74582 --- [enerContainer-1] c.i.s.l.a.consumer.Demo01Consumer        : [onMessage][线程编号:18 消息内容：Demo01Message{id=1576381963}]
```



- 异步发送的消息，成功也被消费。

# 4. 消息模式

> 示例代码对应仓库：[lab-32-activemq-demo-message-model](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-message-model)

在 JMS 规范中，定义了两种消息模式：

- 点对点（point to point）：基于 Queue 队列的方式。
- 发布/订阅（publish/subscribe）：基于 Topic 主题的方式。

具体的概念，艿艿就先不解释，胖友可以看看[《消息队列两种模式：点对点与发布订阅》](http://www.iocoder.cn/Fight/There-are-two-modes-of-message-queuing-point-to-point-and-publish-subscription/?self)文章。🙂 实际上，我们在[「3. 快速入门」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#)中，就采用的是点对点的消息模式。

如果胖友有使用过 RocketMQ 或者 Kafka 消息队列，可能比较习惯的叫法是：

> **集群消费（Clustering）**：对应「点对点」 集群消费模式下，相同 Consumer Group 的每个 Consumer 实例平均分摊消息。
>
> **广播消费（Broadcasting）**：对应「发布订阅」 广播消费模式下，相同 Consumer Group 的每个 Consumer 实例都接收全量的消息。

😈 考虑到艿艿自己的习惯，下文我们统一采用集群消费和广播消费叫法。

下面，我们分别在[「4.1 集群消费」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#)和[「4.2 广播消费」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#)的示例代码。两个示例，我们都会放在一个 [lab-32-activemq-demo-message-model](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-message-model) 项目。

## 4.1 集群消费

在 ActiveMQ 中，如果多个 Consumer 订阅相同的 Queue ，那么每一条消息有且仅会被一个 Consumer 所消费。这个特性，就为我们实现集群消费提供了基础。

在本示例中，我们会把一个 Queue 作为一个 Consumer Group ，同时创建消费该 Queue 的 Consumer 。这样，在我们启动多个 JVM 进程时，就会有多个 Consumer 消费该 Queue ，从而实现集群消费的效果。

下面，让我们开始集群消费的示例。

### 4.1.1 引入依赖

和 [「3.1 引入依赖」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#) 一致，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-message-model/pom.xml) 文件。

### 4.1.2 应用配置文件

和 [「3.2 应用配置文件」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#) 一致，见 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-message-model/src/main/resources/application.yaml) 文件。

### 4.1.3 ClusteringMessage

在 [`cn.iocoder.springboot.lab32.activemqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-message-model/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/message/) 包下，创建 [ClusteringMessage](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-message-model/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/message/ClusteringMessage.java) 消息类，提供给当前示例使用。代码如下：



```
// ClusteringMessage.java

public class ClusteringMessage implements Serializable {

    public static final String QUEUE = "QUEUE_CLUSTERING";

    /**
     * 编号
     */
    private Integer id;

    // ... 省略 set/get/toString 方法

}

显示详细信息
```



- 在消息类里，我们枚举了 Queue 的名字。

### 4.1.4 ActiveMQConfig

在 [`cn.iocoder.springboot.lab32.activemqdemo.config`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-message-model/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/config) 包下，创建 [ActiveMQConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-message-model/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/config/ActiveMQConfig.java) 配置类，添加集群消费需要的配置。代码如下：



```
// ActiveMQConfig.java

public static final String CLUSTERING_JMS_LISTENER_CONTAINER_FACTORY_BEAN_NAME = "clusteringJmsListenerContainerFactory";

public static final String CLUSTERING_JMS_TEMPLATE_BEAN_NAME = "clusteringJmsTemplate";

@Bean(CLUSTERING_JMS_LISTENER_CONTAINER_FACTORY_BEAN_NAME)
public DefaultJmsListenerContainerFactory clusteringJmsListenerContainerFactory(
        DefaultJmsListenerContainerFactoryConfigurer configurer, ConnectionFactory connectionFactory) {
    DefaultJmsListenerContainerFactory factory = new DefaultJmsListenerContainerFactory();
    configurer.configure(factory, connectionFactory);
    factory.setPubSubDomain(false);
    return factory;
}

@Bean(CLUSTERING_JMS_TEMPLATE_BEAN_NAME)
public JmsMessagingTemplate clusteringJmsTemplate(ConnectionFactory connectionFactory) {
    // 创建 JmsTemplate 对象
    JmsTemplate template = new JmsTemplate(connectionFactory);
    template.setPubSubDomain(false);
    // 创建 JmsMessageTemplate
    return new JmsMessagingTemplate(template);
}

显示详细信息
```



- 通过



  ```
  spring.jms.pub-sub-domain
  ```



配置项，控制创建的 JmsTemplate 和 DefaultJmsListenerContainerFactory Bean 是针对



  ```
  true
  ```



广播消费，还是



  ```
  false
  ```



集群消费。

- 因为在本小节的示例项目中，我们既要支持集群消费，又要支持广播消费，所以我们需要**手动**创建两套 JmsTemplate 和 DefaultJmsListenerContainerFactory Bean ，分别给[「4.1 集群消费」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#)和[「4.2 广播消费」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#)使用。在这里，我们先给集群消费创建了一套 JmsTemplate 和 DefaultJmsListenerContainerFactory 。
- 如果胖友的项目中，只需要持集群消费或广播消费的其中一种，仅仅需要 `spring.jms.pub-sub-domain` 配置项即可，无需**手动**一套 JmsTemplate 和 DefaultJmsListenerContainerFactory Bean 。

- 另外，因为我们在 Producer 中，使用的是 JmsMessagingTemplate 来发送消息，所以这里最终创建的是 JmsMessagingTemplate Bean 。

### 4.1.5 ClusteringProducer

在 [`cn.iocoder.springboot.lab32.activemqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-message-model/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/producer) 包下，创建 [ClusteringProducer](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-message-model/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/producer/ClusteringProducer.java) 类，它会使用 Spring-JMS 封装提供的 JmsMessagingTemplate ，实现发送消息。代码如下：



```
// ClusteringProducer.java

@Component
public class ClusteringProducer {

    @Resource(name = ActiveMQConfig.CLUSTERING_JMS_TEMPLATE_BEAN_NAME)
    private JmsMessagingTemplate jmsTemplate;

    public void syncSend(Integer id) {
        // 创建 ClusteringMessage 消息
        ClusteringMessage message = new ClusteringMessage();
        message.setId(id);
        // 同步发送消息
        jmsTemplate.convertAndSend(ClusteringMessage.QUEUE, message);
    }

}

显示详细信息
```



- 注意，要注入对应的 Bean 名字为 `ActiveMQConfig.CLUSTERING_JMS_TEMPLATE_BEAN_NAME` 的 JmsMessagingTemplate 对象。

### 4.1.6 ClusteringConsumer

在 [`cn.iocoder.springboot.lab32.activemqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-message-model/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/consumer) 包下，创建 [ClusteringConsumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-message-model/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/consumer/ClusteringConsumer.java) 类，**集群**消费消息。代码如下：



```
// ClusteringConsumer.java

@Component
public class ClusteringConsumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @JmsListener(destination = ClusteringMessage.QUEUE,
            containerFactory = ActiveMQConfig.CLUSTERING_JMS_LISTENER_CONTAINER_FACTORY_BEAN_NAME)
    public void onMessage(ClusteringMessage message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }

}

显示详细信息
```



- 在 `@JmsListener` 注解的 `containerFactory` 属性，设置 Bean 名字为 `ActiveMQConfig.CLUSTERING_JMS_TEMPLATE_BEAN_NAME` 的 DefaultJmsListenerContainerFactory 对象。

### 4.1.7 简单测试

创建 [ClusteringProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-message-model/src/test/java/cn/iocoder/springboot/lab32/activemqdemo/producer/ClusteringProducerTest.java) 测试类，用于测试集群消费。代码如下：



```
// ClusteringProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class ClusteringProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private ClusteringProducer producer;

    @Test
    public void mock() throws InterruptedException {
        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

    @Test
    public void testSyncSend() throws InterruptedException {
        // 发送 3 条消息
        for (int i = 0; i < 3; i++) {
            int id = (int) (System.currentTimeMillis() / 1000);
            producer.syncSend(id);
            logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);
        }

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}

显示详细信息
```



首先，执行 `#mock()` 测试方法，先启动一个消费 `"QUEUE_CLUSTERING"` 这个 Queue 的 Consumer 节点。

然后，执行 `#testSyncSend()` 测试方法，再启动一个消费 `"QUEUE_CLUSTERING"` 这个 Queue 的 Consumer 节点。同时，该测试方法，调用 `ClusteringProducer#syncSend(id)` 方法，同步发送了 3 条消息。控制台输出如下：



```
// #### testSyncSend 方法对应的控制台 ####

# Producer 同步发送消息成功
2019-12-15 13:43:07.010  INFO 79244 --- [           main] c.i.s.l.a.p.ClusteringProducerTest       : [testSyncSend][发送编号：[1576388586] 发送成功]
2019-12-15 13:43:07.015  INFO 79244 --- [           main] c.i.s.l.a.p.ClusteringProducerTest       : [testSyncSend][发送编号：[1576388587] 发送成功]
2019-12-15 13:43:07.017  INFO 79244 --- [           main] c.i.s.l.a.p.ClusteringProducerTest       : [testSyncSend][发送编号：[1576388587] 发送成功]

# ClusteringConsumer 消费了 1 条消息
2019-12-15 13:43:07.021  INFO 79244 --- [enerContainer-1] c.i.s.l.a.consumer.ClusteringConsumer    : [onMessage][线程编号:18 消息内容：ClusteringtMessage{id=1576388587}]

// ### mock 方法对应的控制台 ####

# ClusteringConsumer 消费了 2 条消息
2019-12-15 13:43:07.029  INFO 79234 --- [enerContainer-1] c.i.s.l.a.consumer.ClusteringConsumer    : [onMessage][线程编号:18 消息内容：ClusteringtMessage{id=1576388586}]
2019-12-15 13:43:07.034  INFO 79234 --- [enerContainer-1] c.i.s.l.a.consumer.ClusteringConsumer    : [onMessage][线程编号:18 消息内容：ClusteringtMessage{id=1576388587}

显示详细信息
```



- 3 条消息，都仅被 **两个** Consumer 节点的**一个**进行消费。符合集群消费的预期~

### 4.1.8 多集群下的集群消费

在[「4.1 集群消费」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#)的示例中，我们只提供了**单集群**下的集群消费。实际业务场景下，我们可能会存在**多集群**的集群消费。不了解的胖友，可以看看[《ActiveMQ 之 VirtualTopic 是什么？》](https://www.cnblogs.com/Peter2014/p/9389600.html) 。

正如该文章的标题，需要通过 ActiveMQ **自定义**的 VirtualTopic 虚拟主题，而非 JMS 所提供的。具体的示例，胖友可以先看如下两篇文章：

## 4.2 广播消费

在 ActiveMQ 中，如果多个 Consumer 订阅相同的 Topic ，那么每一条消息**都**会被一个 Consumer 所消费。这个特性，就为我们实现广播消费提供了基础。

下面，让我们开始集群消费的示例。

### 4.2.1 BroadcastMessage

在 [`cn.iocoder.springboot.lab32.activemqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-message-model/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/message/) 包下，创建 [BroadcastMessage](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-message-model/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/message/BroadcastMessage.java) 消息类，提供给当前示例使用。代码如下：



```
// ClusteringMessage.java

public class BroadcastMessage implements Serializable {

    public static final String TOPIC = "TOPIC_BROADCASTING";

    /**
     * 编号
     */
    private Integer id;

    // ... 省略 set/get/toString 方法

}

显示详细信息
```



- 在消息类里，我们枚举了 Topic 的名字。

### 4.2.2 ActiveMQConfig

修改 [ActiveMQConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-message-model/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/config/ActiveMQConfig.java) 配置类，添加广播消费需要的配置。代码如下：



```
// ActiveMQConfig.java

public static final String BROADCAST_JMS_LISTENER_CONTAINER_FACTORY_BEAN_NAME = "broadcastJmsListenerContainerFactory";

public static final String BROADCAST_JMS_TEMPLATE_BEAN_NAME = "broadcastJmsTemplate";

    @Bean(BROADCAST_JMS_LISTENER_CONTAINER_FACTORY_BEAN_NAME)
public DefaultJmsListenerContainerFactory broadcastJmsListenerContainerFactory(
        DefaultJmsListenerContainerFactoryConfigurer configurer, ConnectionFactory connectionFactory) {
    DefaultJmsListenerContainerFactory factory = new DefaultJmsListenerContainerFactory();
    configurer.configure(factory, connectionFactory);
    factory.setPubSubDomain(true);
    return factory;
}

@Bean(BROADCAST_JMS_TEMPLATE_BEAN_NAME)
public JmsMessagingTemplate broadcastJmsTemplate(ConnectionFactory connectionFactory) {
    // 创建 JmsTemplate 对象
    JmsTemplate template = new JmsTemplate(connectionFactory);
    template.setPubSubDomain(true);
    // 创建 JmsMessageTemplate
    return new JmsMessagingTemplate(template);
}

显示详细信息
```



- 在这里，我们给集群消费创建了一套 JmsTemplate 和 DefaultJmsListenerContainerFactory 。

### 4.2.3 BroadcastProducer

在 [`cn.iocoder.springboot.lab32.activemqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-message-model/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/producer) 包下，创建 [BroadcastProducer](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-message-model/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/producer/BroadcastProducer.java) 类，它会使用 Spring-JMS 封装提供的 JmsMessagingTemplate ，实现发送消息。代码如下：



```
// BroadcastProducer.java

@Component
public class BroadcastProducer {

    @Resource(name = ActiveMQConfig.BROADCAST_JMS_TEMPLATE_BEAN_NAME)
    private JmsMessagingTemplate jmsMessagingTemplate;

    public void syncSend(Integer id) {
        // 创建 BroadcastMessage 消息
        BroadcastMessage message = new BroadcastMessage();
        message.setId(id);
        // 同步发送消息
        jmsMessagingTemplate.convertAndSend(BroadcastMessage.TOPIC, message);
    }

}

显示详细信息
```



- 和[「4.1.5 ClusteringProducer」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#)是一致，只是使用了不同的 Topic 和消息。
- 注意，要注入对应的 Bean 名字为 `ActiveMQConfig.BROADCAST_JMS_TEMPLATE_BEAN_NAME` 的 JmsMessagingTemplate 对象。

### 4.2.4 BroadcastConsumer

在 [`cn.iocoder.springboot.lab32.activemqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-message-model/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/consumer) 包下，创建 [BroadcastConsumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-message-model/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/consumer/BroadcastConsumer.java) 类，**广播**消费消息。代码如下：



```
// BroadcastConsumer.java

@Component
public class BroadcastConsumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @JmsListener(destination = BroadcastMessage.TOPIC,
            containerFactory = ActiveMQConfig.BROADCAST_JMS_LISTENER_CONTAINER_FACTORY_BEAN_NAME)
    public void onMessage(BroadcastMessage message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }

}

显示详细信息
```



- 总体和[「4.1.6 ClusteringConsumer」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#)是一致的。
- 在 `@JmsListener` 注解的 `containerFactory` 属性，设置 Bean 名字为 `ActiveMQConfig.BROADCAST_JMS_LISTENER_CONTAINER_FACTORY_BEAN_NAME` 的 DefaultJmsListenerContainerFactory 对象。

### 4.2.5 简单测试

创建 [BroadcastProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-message-model/src/test/java/cn/iocoder/springboot/lab32/activemqdemo/producer/BroadcastProducerTest.java) 测试类，用于测试广播消费。代码如下：



```
// BroadcastProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class BroadcastProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private BroadcastProducer producer;

    @Test
    public void mock() throws InterruptedException {
        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

    @Test
    public void testSyncSend() throws InterruptedException {
        for (int i = 0; i < 3; i++) {
            int id = (int) (System.currentTimeMillis() / 1000);
            producer.syncSend(id);
            logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);
        }

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}

显示详细信息
```



首先，执行 `#mock()` 测试方法，先启动一个消费 `"TOPIC_BROADCASTING"` 这个 Topic 的 Consumer 节点。

然后，执行 `#testSyncSend()` 测试方法，再启动一个消费 `"TOPIC_BROADCASTING"` 这个 Topic 的 Consumer 节点。同时，该测试方法，调用 `BroadcastProducer#syncSend(id)` 方法，同步发送了 3 条消息。控制台输出如下：



```
// #### testSyncSend 方法对应的控制台 ####

# Producer 同步发送消息成功
2019-12-15 14:11:41.256  INFO 80487 --- [           main] c.i.s.l.a.p.BroadcastProducerTest        : [testSyncSend][发送编号：[1576390301] 发送成功]
2019-12-15 14:11:41.258  INFO 80487 --- [           main] c.i.s.l.a.p.BroadcastProducerTest        : [testSyncSend][发送编号：[1576390301] 发送成功]
2019-12-15 14:11:41.259  INFO 80487 --- [           main] c.i.s.l.a.p.BroadcastProducerTest        : [testSyncSend][发送编号：[1576390301] 发送成功]

# BroadcastConsumer 消费了 3 条消息
2019-12-15 14:11:41.259  INFO 80487 --- [enerContainer-1] c.i.s.l.a.consumer.BroadcastConsumer     : [onMessage][线程编号:19 消息内容：BroadcastMessage{id=1576390301}]
2019-12-15 14:11:41.261  INFO 80487 --- [enerContainer-1] c.i.s.l.a.consumer.BroadcastConsumer     : [onMessage][线程编号:19 消息内容：BroadcastMessage{id=1576390301}]
2019-12-15 14:11:41.263  INFO 80487 --- [enerContainer-1] c.i.s.l.a.consumer.BroadcastConsumer     : [onMessage][线程编号:19 消息内容：BroadcastMessage{id=1576390301}]

// ### mock 方法对应的控制台 ####

# BroadcastConsumer 也消费了 3 条消
2019-12-15 14:11:41.275  INFO 80478 --- [enerContainer-1] c.i.s.l.a.consumer.BroadcastConsumer     : [onMessage][线程编号:19 消息内容：BroadcastMessage{id=1576390301}]
2019-12-15 14:11:41.278  INFO 80478 --- [enerContainer-1] c.i.s.l.a.consumer.BroadcastConsumer     : [onMessage][线程编号:19 消息内容：BroadcastMessage{id=1576390301}]
2019-12-15 14:11:41.279  INFO 80478 --- [enerContainer-1] c.i.s.l.a.consumer.BroadcastConsumer     : [onMessage][线程编号:19 消息内容：BroadcastMessage{id=1576390301}]

显示详细信息
```



- **两个** Consumer 节点，都消费了这条发送的消息。符合广播消费的预期~

# 5. 定时消息

> 示例代码对应仓库：[lab-32-activemq-demo-delay](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-delay)

ActiveMQ 内置了对定时消息的支持，不了解的胖友，可以看看如下文档：

默认情况下，ActiveMQ 默认未开启定时消息的功能，需要我们**手动**去配置开启。通过编辑 `conf/activemq.xml` 配置文件，添加 `schedulerSupport="true"` 来开启。示例如下：



```
<!--
    The <broker> element is used to configure the ActiveMQ broker.
-->
<broker xmlns="http://activemq.apache.org/schema/core" brokerName="localhost" dataDirectory="${activemq.data}" schedulerSupport="true">
```



配置完成，通过 `bin/macosx/activemq restart` 将 ActiveMQ 重启即可生效。

下面，我们来实现一个定时消息的示例。考虑到不污染上述的示例，我们新建一个 [lab-32-activemq-demo-delay](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-delay) 项目。

## 5.1 引入依赖

和 [「3.1 引入依赖」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#) 一致，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-delay/pom.xml) 文件。

## 5.2 应用配置文件

和 [「3.2 应用配置文件」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#) 一致，见 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-delay/src/main/resources/application.yaml) 文件。

## 5.3 Demo02Message

在 [`cn.iocoder.springboot.lab32.activemqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-delay/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/message) 包下，创建 [Demo02Message](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-delay/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/message/Demo02Message.java) 消息类，提供给当前示例使用。

和[「3.4 Demo01Message」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#)一致，只是 Queue 名字不同。

## 5.4 Demo02Producer

在 [`cn.iocoder.springboot.lab32.activemqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-delay/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/producer) 包下，创建 [Demo02Producer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-delay/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/producer/Demo02Producer.java) 类，它会使用 Spring-JMS 封装提供的 JmsMessagingTemplate ，实现发送**定时**消息。代码如下：



```
// Demo02Producer.java

@Component
public class Demo02Producer {

    @Autowired
    private JmsMessagingTemplate jmsTemplate;

    public void syncSend(Integer id, Integer delay) {
        // 创建 Demo02Message 消息
        Demo02Message message = new Demo02Message();
        message.setId(id);
        // 创建 Header
        Map<String, Object> headers = null;
        if (delay != null && delay > 0) {
            headers = new HashMap<>();
            headers.put(ScheduledMessage.AMQ_SCHEDULED_DELAY, delay);
        }
        // 同步发送消息
        jmsTemplate.convertAndSend(Demo02Message.QUEUE, message, headers);
    }

}

显示详细信息
```



- 调用 `#syncSend(Integer id, Integer delay)` 方法来发送消息时，如果传递了方法参数 `delay` ，则我们会设置到消息的 Header 的 `AMQ_SCHEDULED_DELAY` 中，实现延迟 `delay` 毫秒的定时消息。

## 5.5 Demo02Consumer

和[「3.6 Demo01Consumer」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#)基本一致，差别在于消费的队列是 `"QUEUE_DEMO_02"` 。

## 5.6 简单测试

创建 [Demo02ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-delay/src/test/java/cn/iocoder/springboot/lab32/activemqdemo/producer/Demo02ProducerTest.java) 测试类，编写单元测试方法，测试**定时消息**的效果。代码如下：



```
// Demo02ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo02ProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private Demo02Producer producer;

    @Test
    public void testSyncSend01() throws InterruptedException {
        // 不设置消息的过期时间
        this.testSyncSendDelay(null);
    }

    @Test
    public void testSyncSend02() throws InterruptedException {
        // 设置发送消息的过期时间为 5000 毫秒
        this.testSyncSendDelay(5000);
    }

    private void testSyncSendDelay(Integer delay) throws InterruptedException {
        int id = (int) (System.currentTimeMillis() / 1000);
        producer.syncSend(id, delay);
        logger.info("[testSyncSendDelay][发送编号：[{}] 发送成功]", id);

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}

显示详细信息
```



- `#testSyncSend01()` 方法，不设置消息的过期时间。
- `#testSyncSend02()` 方法，发送消息的**过期时间为 5000 毫秒**。

我们先来执行 `#testSyncSend01()` 方法，不设置消息的过期时间。控制台输出如下：



```
# Producer 同步发送消息成功。
2019-12-15 17:40:07.597  INFO 84240 --- [           main] c.i.s.l.a.producer.Demo02ProducerTest    : [testSyncSendDelay][发送编号：[1576402807] 发送成功]

# Consumer 立即消费到该消息
2019-12-15 17:40:07.599  INFO 84240 --- [enerContainer-1] c.i.s.l.a.consumer.Demo02Consumer        : [onMessage][线程编号:18 消息内容：Demo01Message{id=1576402807}]
```



- 符合预期，消息被 Consumer 立即消费。

我们再来执行 `#testSyncSend02()` 方法，发送消息的**过期时间为 5000 毫秒**。控制台输出如下：



```
# Producer 同步发送消息成功。
2019-12-15 17:42:34.560  INFO 84344 --- [           main] c.i.s.l.a.producer.Demo02ProducerTest    : [testSyncSendDelay][发送编号：[1576402954] 发送成功]

# Consumer 5 秒后，消费到该消息
2019-12-15 17:42:40.010  INFO 84344 --- [enerContainer-1] c.i.s.l.a.consumer.Demo02Consumer        : [onMessage][线程编号:18 消息内容：Demo02Message{id=1576402954}]
```



- 符合预期，消息 5 秒后被 Consumer 立即消费。

# 6. 并发消费

> 示例代码对应仓库：[lab-32-activemq-demo-concurrency](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-concurrency)

在上述的示例中，我们配置的每一个 Spring-JMS `@JmsListener` ，都是**串行**消费的。显然，这在监听的 Queue 每秒消息量比较大的时候，会导致消费不及时，导致消息积压的问题。

虽然说，我们可以通过启动多个 JVM 进程，实现**多进程的并发消费**，从而加速消费的速度。但是问题是，否能够实现**多线程**的并发消费呢？答案是**有**。

在 `@JmsListener` 注解中，有 `concurrency` 属性，它可以指定并发消费的线程数。例如说，如果设置 `concurrency=4` 时，Spring-AMQP 就会为**该** `@JmsListener` 创建**至多** 4 个线程，进行并发消费。

考虑到让胖友能够更好的理解 `concurrency` 属性，我们来简单说说 Spring-JMS 在这块的实现方式。我们来举个例子：

- 首先，我们来创建一个 Queue 为 `"DEMO_03"` 。
- 然后，我们创建一个 Demo03Consumer 类，并在其消费方法上，添加 `@JmsListener(concurrency=2)` 注解。
- 再然后，我们启动项目。Spring-AMQP 会根据 `@JmsListener(concurrency=2)` 注解，创建 **2** 个 ActiveMQ Consumer 。注意噢，是 **2** 个 ActiveMQ Consumer 呢！！！后续，每个 ActiveMQ Consumer 会被**单独**分配到一个线程中，进行拉取消息，消费消息。

酱紫讲解一下，胖友对 Spring-JMS 实现**多线程**的并发消费的机制，是否理解了。

下面，我们开始本小节的示例。本示例就是上述举例的具体实现。考虑到不污染上述的示例，我们新建一个 [lab-32-activemq-demo-concurrency](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-concurrency) 项目。

## 6.1 引入依赖

和 [「3.1 引入依赖」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#) 一致，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-concurrency/pom.xml) 文件。

## 6.2 应用配置文件

和 [「3.2 应用配置文件」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#) 一致，见 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-concurrency/src/main/resources/application.yaml) 文件。

## 6.3 Demo03Message

在 [`cn.iocoder.springboot.lab32.activemqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-concurrency/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/message) 包下，创建 [Demo03Message](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-concurrency/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/message/Demo03Message.java) 消息类，提供给当前示例使用。

和[「3.4 Demo01Message」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#)一致，只是 Queue 名字不同。

## 6.4 Demo03Producer

在 [`cn.iocoder.springboot.lab32.activemqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-concurrency/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/producer/) 包下，创建 [Demo03Producer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-concurrency/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/producer/Demo03Producer.java) 类，它会使用 Spring-JMS 封装提供的 JmsMessagingTemplate ，实现发送消息。

和[「3.5 Demo01Producer」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#)一致，只是 Queue 名字不同。

## 6.5 Demo03Consumer

在 [`cn.iocoder.springboot.lab32.activemqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-concurrency/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/consumer/) 包下，创建 [Demo03Consumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-concurrency/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/consumer/Demo03Consumer.java) 类，**并发**消费消息。代码如下：



```
// Demo03Consumer.java

@Component
public class Demo03Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @JmsListener(destination = Demo03Message.QUEUE,
        concurrency = "2")
    public void onMessage(Demo03Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }

}

显示详细信息
```



- 和[「3.6 Demo06Consumer」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#)一致，只差在消费不同的队列。
- 【重要】另外，可以通过 `@JmsListener` 注解的 `concurrency` 属性，设置并发数为 **2**。

## 6.6 简单测试

创建 [Demo03ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-concurrency/src/test/java/cn/iocoder/springboot/lab32/activemqdemo/producer/Demo03ProducerTest.java) 测试类，编写一个单元测试方法，发送 10 条消息，观察并发消费情况。代码如下：



```
// Demo03ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo03ProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private Demo03Producer producer;

    @Test
    public void testSyncSend() throws InterruptedException {
        for (int i = 0; i < 10; i++) {
            int id = (int) (System.currentTimeMillis() / 1000);
            producer.syncSend(id);
//            logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);
        }

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}

显示详细信息
```



执行单元测试方法，控制台输出如下：



```
# 线程编号为 18
2019-12-15 18:17:13.796  INFO 85887 --- [enerContainer-1] c.i.s.l.a.consumer.Demo03Consumer        : [onMessage][线程编号:18 消息内容：Demo03Message{id=1576405033}]
2019-12-15 18:17:13.800  INFO 85887 --- [enerContainer-1] c.i.s.l.a.consumer.Demo03Consumer        : [onMessage][线程编号:18 消息内容：Demo03Message{id=1576405033}]
2019-12-15 18:17:13.802  INFO 85887 --- [enerContainer-1] c.i.s.l.a.consumer.Demo03Consumer        : [onMessage][线程编号:18 消息内容：Demo03Message{id=1576405033}]
2019-12-15 18:17:13.805  INFO 85887 --- [enerContainer-1] c.i.s.l.a.consumer.Demo03Consumer        : [onMessage][线程编号:18 消息内容：Demo03Message{id=1576405033}]
2019-12-15 18:17:13.808  INFO 85887 --- [enerContainer-2] c.i.s.l.a.consumer.Demo03Consumer        : [onMessage][线程编号:21 消息内容：Demo03Message{id=1576405033}]

# 线程编号 18
2019-12-15 18:17:13.796  INFO 85887 --- [enerContainer-2] c.i.s.l.a.consumer.Demo03Consumer        : [onMessage][线程编号:21 消息内容：Demo03Message{id=1576405033}]
2019-12-15 18:17:13.800  INFO 85887 --- [enerContainer-2] c.i.s.l.a.consumer.Demo03Consumer        : [onMessage][线程编号:21 消息内容：Demo03Message{id=1576405033}]
2019-12-15 18:17:13.804  INFO 85887 --- [enerContainer-2] c.i.s.l.a.consumer.Demo03Consumer        : [onMessage][线程编号:21 消息内容：Demo03Message{id=1576405033}]
2019-12-15 18:17:13.807  INFO 85887 --- [enerContainer-1] c.i.s.l.a.consumer.Demo03Consumer        : [onMessage][线程编号:18 消息内容：Demo03Message{id=1576405033}]
2019-12-15 18:17:13.811  INFO 85887 --- [enerContainer-1] c.i.s.l.a.consumer.Demo03Consumer        : [onMessage][线程编号:18 消息内容：Demo03Message{id=1576405033}]

显示详细信息
```



- 我们可以看到，两个线程在消费 `"QUEUE_DEMO_09"` 下的消息。

此时，如果我们使用 [ActiveMQ Web Console](https://activemq.apache.org/web-console) 来查看 `"QUEUE_DEMO_03"` 的消费者列表：![ActiveMQ-消费者列表](https://github.com/Gqyanxin/Gqyanxin.github.io/blob/main/assets/images/activemq/ActiveMQ-消费者列表.png)

# 7. 顺序消息

> 示例代码对应仓库：[lab-32-activemq-demo-orderly](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-orderly)

我们先来一起了解下顺序消息的**顺序消息**的定义：

- 普通顺序消息 ：Producer 将相关联的消息发送到相同的消息队列。
- 完全严格顺序 ：在【普通顺序消息】的基础上，Consumer 严格顺序消费。

那么，让我们来思考下，如果我们希望在 ActiveMQ 上，实现顺序消息需要做两个事情。

① **事情一**，我们需要保证 ActiveMQ Producer 发送相关联的消息发送到相同的 Queue 中。例如说，我们要发送用户信息发生变更的 Message ，那么如果我们希望使用顺序消息的情况下，可以将**用户编号**相同的消息发送到相同的 Queue 中。

② **事情二**，我们在有**且仅启动一个** Consumer 消费该队列，保证 Consumer 严格顺序消费。

不过如果这样做，会存在两个问题，我们逐个来看看。

① **问题一**，正如我们在[「6. 并发消费」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#)中提到，如果我们将消息仅仅投递到一个 Queue 中，并且采用单个 Consumer **串行**消费，在监听的 Queue 每秒消息量比较大的时候，会导致消费不及时，导致消息积压的问题。

此时，我们有两种方案来解决：

- 方案一，在 Producer 端，将 Queue 拆成多个**子** Queue 。假设原先 Queue 是 `QUEUE_USER` ，那么我们就分拆成 `QUEUE_USER_00` 至 `QUEUE_USER_..${N-1}` 这样 N 个队列，然后基于消息的用户编号取余，路由到对应的**子** Queue 中。
- 方案二，在 Consumer 端，将 Queue 拉取到的消息，将相关联的消息发送到**相同的线程**中来消费。例如说，还是 Queue 是 `QUEUE_USER` 的例子，我们创建 N 个线程池大小为 1 的 [ExecutorService](https://github.com/JetBrains/jdk8u_jdk/blob/master/src/share/classes/java/util/concurrent/ExecutorService.java) 数组，然后基于消息的用户编号取余，提交到对应的 ExecutorService 中的单个线程来执行。

两个方案，并不冲突，可以结合使用。

② **问题二**，如果我们启动相同 Consumer 的**多个进程**，会导致相同 Queue 的消息被分配到多个 Consumer 进行消费，破坏 Consumer 严格顺序消费。

此时，我们有两种方案来解决：

- 方案一，引入 Zookeeper 来协调，动态设置多个进程中的**相同的** Consumer 的开关，保证有且仅有一个 Consumer 开启对**同一个** Queue 的消费。
- 方案二，仅适用于【问题一】的【方案一】。还是引入 Zookeeper 来协调，动态设置多个进程中的**相同的** Consumer 消费的 Queue 的分配，保证有且仅有一个 Consumer 开启对**同一个** Queue 的消费。

下面，我们开始本小节的示例。本示例就是上述举例的具体实现。考虑到不污染上述的示例，我们新建一个 [lab-32-activemq-demo-orderly](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-orderly) 项目。

- 对于问题一，我们采用方案一。因为在 Spring-JMS 中，自己定义线程来消费消息，无法和现有的 [DefaultMessageListenerContainer](https://github.com/spring-projects/spring-framework/blob/master/spring-jms/src/main/java/org/springframework/jms/listener/DefaultMessageListenerContainer.java) 的实现所结合，除非自定义一个 MessageListenerContainer 实现类。
- 对于问题二，因为实现起来比较复杂，暂时先不提供。

## 7.1 引入依赖

和 [「3.1 引入依赖」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#) 一致，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-orderly/pom.xml) 文件。

## 7.2 应用配置文件

和 [「3.2 应用配置文件」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#) 一致，见 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-orderly/src/main/resources/application.yaml) 文件。

## 7.3 Demo04Message

在 [`cn.iocoder.springboot.lab32.activemqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-message-model/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/message/) 包下，创建 [Demo04Message](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-orderly/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/message/Demo04Message.java) 消息类，提供给当前示例使用。代码如下：



```
// Demo04Message.java

public class Demo04Message implements Serializable {

    public static final String QUEUE_BASE = "QUEUE_DEMO_04-";
    public static final String QUEUE_0 = QUEUE_BASE + "0";
    public static final String QUEUE_1 = QUEUE_BASE + "1";
    public static final String QUEUE_2 = QUEUE_BASE + "2";
    public static final String QUEUE_3 = QUEUE_BASE + "3";

    public static final int QUEUE_COUNT = 4;

    /**
     * 编号
     */
    private Integer id;

    // ... 省略 set/get/toString 方法

}

显示详细信息
```



- 定义了 `QUEUE_DEMO_04-` 的四个**子** Queue 。

## 7.4 Demo04Producer

在 [`cn.iocoder.springboot.lab32.activemqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-orderly/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/producer) 包下，创建 [Demo04Producer](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-orderly/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/producer/Demo04Producer.java) 类，它会使用 Spring-JMS 封装提供的 JmsMessagingTemplate ，实现发送消息到**子** Queue 中。代码如下：



```
// Demo04Producer.java

@Component
public class Demo04Producer {

    @Autowired
    private JmsMessagingTemplate jmsTemplate;

    public void syncSend(Integer id) {
        // 创建 Demo04Message 消息
        Demo04Message message = new Demo04Message();
        message.setId(id);
        // 同步发送消息
        jmsTemplate.convertAndSend(this.getQueue(id), message);
    }

    private String getQueue(Integer id) {
        return Demo04Message.QUEUE_BASE + (id % Demo04Message.QUEUE_COUNT);
    }

}

显示详细信息
```



- 发送消息时，我们对 `Demo04Message.id % 队列编号` 进行取余，获得**队列编号**作为 Queue 后缀，从而获得到对应的**子** Queue 中。

## 7.5 Demo04Consumer

在 [`cn.iocoder.springboot.lab32.activemqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-orderly/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/consumer) 包下，创建 [Demo04Consumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-orderly/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/consumer/Demo04Consumer.java) 类，**严格**消费**顺序**消息。代码如下：



```
// Demo04Consumer.java

@Component
public class Demo04Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @JmsListener(destination = Demo04Message.QUEUE_0)
    @JmsListener(destination = Demo04Message.QUEUE_1)
    @JmsListener(destination = Demo04Message.QUEUE_2)
    @JmsListener(destination = Demo04Message.QUEUE_3)
    public void onMessage(Demo04Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }

}

显示详细信息
```



- 为了实现每个**子** Queue 能够被每个 Consumer **串行**消费，从而实现基于**子** Queue 的**并行**的**严格**消费**顺序**消息，我们只好在类上添了四个 `@JmsListener` 注解，每个对应一个**子** Queue 。
- 如果胖友使用一个 `@JmsListener` 注解，并添加四个**子** Queue ，然后设置 `concurrency = 4` 时，实际是并发四个线程，消费四个**子** Queue 的消息，无法保证**严格**消费**顺序**消息。

## 7.6 简单测试

创建 [Demo04ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-orderly/src/test/java/cn/iocoder/springboot/lab32/activemqdemo/producer/Demo04ProducerTest.java) 测试类，编写一个单元测试方法，发送 8 条消息，观察顺序消费情况。代码如下：



```
// Demo04ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo04ProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private Demo04Producer producer;

    @Test
    public void testSyncSend() throws InterruptedException {
        for (int i = 0; i < 2; i++) {
            for (int id = 0; id < 4; id++) {
                producer.syncSend(id);
//                logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);
            }
        }

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}

显示详细信息
```



- 发送 2 轮消息，每一轮消息的编号都是 0 至 3 。

执行单元测试方法，控制台输出如下：



```
# 线程编号为 21
2019-12-15 21:44:05.582  INFO 90457 --- [enerContainer-1] c.i.s.l.a.consumer.Demo04Consumer        : [onMessage][线程编号:21 消息内容：Demo04Message{id=0}]
2019-12-15 21:44:05.599  INFO 90457 --- [enerContainer-1] c.i.s.l.a.consumer.Demo04Consumer        : [onMessage][线程编号:21 消息内容：Demo04Message{id=0}]

# 线程编号为 18
2019-12-15 21:44:05.591  INFO 90457 --- [enerContainer-1] c.i.s.l.a.consumer.Demo04Consumer        : [onMessage][线程编号:18 消息内容：Demo04Message{id=2}]
2019-12-15 21:44:05.605  INFO 90457 --- [enerContainer-1] c.i.s.l.a.consumer.Demo04Consumer        : [onMessage][线程编号:18 消息内容：Demo04Message{id=2}]

# 线程编号为 20
2019-12-15 21:44:05.597  INFO 90457 --- [enerContainer-1] c.i.s.l.a.consumer.Demo04Consumer        : [onMessage][线程编号:20 消息内容：Demo04Message{id=3}
2019-12-15 21:44:05.606  INFO 90457 --- [enerContainer-1] c.i.s.l.a.consumer.Demo04Consumer        : [onMessage][线程编号:20 消息内容：Demo04Message{id=3}]

# 线程编号为 19
2019-12-15 21:44:05.583  INFO 90457 --- [enerContainer-1] c.i.s.l.a.consumer.Demo04Consumer        : [onMessage][线程编号:19 消息内容：Demo04Message{id=1}]
2019-12-15 21:44:05.602  INFO 90457 --- [enerContainer-1] c.i.s.l.a.consumer.Demo04Consumer        : [onMessage][线程编号:19 消息内容：Demo04Message{id=1}]

显示详细信息
```



- 相同编号的消息，被投递到相同的**子** Queue ，被相同的线程所消费。符合预期~

# 8. 消费重试

> 示例代码对应仓库：[lab-32-activemq-demo-consume-retry](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-consume-retry)

在消息**消费失败**的时候，ActiveMQ 会通过**自带的** [ReDelivery(重新投递)](https://activemq.apache.org/message-redelivery-and-dlq-handling) 机制，重新投递该消息给 Consumer ，让 Consumer 有机会重新消费消息，实现消费成功。

当然，ActiveMQ 并不会无限重新投递消息给 Consumer 重新消费，而是在默认情况下，达到 N 次重试次数时，Consumer 还是消费失败时，该消息就会进入到**死信队列**(默认为 `"ActiveMQ.DLQ"` 队列)。后续，我们可以通过对死信队列中的消息进行重发，来使得消费者实例再次进行消费。

另外，每条消息的失败重试，是可以配置一定的**间隔时间**。具体，我们在示例的代码中，来进行具体的解释。

下面，我们来实现一个 Consumer 消费重试的示例。考虑到不污染上述的示例，我们新建一个 [lab-32-activemq-demo-consume-retry](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-consume-retry) 项目。

## 8.1 引入依赖

和 [「3.1 引入依赖」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#) 一致，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-consume-retry/pom.xml) 文件。

## 8.2 应用配置文件

和 [「3.1.2 应用配置文件」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#) 一致，见 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-consume-retry/src/main/resources/application.yaml) 文件。

## 8.3 ActiveMQConfig

在 [`cn.iocoder.springboot.lab32.activemqdemo.config`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-consume-retry/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/config) 包下，创建 [ActiveMQConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-consume-retry/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/config/ActiveMQConfig.java) 类，实现 [ActiveMQConnectionFactoryCustomizer](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jms/activemq/ActiveMQConnectionFactoryCustomizer.java) 接口，可以对 Spring Boot 自动创建的 [ActiveMQConnectionFactory](https://github.com/apache/activemq/blob/master/activemq-client/src/main/java/org/apache/activemq/ActiveMQConnectionFactory.java) 进行**自定义配置**。代码如下：



```
// ActiveMQConfig.java

@Configuration
public class ActiveMQConfig implements ActiveMQConnectionFactoryCustomizer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    public void customize(ActiveMQConnectionFactory factory) {
        logger.info("[customize][默认重试策略: {}]", factory.getRedeliveryPolicy());
    }

}

显示详细信息
```



- 默认情况下，已经开启 ActiveMQ ReDelivery 机制。这里，我们先打印下默认的 [RedeliveryPolicy](https://github.com/apache/activemq/blob/master/activemq-client/src/main/java/org/apache/activemq/RedeliveryPolicy.java) 重投策略。如下：



  ```
  2019-12-15 23:01:41.406  INFO 93850 --- [           main] QConfig$$EnhancerBySpringCGLIB$$13367ef1 : [customize][默认重试策略: RedeliveryPolicy {destination = null, collisionAvoidanceFactor = 0.15, maximumRedeliveries = 6, maximumRedeliveryDelay = -1, initialRedeliveryDelay = 1000, useCollisionAvoidance = false, useExponentialBackOff = false, backOffMultiplier = 5.0, redeliveryDelay = 1000, preDispatchCheck = true}]
  ```



- 默认重投 6 次，每次间隔 1000 毫秒。

- 如果胖友想要创建 RedeliveryPolicy 对象，自定义重投策略。更多可以参考 [《ActiveMQ 文档 —— Redelivery Policy》](https://activemq.apache.org/redelivery-policy) 。

## 8.4 Demo05Message

在 [`cn.iocoder.springboot.lab32.activemqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-consume-retry/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/message) 包下，创建 [Demo05Message](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-consume-retry/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/message/Demo05Message.java) 消息类，提供给当前示例使用。

和[「3.4 Demo01Message」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#)一致，只是 Queue 名字不同。

## 8.5 Demo05Producer

在 [`cn.iocoder.springboot.lab32.activemqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-consume-retry/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/producer) 包下，创建 [Demo05Producer](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-consume-retry/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/producer/Demo05Producer.java) 类，它会使用 Spring-JMS 封装提供的 JmsMessagingTemplate ，实现发送消息。

和[「3.5 Demo01Producer」](read://https_www.iocoder.cn/?url=https%3A%2F%2Fwww.iocoder.cn%2FSpring-Boot%2FActiveMQ%2F%3Fgithub#)一致，只是 Queue 名字不同。

## 8.6 Demo05Consumer

在 [`cn.iocoder.springboot.lab32.activemqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-consume-retry/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/consumer) 包下，创建 [Demo05Consumer](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-32/lab-32-activemq-demo-consume-retry/src/main/java/cn/iocoder/springboot/lab32/activemqdemo/consumer/Demo05Consumer.java) 类，消费消息。代码如下：



```
// Demo05Consumer.java

@Component
public class Demo05Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @JmsListener(destination = Demo05Message.QUEUE)
    public void onMessage(Demo05Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
        // <X> 注意，此处抛出一个 RuntimeException 异常，模拟消费失败
        throw new RuntimeException("我就是故意抛出一个异常");
    }

}

显示详细信息
```



- 在 `<X>` 处，我们在消费消息时候，抛出一个 RuntimeException 异常，模拟消费失败。

## 8.7 简单测试

创建 [Demo05ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-32/lab-32-activemq-demo-consume-retry/src/test/java/cn/iocoder/springboot/lab32/activemqdemo/producer/Demo05ProducerTest.java) 测试类，编写单元测试方法，测试**消费重试**的效果。代码如下：



```
// Demo05ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo05ProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private Demo05Producer producer;

    @Test
    public void testSyncSend() throws InterruptedException {
        // 发送消息
        int id = (int) (System.currentTimeMillis() / 1000);
        producer.syncSend(id);
        logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}

显示详细信息
```



我们来执行 `#testSyncSend()` 方法，测试 Consumer 消费重试的效果。控制台输出如下：



```
// Producer 同步发送消息成功。
2019-12-15 23:04:26.865  INFO 94045 --- [           main] c.i.s.l.a.producer.Demo05ProducerTest    : [testSyncSend][发送编号：[1576422266] 发送成功]

// Consumer 第 1 次消费
2019-12-15 23:04:26.868  INFO 94045 --- [enerContainer-1] c.i.s.l.a.consumer.Demo05Consumer        : [onMessage][线程编号:18 消息内容：Demo05Message{id=1576422266}]
2019-12-15 23:04:26.877  WARN 94045 --- [enerContainer-1] o.s.j.l.DefaultMessageListenerContainer  : Execution of JMS message listener failed, and no ErrorHandler has been set.

org.springframework.jms.listener.adapter.ListenerExecutionFailedException: Listener method 'public void cn.iocoder.springboot.lab32.activemqdemo.consumer.Demo05Consumer.onMessage(cn.iocoder.springboot.lab32.activemqdemo.message.Demo05Message)' threw exception; nested exception is java.lang.RuntimeException: 我就是故意抛出一个异常
    // ... 省略异常堆栈

// 1 秒后，Consumer 第 1 次重试消费
2019-12-15 23:04:27.875  INFO 94045 --- [enerContainer-1] c.i.s.l.a.consumer.Demo05Consumer        : [onMessage][线程编号:18 消息内容：Demo05Message{id=1576422266}]
2019-12-15 23:04:27.877  WARN 94045 --- [enerContainer-1] o.s.j.l.DefaultMessageListenerContainer  : Execution of JMS message listener failed, and no ErrorHandler has been set.

org.springframework.jms.listener.adapter.ListenerExecutionFailedException: Listener method 'public void cn.iocoder.springboot.lab32.activemqdemo.consumer.Demo05Consumer.onMessage(cn.iocoder.springboot.lab32.activemqdemo.message.Demo05Message)' threw exception; nested exception is java.lang.RuntimeException: 我就是故意抛出一个异常
    // ... 省略异常堆栈

// 1 秒后，Consumer 第 2 次重试消费
2019-12-15 23:04:28.878  INFO 94045 --- [enerContainer-1] c.i.s.l.a.consumer.Demo05Consumer        : [onMessage][线程编号:18 消息内容：Demo05Message{id=1576422266}]
2019-12-15 23:04:28.879  WARN 94045 --- [enerContainer-1] o.s.j.l.DefaultMessageListenerContainer  : Execution of JMS message listener failed, and no ErrorHandler has been set.

org.springframework.jms.listener.adapter.ListenerExecutionFailedException: Listener method 'public void cn.iocoder.springboot.lab32.activemqdemo.consumer.Demo05Consumer.onMessage(cn.iocoder.springboot.lab32.activemqdemo.message.Demo05Message)' threw exception; nested exception is java.lang.RuntimeException: 我就是故意抛出一个异常
    // ... 省略异常堆栈

// 1 秒后，Consumer 第 3 次重试消费
2019-12-15 23:04:29.884  INFO 94045 --- [enerContainer-1] c.i.s.l.a.consumer.Demo05Consumer        : [onMessage][线程编号:18 消息内容：Demo05Message{id=1576422266}]
2019-12-15 23:04:29.887  WARN 94045 --- [enerContainer-1] o.s.j.l.DefaultMessageListenerContainer  : Execution of JMS message listener failed, and no ErrorHandler has been set.

org.springframework.jms.listener.adapter.ListenerExecutionFailedException: Listener method 'public void cn.iocoder.springboot.lab32.activemqdemo.consumer.Demo05Consumer.onMessage(cn.iocoder.springboot.lab32.activemqdemo.message.Demo05Message)' threw exception; nested exception is java.lang.RuntimeException: 我就是故意抛出一个异常
    // ... 省略异常堆栈

// 1 秒后，Consumer 第 4 次重试消费
2019-12-15 23:04:30.887  INFO 94045 --- [enerContainer-1] c.i.s.l.a.consumer.Demo05Consumer        : [onMessage][线程编号:18 消息内容：Demo05Message{id=1576422266}]
2019-12-15 23:04:30.890  WARN 94045 --- [enerContainer-1] o.s.j.l.DefaultMessageListenerContainer  : Execution of JMS message listener failed, and no ErrorHandler has been set.

org.springframework.jms.listener.adapter.ListenerExecutionFailedException: Listener method 'public void cn.iocoder.springboot.lab32.activemqdemo.consumer.Demo05Consumer.onMessage(cn.iocoder.springboot.lab32.activemqdemo.message.Demo05Message)' threw exception; nested exception is java.lang.RuntimeException: 我就是故意抛出一个异常
    // ... 省略异常堆栈

// 1 秒后，Consumer 第 5 次重试消费
2019-12-15 23:04:31.893  INFO 94045 --- [enerContainer-1] c.i.s.l.a.consumer.Demo05Consumer        : [onMessage][线程编号:18 消息内容：Demo05Message{id=1576422266}]
2019-12-15 23:04:31.895  WARN 94045 --- [enerContainer-1] o.s.j.l.DefaultMessageListenerContainer  : Execution of JMS message listener failed, and no ErrorHandler has been set.

org.springframework.jms.listener.adapter.ListenerExecutionFailedException: Listener method 'public void cn.iocoder.springboot.lab32.activemqdemo.consumer.Demo05Consumer.onMessage(cn.iocoder.springboot.lab32.activemqdemo.message.Demo05Message)' threw exception; nested exception is java.lang.RuntimeException: 我就是故意抛出一个异常
    // ... 省略异常堆栈

// 1 秒后，Consumer 第 6 次重试消费
2019-12-15 23:04:32.897  INFO 94045 --- [enerContainer-1] c.i.s.l.a.consumer.Demo05Consumer        : [onMessage][线程编号:18 消息内容：Demo05Message{id=1576422266}]
2019-12-15 23:04:32.901  WARN 94045 --- [enerContainer-1] o.s.j.l.DefaultMessageListenerContainer  : Execution of JMS message listener failed, and no ErrorHandler has been set.

org.springframework.jms.listener.adapter.ListenerExecutionFailedException: Listener method 'public void cn.iocoder.springboot.lab32.activemqdemo.consumer.Demo05Consumer.onMessage(cn.iocoder.springboot.lab32.activemqdemo.message.Demo05Message)' threw exception; nested exception is java.lang.RuntimeException: 我就是故意抛出一个异常
    // ... 省略异常堆栈

显示详细信息
```



- Consumer 重试消费消息 6 次，每次间隔 1 秒，全部都失败，最终该消息转发到死信队列中。

此时，如果我们使用 [ActiveMQ Web Console](https://activemq.apache.org/web-console) 来查看 `"ActiveMQ.DLQ"` 的队列的消息：![ActiveMQ-队列消息](https://github.com/Gqyanxin/Gqyanxin.github.io/blob/main/assets/images/activemq/ActiveMQ-队列消息.png)

- `"ActiveMQ.DLQ "` 队列中有 1 条消息，就是我们刚消费失败到达上限的该消息。

更多 ActiveMQ ReDelivery 的内容，可额外阅读如下文章：

# 9. 事务消息

推荐阅读文章：

# 10. 消费者的消息确认

推荐阅读文章：

# 11. 生产者的发送确认

推荐阅读文章：

- [《ActiveMQ 消息的发送原理》](https://www.cnblogs.com/wuzhenzhao/p/10084058.html)

# 12. RPC 远程调用

推荐阅读文章：

# 13. MessageConverter

使用 JSON 作为消息的序列化方式。推荐阅读文章：

# 14. 消费异常处理器

推荐阅读文章：

# 666. 彩蛋

最后几个小节的内容，偷懒了一下，找了一些文章，进行了下推荐。主要是，艿艿并没有打算特别深入的学习 ActiveMQ 的内容，所以也就只写了自己比较感兴趣的内容。当然，未来如果工作上有需要，艿艿还是会补充完善下的。

因为艿艿个人在生产环境下，主要是使用 RocketMQ 作为消息队列。如果有写的不正确的地方，辛苦胖友帮忙指正。这里额外在推荐一些 Activemq 不错的内容：

最后弱弱的说一下，还是 RocketMQ 更加好用，哈哈哈哈~

最后的最后，艿艿用一张图概括下，目前基于 [Spring-Messaging](https://github.com/spring-projects/spring-framework/tree/master/spring-messaging/src/main/java/org/springframework/messaging) 体系，访问常用消息中间件的图：![Spring-Messaging 生态](https://github.com/Gqyanxin/Gqyanxin.github.io/blob/main/assets/images/activemq/ActiveMQ-Spring-Messaging生态.png)