# 为什么使用MQ

- 通过异步处理提高系统性能（减少响应所需时间）
- 削峰/限流：先将短时间高并发产生的事务消息存储在消息队列中，然后后端服务再慢慢根据自己的能力去消费这些消息，这样就避免直接把后端服务打垮掉。
- 降低系统耦合性：使用消息队列还可以降低系统耦合性。我们知道如果模块之间不存在直接调用，那么新增模块或者修改模块就对其他模块影响较小，这样系统的可扩展性无疑更好一些

## 优点

- 解耦
- 限流削峰
- 异步分发

## 缺点

- 系统可用性降低
- 系统复杂度提高
- 一致性问题

# 保证消息队列的高可用



# 如何保证消息不被重复消费？或者说，如何保证消息消费的幂等性？

产生的原因

- 生产者发给mq由于mq 的ack 丢失，导致发送重复。
- 消费者消费mq数据时，ack丢失，导致mq消息重复发送。

解决方案：

由消费者接口保证幂等性，保证消息只被消费一次

- 数据库唯一主键约束，如果抛出唯一约束错误，则直接丢弃任务不处理
- 乐观锁，使用version字段或updatetime字段保证多次更新不会造成影响
- 状态机：状态的更改是有先后条件的，根据状态字段判断保证
- 每个消息颁发一个token并放入redis，消费过就删除
- 生成一个全局唯一id，贯穿整个系统，下游系统消费过就进行记录，下次消费进行比对
- 

# 保证顺序消费

多个消息发送者，一个Queue，多个消费者，无法保证消费顺序。

**解决方案：**

一个队列中的消息FIFO总是有序的，新建多个队列，一个消费者对应一个队列，实现消息的顺序消费。

# 消息的可靠性传输

RabbitMq

丢失消息的三个场景

生产者发送到mq丢失：启用mq的事务机制（同步，性能差） 或者 confirm机制（异步，性能高），收到消息后会进行ack确认。

mq丢失：mq持久化机制，持久化到磁盘。

消费者丢失：手动ack机制，成功消费才ack。mq收到ack才会删除消息



**mq持久化有可能在还没有持久化就宕机了，概率极小，怎么处理？**

网上没有搜到答案

思考：

设置一个配置，是否使用完全一致性，不能接受mq丢失消息。

在业务生产方提供一个任务对比接口，服务下游定期访问接口对比任务是否能对上，是否全部消费。如果没有消费，通知生产者进行补偿。



# 大量消息积压怎么办

- 增加消费者
- 如果是消费者代码异常无法消费，重写一个业务快速上线，读取消息后直接持久化到数据库，解决堆积问题，等修复bug后，重新发送数据到mq消费



#  延迟队列

Rabbitmq

ttl + 死信队列



# 设计一个MQ的思路



