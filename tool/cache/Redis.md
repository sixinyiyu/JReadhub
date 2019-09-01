
### Redis支持是数据结构

```redis```支持String、List、Set、Hashs、Sorted Set等数据结构


### Redis如何保证高可用性


### Redis的数据过期策略


### Redis缓存一致性处理

先更新库、在失效缓存

失效：应用程序先从cache取数据，没有得到，则从数据库中取数据，成功后，放到缓存中。

命中：应用程序从cache中取数据，取到后返回。

更新：先把数据存到数据库中，成功后，再让缓存失效。

如果删除缓存失败、则投递到mq，进行重试

或者删除缓存、更新库、在延迟一点时间删除缓存

监听mysql日志binlog 利用日志更新缓存

### 单线程的Redis是如何实现高并发的


### Redis与memcached有什么区别

### 如何处理Redis热点key

### 如何处理Redis大value的key获取