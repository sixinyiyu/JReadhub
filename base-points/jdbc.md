现在项目中，还是非常少要手写JDBC代码的，大都时候我们都是借助的是ORM框架来实现对数据库的操作，其实ORM框架也只是封装了原生的JDBC操作，作为一位合格的Javaer还是非常有必要知道如何使用原生的JDBC代码来操作数据库。

* 注册驱动

```java
Class.forName("com.mysql.jdbc.Driver")
```

* 获取数据库连接

```java
Connection connection = DriverManager.getConnection(url,name,password);
```


* 创建Statement/PreparedStatement

PreparedStatement能够预编译SQL，效率比较高并且有效的阻止SQL注入

```java
PreparedStatement ps = connection.prepareStatement(sql);
```

* 执行SQL
```java
ps.executeQuery(sql);
```

* 处理结果ResultSet

* 释放资源连接等