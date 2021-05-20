# RabbitMQ

## 如何保证消息的可靠性传输？或者说，如何处理消息丢失的问题？

https://github.com/shishan100/Java-Interview-Advanced/blob/master/docs/high-concurrency/how-to-ensure-the-reliable-transmission-of-messages.md

### 生产者弄丢了数据

生产者发送给mq时，数据就在半路给搞丢了，因为网络问题啥的

**发送数据之前**开启 RabbitMQ 事务`channel.txSelect`，然后发送消息，如果消息没有成功被 RabbitMQ 接收到，那么生产者会收到异常报错，此时就可以回滚事务`channel.txRollback`，然后重试发送消息；如果收到了消息，那么可以提交事务`channel.txCommit`。

```java
// 开启事务
channel.txSelect
try {
    // 这里发送消息
} catch (Exception e) {
    channel.txRollback

    // 这里再次重发这条消息
}

// 提交事务
channel.txCommit
```

RabbitMQ 事务机制（同步）一搞，基本上**吞吐量会下来，因为太耗性能**。

**`confirm` 机制 （确认机制）mq会回调生产者告诉生产者我收到消息了，保证消息不会丢失。**

**事务机制是同步的**，你提交一个事务之后会**阻塞**在那儿，但是 `confirm` 机制是**异步**的，你发送个消息之后就可以发送下一个消息，然后那个消息 RabbitMQ 接收了之后会异步回调你的一个接口通知你这个消息接收到了。

所以一般在生产者这块**避免数据丢失**，都是用 `confirm` 机制的。

### RabbitMQ 弄丢了数据

几率很小，除非MQ在使用过程中服务器坏了，mq挂掉了，宕机了会出现消息丢失问题。

**开启 RabbitMQ 的持久化**，就是消息写入之后会持久化到磁盘，哪怕是 RabbitMQ 自己挂了，**恢复之后会自动读取之前存储的数据**，一般数据不会丢

设置持久化有**两个步骤**：

- 创建 queue 的时候将其设置为持久化
  这样就可以保证 RabbitMQ 持久化 queue 的元数据，但是它是不会持久化 queue 里的数据的。
- 第二个是发送消息的时候将消息的 `deliveryMode` 设置为 2
  就是将消息设置为持久化的，此时 RabbitMQ 就会将消息持久化到磁盘上去。

**持久化可以跟生产者那边的 `confirm` 机制配合起来，只有消息被持久化到磁盘之后，才会通知生产者 `ack` 了，所以哪怕是在持久化到磁盘之前，RabbitMQ 挂了，数据丢了，生产者收不到 `ack`，你也是可以自己重发的**

#### 

###  消费端弄丢了数据

主要是因为你消费的时候，**刚消费到，还没处理，结果进程挂了**，比如重启了，那么就尴尬了，RabbitMQ 认为你都消费了，这数据就丢了。

关闭mq的自动ack机制，使用手动ack。

![image-20201021232420299](assets/image-20201021232420299.png)



## 如何保证消息的顺序性？

https://github.com/shishan100/Java-Interview-Advanced/blob/master/docs/high-concurrency/how-to-ensure-the-order-of-messages.md

举个例子，我们以前做过一个 mysql `binlog` 同步的系统，压力还是非常大的，日同步数据要达到上亿，就是说数据从一个 mysql 库原封不动地同步到另一个 mysql 库里面去（mysql -> mysql）。常见的一点在于说比如大数据 team，就需要同步一个 mysql 库过来，对公司的业务系统的数据做各种复杂的操作。

你在 mysql 里增删改一条数据，对应出来了增删改 3 条 `binlog` 日志，接着这三条 `binlog` 发送到 MQ 里面，再消费出来依次执行，起码得保证人家是按照顺序来的吧？不然本来是：增加、修改、删除；你楞是换了顺序给执行成删除、修改、增加，不全错了么。

本来这个数据同步过来，应该最后这个数据被删除了；结果你搞错了这个顺序，最后这个数据保留下来了，数据同步就出错了。

RabbitMq出现顺序错乱场景

一个 queue，多个 consumer。比如，生产者向 RabbitMQ 里发送了三条数据，顺序依次是 data1/data2/data3，压入的是 RabbitMQ 的一个内存队列。有三个消费者分别从 MQ 中消费这三条数据中的一条，结果消费者2先执行完操作，把 data2 存入数据库，然后是 data1/data3。这不明显乱了



![image-20201021232740472](assets/image-20201021232740472.png)

解决方案：

保证一个队列对应一个消费者，就能保证压入消息队列的顺序就是消费者消费的顺序。

**拆分多个 queue，每个 queue 一个 consumer，就是多一些 queue 而已，确实是麻烦点；或者就一个 queue 但是对应一个 consumer**

![image-20201021232919136](assets/image-20201021232919136.png)



## 如何解决消息队列的延时以及过期失效问题？消息队列满了以后该怎么处理？有几百万消息持续积压几小时，说说怎么解决？

​	https://github.com/shishan100/Java-Interview-Advanced/blob/master/docs/high-concurrency/mq-time-delay-and-expired-failure.md



## 如果让你写一个消息队列，该如何进行架构设计？说一下你的思路。

https://github.com/shishan100/Java-Interview-Advanced/blob/master/docs/high-concurrency/mq-design.md



## MQ高可用

### 单机模式

只部署一个实例，线上不用



### 集群模式

部署 多个rabbitmq节点， A、B、C

A是主节点，那么BC连接A后会同步A的元数据（exchange、queue、vhost等）不会冗余真正的数据信息。当消息生产者连接BC时，BC起到一个转发作用，通过同步的元数据，转发数据存储到A节点中，消费者连接BC时，同样起到一个转发作用，会将数据从A拉取到本地然后发送给消费者。

并不能实现真正的高可用。能够提高处理能力。



### 镜像部署模式

rabbitmq真正高可用模式，每个节点都冗余了数据信息，消费者、生产者连接那个节点都可以，如果一个节点挂了，其他节点还能正常提供服务。





