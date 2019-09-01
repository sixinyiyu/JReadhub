
```Dubbo```支持多种方式的注册中心实现；```Zookeeper注册中心```、```Redis注册中心```、```Multicast 注册中心```、简单注册中心。

### ```Multicast 注册中心```

组播注册中心，不需要启动注册中心，只是广播地址

主要流程是：

1. 服务提供者启动时广播自己的地址
2. 消费者启动时广播订阅请求
3. 提供者收到订阅请求，就将自己的地址广播给消费者
4. 消费者收到提供者广播的地址后，利用地址进行RPC调用

组播注册中心，用的还是非常的少的

### ```Redis注册中心```

利用Reids来做注册中心，数据结构是Key-Map方式，并利用```Redis的 Publish/Subscribe```来做数据变更

数据结构

* 主 Key 为服务名和类型
* Map 中的 Key 为 URL 地址
* Map 中的 Value 为过期时间，用于判断脏数据，脏数据由监控中心删除

主要流程是：

1. 服务提供者启动向拼接成主Key(Key:/dubbo/com.dubbo.demo.UserService/providers)下注册自己的地址并对对
Channel:/dubbo/com.dubbo.demo.UserService/providers发送register事件
2. 消费者启动时从Channel:/dubbo/com.dubbo.demo.UserService/providers注册register 和 unregister事件，同时注册消费者的Key:/dubbo/com.dubbo.demo.UserService/consumers 
3. 消费者收到事件后，根据事件类型获取服务提供者的地址
4. 服务监控中心启动时，从 Channel:/dubbo/* 订阅 register 和 unregister，以及 subscribe 和unsubsribe事件
5. 服务监控中心收到 register 和 unregister 事件后，从 Key:/dubbo/com.dubbo.demo.UserService/providers 下获取提供者地址列表
6. 服务监控中心收到 subscribe 和 unsubsribe 事件后，从 Key:/dubbo/com.dubbo.demo.UserService/consumers 下获取消费者地址列表

Redis注册中心，项目中用到的也很少

### ```Zookeeper注册中心```

利用zk的特性来做注册中心利用zk的节点以及低延时能非常实时发反应提供者、消费者的信息变更情况，是官方强烈推荐的。

主要流程是：
1. 服务提供者启动时: 向 /dubbo/com.dubbo.demo.UserService/providers 目录下写入自己的 URL 地址
2. 服务消费者启动时: 订阅 /dubbo/com.dubbo.demo.UserService/providers 目录下的提供者 URL 地址 并向 /dubbo/com.dubbo.demo.UserService/consumers 目录下写入自己的 URL 地址
3. 监控中心启动时: 订阅 /dubbo/com.dubbo.demo.UserService 目录下的所有提供者和消费者 URL 地址

### ```Simple注册中心```

很少使用一般在开发中用