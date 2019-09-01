
Dubbo允许同服务上支持不同协议或者同一服务上同时支持多种协议，大数据用短连接协议，小数据大并发用长连接协议。

多协议配置示例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd"> 
    <dubbo:application name="world"  />
    <dubbo:registry id="registry" address="10.20.141.150:9090" username="admin" password="hello1234" />
    <!-- 多协议配置 -->
    <dubbo:protocol name="dubbo" port="20880" />
    <dubbo:protocol name="rmi" port="1099" />
    <!-- 使用dubbo协议暴露服务 -->
    <dubbo:service interface="com.alibaba.hello.api.HelloService" version="1.0.0" ref="helloService" protocol="dubbo" />
    <!-- 使用rmi协议暴露服务 -->
    <dubbo:service interface="com.alibaba.hello.api.DemoService" version="1.0.0" ref="demoService" protocol="rmi" /> 
</beans>
```

下面具体列举下Dubbo支持下协议有哪些!

### ```dubbo```协议

```dubbo```协议是默认的缺省协议，底层采用的是异步(NIO)单一长连接，适合数据量小，并发量高的和消费者数远远大于提供者数目的情况。

特性：
* 单连接
* TCP长连接
* NIO异步传输
* Hessian二进制序列化
* 适用于消费者远多于提供者，数据包比较小并发要求较高的情况

### ```rmi```协议

```rmi```协议是JDK 标准的```java.rmi.*```实现，采用阻塞式短连接和 JDK 标准序列化方式。

特性：
* 多连接
* TCP短连接
* 同步传输
* Java二进制序列化
* 适用于消费者、提供者差不多，传输数据包大小混合，可以用来传输文件

### ```hessian```协议

```hessian```协议是基于```Hessian```服务，底层基于Http通信采用```Servlet```暴露服务。

特性：
* 多连接
* Http短连接
* 同步传输
* Hessian二进制序列化
* 适用于传入传出参数数据包较大，提供者比消费者个数多，提供者压力较大，可传文件

### ```http```协议

```http```协议是基于HTTP表单的远程调用协议，采用 Spring 的 HttpInvoker 实现。

特性：
* 多连接
* Http短连接
* 同步传输
* 表单序列化
* 适用于传入传出参数数据包大小混合，提供者比消费者个数多，不支持传输文件

### ```webservice```协议

基于```WebService```的远程调用协议

特性：
* 多连接
* Http短连接
* 同步传输
* SOAP 文本序列化

### ```thrif```协议

### ```redis```协议

### ```rest```协议

基于标准的Java REST API实现的REST调用支持


项目中我们一般用的比较多的就是缺省的```dubbo```协议还有就是偶尔用到```hessian```协议(文件什么的)