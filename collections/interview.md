

* HashMap扩容为原长度的2倍，默认初始容量为16，负载因子0.75

* HashMap、LinkedHashMap 均允许插入Key、Value为空的情况，如果Kev为空则会覆盖原值

* ArrayList默认初始容量是10，<默认扩容1.5倍 

* Vector默认初始容量是10，默认扩容是2倍，每个方法都采用```synchronized```保证同步

* ```HashMap/HashTable```区别
 1. ```HashMap```非线程安全而```HashTable```通过```synchronized```关键字保证了线程安全；所以在效率上```HashMap```要高
 2. 默认初始化容量，```HashMap```初始容量是16，而```HashTable```初始容量是11；```HashMap```每次扩容为原容量的2倍，而```HashTable```扩容为原2n+1
 3. ```HashMap```中允许存在Null的key或value，而```HashTable```不允许key或value为Null
 4. ```HashMap```在底层为了减少hash冲突采用了黑红树的结构，而```HashTable```没有这样的方案
