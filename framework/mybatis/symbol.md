
`#`与`$`的区别最大在于：`#{}` 传入值时，sql解析时，参数是带引号的，而`${}`穿入值，sql解析时，参数是不带引号的

`#{}`: 解析为一个JDBC预编译语句（`PreparedStatement`）的参数标记符，一个 `#{ }` 被解析为一个参数占位符 

`${}`: 仅仅为一个纯碎的字符串替换，在动态 SQL 解析阶段将会进行变量替换