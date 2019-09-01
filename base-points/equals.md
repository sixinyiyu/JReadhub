* equals、'==' 区别

首先`equals`方法是用来判断对象是否相等的，通过判断两个对象的地址是否一致

```java
public boolean equals(Object obj) {
        return (this == obj);
    }
```

默认`equals`方法等价于'=='操作符，都是判断对象的地址是否相等来处理，项目中一般都会根据业务重写对象的`equals`方法


在项目中，'=='操作符一般用来判断基本数据类型(特殊性)，对于对象一般用`equals`方法比较
