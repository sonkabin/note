# Redis设计与实现2——发布订阅模型

Redis完全就是基于观察者模式思想来实现的

## 订阅原理

channel和pattern都放在redisServer中

### channel

1. 用`pubsub_channel`维护订阅信息，它是一个字典，键是channel，值是订阅channel的客户端（以链表形式串联）
2. 订阅channel（`SUBSCRIBE`）：如果channel存在，则加入到链表末尾；否则创建键，将客户端加入到链表中
3. 退订channel（`UNSUBSCRIBE`）：删除客户端即可；如果链表变为空链表，删除键

### pattern

1. 用`pubsub_patterns`维护信息，它是一个链表。因此找模式的时候需要遍历整个链表
2. 订阅pattern（`PSUBSCRIBE`）：新建`pubsubPattern`，设置客户端和模式后，加入到链表中
3. 退订pattern（`PUNSUBSCRIBE`）：找到，删除

## 实现原理

`PUBLISH`消息后，服务器遍历channel对应键的值和pattern，发送消息给对应的客户端

## 发布订阅模型可以用作消息队列吗？

它解决了消息重复消费的问题

缺点：

- 发布订阅模型没有ACK，无法知道一个消息是否被正常处理