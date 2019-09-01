
前面我们分析了```HashMap```,知道了其实现原理，也知道它不是线程安全的，在平时编码中要使用线程安全的Map，当然想到的是```ConcurrentHashMap```了，接下来我们就分析下其实现原理看看是怎么做到线程安全的。

> 首先看看一些成员变量

```java
	// 最大容量
	private static final int MAXIMUM_CAPACITY = 1 << 30;

	// 默认容量
	private static final int DEFAULT_CAPACITY = 16;

	// 负载因子
	private static final float LOAD_FACTOR = 0.75f;

	// 链表转二叉树阀值
	static final int TREEIFY_THRESHOLD = 8;

	// 红黑树转为链表阀值
	static final int UNTREEIFY_THRESHOLD = 6;

	// table最小容量,只有当table。length>=64 才会出现二叉树
	static final int MIN_TREEIFY_CAPACITY = 64;

	// 存储数据的数组
	transient volatile Node<K,V>[] table;
```


## ```put```方法

1. 判断入參，这里可以知道```ConcurrentHashMap```的Key或Value不能为空

2. 计算key的hash值，判断是否需要初始化存储数组

3. 根据Key的hash值与数组长度计算Key所在是索引,得到tab[i]的元素，

4. 如果元素为空则利用CAS操作直接设值，知道成功返回；否则看是否在扩容，如果是则帮助处理扩容；否则则锁住此节点出桶进行后续处理

5. 再次利用CAS操作获取索引i处节点，判断桶节点是链表(链表则直接插入)还是二叉树(如果链表中存在相同的key则进行值覆盖，否则直接插入到链表尾部)

6. 检查链表长度是否达到转化为二叉树的阀值，如果是则将链表转为二叉树

先看下面代码
```java
	public V put(K key, V value) {
        return putVal(key, value, false);
    }

    final V putVal(K key, V value, boolean onlyIfAbsent) {
    	// 1. 判断入參，这里可以知道```ConcurrentHashMap```的Key或Value不能为空
        if (key == null || value == null) throw new NullPointerException();
        // 2. 计算Key的hash值
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            // 3. 数组未初始化则进行初始化
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            // 4. 根据Key的hash值与数组长度计算Key所在是索引,得到tab[i]的元素，如果元素为空
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            	// 则利用CAS操作设置索引为i的值
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            } // 5.如果正在扩容则帮忙进行扩容
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                synchronized (f) { // 锁某个桶节点
                    if (tabAt(tab, i) == f) { // 利用CAS 获取索引i出的桶节点 (此处用CAS操作而不直接用索引是为了保证得到的桶节点是最新的)
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) { // 遍历链表
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) { // 匹配链表的每一个元素
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) { // 插入链表尾部
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2; // 如果节点处为二叉树则进行二叉树插入
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD) // 超过阀值进行红黑二叉树转换
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
```

## ```get```方法

1. 计算key的hash值

2. 如果table为空或者根据hash与table长度计算的索引处桶为空直接返回空

3. 根据匹配到的桶，首先先判断桶的第一个节点是否是要匹配的(比较hash以及key是否相等)，若没匹配到则根据情况二叉树查找或遍历链表查询

```java
	public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
        // 1. 计算key的hash值
        int h = spread(key.hashCode());
        // 2. 判断table数组，若不为空则利用CAS操作获取索引位的桶节点
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (e = tabAt(tab, (n - 1) & h)) != null) {
        	// 3. 判断桶节点的第一个元素，如果hash值一样且key也相等则匹配到了
            if ((eh = e.hash) == h) {
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
            // 4. 在二叉树中查找
            else if (eh < 0)
                return (p = e.find(h, key)) != null ? p.val : null;
            // 5. 遍历链表查询
            while ((e = e.next) != null) {
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
```