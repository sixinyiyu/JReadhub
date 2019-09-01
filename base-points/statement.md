
`Statement`与`PreparedStatement`的对象都可以用来执行sql语句但是在许多方面我们还是使用的`PreparedStatement`类的对象比较多，
`PreparedStatement`是`Statement`的子类，后者可以预编译sql语句，预编译后的sql语句储存在`PreparedStatement`对象中，
可以多次高效的执行sql语句，`PreparedStatement`还支持批量操作，
`PreparedStatement`使用了占位符相对于`Statement`的硬编码来说要方便些，也能安全能够有效的阻止SQL注入