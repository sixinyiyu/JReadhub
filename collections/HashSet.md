```HashSet``` 实现了```Set```接口，其底层是通过```HashMap```来实现具体的功能，不允许存放重复元素。

> 首先看看属性

通过注解可以发现，底层就是通通过```HashMap```存储，其中```HashMap```的Key就是```HashSet```存储的值，Value固定是一个Object对象填充，其中```HashMap```的默认容量是16(也就是```HashSet```初始化容量),负载因子是0.75。

```java

	private transient HashMap<E,Object> map;

    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();

    /**
     * Constructs a new, empty set; the backing <tt>HashMap</tt> instance has
     * default initial capacity (16) and load factor (0.75).
     */
    public HashSet() {
        map = new HashMap<>();
    }
```

> 总结

* ```HashSet```底层使用的是```HashMap```来实现所以很多特性基本上一样，比如初始容量、负载因子，扩容等

* ```HashSet```是无序的，允许存储空值

* ```HashSet```通过存储元素的```hashcode```、```equals```方法来判断是否重复