# 介绍常用的消息队列

面试问点：工作中用了哪些消息队列，除了RabbitMQ你还了解其他消息队列吗？ 它们各有什么优点



| 特性           | ActiveMQ           | RabbitMQ           | RocketMQ          |  Kafka  |
|----------------|:---:|-----------|-------:|------:|
| 客户端SDK      | C++、Java、.Net、Python、 Php、 Ruby | .Net、Java、Ruby、Erlang     | Java, C++, Go  |  Java, Scala    |
| 协议           | 推模式、STOMP, AMQP, MQTT, JMS     | 推/拉模式，AMQP，XMPP, SMTP, STOMP，HTTP|  拉模式、TCP, JMS  |  拉模式、TCP   |
| 可用性         | 高 基于主从架构实现高可用性   | 高 基于主从架构实现高可用性 | 非常高  分布式架构  |  非常高 分布式架构  |
| 时效性         | ms   | 微妙 延迟非常低 |  ms|   ms 以内|
| 单机吞吐量     | 万级比RocketMQ、Kafka低一个数量级 | 万级比RocketMQ、Kafka低一个数量级| 10万级| 10万级 Kafka特点就是吞吐量高|
| 消息持久性     | 支持持久化 |  支持持久化| 支持持久化|支持持久化|
| 总结          | 系统非常成熟，偶尔会存在丢失消息, 较少在大规模场景中使用 |Erlang 开发，比较重所以吞吐量比较小，管理界面完善，很难做定制化或解决bug | 使用简单，吞吐量高，社区活跃 Java开发| 功能较少，吞吐量超高，可扩展性高，会存在有重复的消息|
