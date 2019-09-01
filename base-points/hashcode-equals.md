# equals、hashCode方法的关联

`hashCode`方法是对象或变量通过哈希算法计算出的哈希值，哈希玛是用来在散列存储结构中确定对象的存储地址的，主要是用来Set集合中，用来做快速判断元素是否相等的

重写了`equals`方法，必须重写`hashCode`方法

equal()相等的两个对象他们的hashCode()肯定相等，也就是用equal()对比是绝对可靠的。
hashCode()相等的两个对象他们的equal()不一定相等，也就是hashCode()不是绝对可靠的。

##### 重写equals方法要注意

* 对称性：如果x.equals(y)返回是`true`，那么y.equals(x)也应该返回是`true`

* 反射性：x.equals(x)必须返回是`true`

* 类推性：如果x.equals(y)返回是`true`，而且y.equals(z)返回是`true`，那么z.equals(x)也应该返回是`true`

* 还有一致性：如果x.equals(y)返回是`true`，只要x和y内容一直不变，不管你重复x.equals(y)多少次，返回都是`true`

* 任何情况下，x.equals(null)，永远返回是`false`；x.equals(和x不同类型的对象)永远返回是`false`