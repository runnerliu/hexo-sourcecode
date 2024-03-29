---
title: RocketMQ如何处理消息丢失
date: 2021-05-18 22:28:36
tags:
 - RocketMQ
 - MQ
categories:
 - MQ
---

#### 消息的发送过程

![2021-5-18T223009.png](/images/2021-5-18T223009.png)

- 生产阶段：Producer 新建消息，然后通过网络将消息投递给 MQ Broker

- 存储阶段：消息将会存储在 Broker 端磁盘中

- 消费阶段：Consumer 将会从 Broker 拉取消息

以上任一阶段都可能会丢失消息，我们只要找到这三个阶段丢失消息原因，采用合理的办法避免丢失，就可以彻底解决消息丢失的问题。

#### 生产阶段

生产者（Producer） 通过网络发送消息给 Broker，当 Broker 收到之后，将会返回确认响应信息给 Producer。所以生产者只要接收到返回的确认响应，就代表消息在生产阶段未丢失。

##### 同步发送

有三种Send方法，同步发送、异步发送、单向发送。我们可以采取同步发送的方式进行发送消息，发消息的时候会同步阻塞等待broker返回的结果，如果没成功，则不会收到SendResult，这种是最可靠的。其次是异步发送，在回调方法里可以得知是否发送成功。单向发送是最不靠谱的一种发送方式，我们无法保证消息真正可达。

##### 失败重试

发送消息如果失败或者超时了，则会自动重试。默认是重试三次，可以根据API进行更改，比如改为10次：

```
producer.setRetryTimesWhenSendFailed(10);
```

#### 存储阶段

MQ持久化消息分为两种：同步刷盘和异步刷盘。

##### 同步刷盘

默认情况是异步刷盘，Broker收到消息后会先存到Cache里然后立马通知Producer成功，然后Broker启动异步线程的去持久化到磁盘中，但是Broker还没持久化到磁盘就宕机的话，消息就丢失了。同步刷盘的话是收到消息存到Cache后并不会通知Producer说消息已经OK了，而是会等到持久化到磁盘中后才会通知Producer说消息完事了。这也保障了消息不会丢失，但是性能不如异步高。看业务场景取舍。

##### 主从同步复制

即使Broker设置了同步刷盘策略，但是Broker刷完盘后磁盘坏了，这会导致盘上的消息全TM丢了。但是如果即使是1主1从了，但是Master刷完盘后还没来得及同步给Slave就磁盘坏了，也会导致消息丢失。所以我们还可以配置不仅是等Master刷完盘就通知Producer，而是等Master和Slave都刷完盘后才去通知Producer说消息OK了。

#### 消费阶段

消费者从 broker 拉取消息，然后执行相应的业务逻辑。一旦执行成功，将会返回 `ConsumeConcurrentlyStatus.CONSUME_SUCCESS` 状态给 Broker。如果 Broker 未收到消费确认响应或收到其他状态，消费者下次还会再次拉取到该条消息，进行重试。这样的方式有效避免了消费者消费过程发生异常，或者消息在网络传输中丢失的情况。



Read More:

> [面试官再问我如何保证 RocketMQ 不丢失消息,这回我笑了！](https://www.cnblogs.com/goodAndyxublog/p/12563813.html)
>
> [udp怎么保证不丢包_从入门到入土（三）RocketMQ 怎么保证的消息不丢失？](https://blog.csdn.net/weixin_39601088/article/details/111343167)
>