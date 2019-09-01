```Mybatis```运行流程、原理

> 流程

1. 首先当然是配置文件的加载、解析了(先是解析mybatic-config.xml 解析其中的environments、datasorce、mapper节点)；然后解析指定的mapper配置文件生成全局的```Configuration```对象(包含配置信息)和N个```MappedStatement```对象(包含sql动态配置)
2. 接着利用建造者模式，```SqlSessionFactoryBuilder```通过配置对象```Configuration```创建全局```SqlSessionFactory```(线程安全)，并利用```SqlSessionFactory```获取```SqlSession```来进行与数据库的交互
3. 在程序调用```Mapper```接口时，利用全局唯一的MappedStatementId来找到对应的```MappedStatement```对象，这里用到了代理模式(JDK动态代理)；真正的Mapper调用对象都是通过```MapperProxyFactory```根据```Mapper```接口动态生成的。
4. 后面就是SQl的解析、拼接什么的了，通过```Excutor```将```MapperProxy```进行参数解析填充并进行解析动态的生成最后要执行的SQL
5. 执行JDBC操作，并对结果利用```MappedStatement```中的结果映射进行处理，转换成具体对象返回