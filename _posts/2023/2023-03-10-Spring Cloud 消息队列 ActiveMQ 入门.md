---
layout: post
title: Spring Cloud 消息队列 ActiveMQ 入门
category: CSS
tags: [css]
---

## Spring Cloud 消息队列 ActiveMQ 入门


摘要: 原创出处 http://www.iocoder.cn/Spring-Cloud/ActiveMQ/

------

------

> 本文在提供完整代码示例，可见 https://github.com/YunaiV/SpringBoot-Labs
>
> 原创不易，给点个 [Star](https://github.com/YunaiV/SpringBoot-Labs/stargazers) 嘿，一起冲鸭！

在 Spring Cloud 中，我们使用对应消息队列的 [Spring Cloud Stream](https://github.com/spring-cloud/spring-cloud-stream) 实现库，接入其作为消息中间件，实现消息驱动的微服务。

不过 ActiveMQ 对应的实现库 [`spring-cloud-stream-binder-jms`](https://github.com/spring-cloud/spring-cloud-stream-binder-jms) 处于停止更新的状态，最后一次提交也是 3 年之前，如下图所示：![img](https://github.com/Gqyanxin/Gqyanxin.github.io/blob/main/assets/images/activemq/ActiveMQ-SpringCloud实现库.png)

因此，如果我们想要在 Spring Cloud 中使用 ActiveMQ 的话，暂时只能使用 Spring-JMS 库，具体可以参考[《Spring Boot 消息队列 ActiveMQ 入门》](https://gqyanxin.github.io/css/2023/03/10/Spring-Boot-%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97-ActiveMQ-%E5%85%A5%E9%97%A8.html)文章。

另外 ActiveMQ 采用的公司越来越少，推荐可以尝试 RocketMQ、RabbitMQ、Kafka 等更优秀的消息队列。

------
