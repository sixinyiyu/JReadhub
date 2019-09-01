### HashMap原理剖析

> 以下基于 JDK1.8 分析

首先先说说HashMap的底层的数据结构，HashMap采用的是数组+链表。

![HashMap数据结构](http://wx3.sinaimg.cn/large/9c349f47gy1fxtjwgj6eej20l00bgmxn.jpg)

数组在存储空间上是连续的，占用内存大所以空间复杂度很大，但是数组的二分查找时间复杂度很小为O(1);特点就变成了寻址容易，插入或删除困难

链表存储空间离散，占用内存比较宽松，空间复杂度小，时间复杂度大为O(N)特点是：寻址困难，插入和删除容易

所以HashMap结合了数组+链表，并且在链表长度达到某一阀值时链表会转化为二叉树。


* 重要变量

```java
// 默认初始容量
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
// 最大容量
static final int MAXIMUM_CAPACITY = 1 << 30;
// 默认负载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;
// 链表默认长度为此值时转为二叉树
static final int TREEIFY_THRESHOLD = 8;
```

## put方法分析

put(a,b)的时候 hash(a)%len(数组) 确定这个值放在数组的那个索引上，所有就会存在A=put(a,b) B=put(c,b)的时候；a、c得到的数组索引都是0；然后先put A，因为数组内部的元素其实是Entry是一个链表；Entry[0]就是A,在putB的时候发现索引也是0，那Entry[0]=B,B.next=A这里就是一个链表结构 数组中存储的就是最后放入的元素。

1. 计算Key的hash值

2. 判断数组为空或长度为0，即未初始化则进行初始化

3. 根据数组长度以及Key的hash值确定数组索引，若索引为元素为空则直接插入

4. 数组索引i不为空，且Key的hash值相等，且k也相等则直接覆盖原值

5. 做索引位置为黑红二叉树则直接插入到二叉树中

6. 链表处理，遍历链表匹配到了则替换否则插入到链表尾部

7. Map中存在旧值则返回旧值

8. 再次判断是否需要扩容

```java
    public V put(K key, V value) {
  		// 1.首先计算key的hash值
        return putVal(hash(key), key, value, false, true);
    }

    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

```java
 
 final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 2.判断数组为空或长度为0,即未初始化则进行初始化
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null) // 3.根据数组长度以及Key的hash值确定数组索引，若索引为元素为空则直接插入
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            // 4.数组索引i不为空，且Key的hash值相等，且k也相等则直接覆盖原值
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)// 5.做索引位置为黑红二叉树则直接插入到二叉树中
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {// 6. 链表处理，遍历链表匹配到了则替换否则插入到链表尾部
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                    	// 已经是链表尾部，则直接插入数据在链表尾部
                        p.next = newNode(hash, key, value, null);
                        // 判断是否需要转为为二叉树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 一个一个的遍历链表
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            // 7. 存在旧值则返回旧值
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        // 8. 再次判断是否需要扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

## ```resize```扩容方法

```java
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) { // 到达最大容量
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }// 默认扩容2倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) { // 遍历数组
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null) // 数组位置，只有一个元素，直接将旧值放到新数组对应位置去
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)// 处理二叉树
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        // 遍历处理链表，一个一个复制处理
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```


## ```get```方法

1. 还是先计算Key的hash值

2. 数组为空直接返回Null，否则根据hash以及数组里长度确定hash所在的索引i的元素

3. 判断索引i元素的hash、key是否与目标一致，是则直接返回

4. 索引位置(至少是链表或者二叉树)，先匹配链表首节点，若为二叉树则利用二叉树查找，否则则遍历链表匹配

```java
	public V get(Object key) {
        Node<K,V> e;
        // 1.计算Key的hash值
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }


	final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
        	// 2.根据hash以及数组里长度确定hash所在的索引i
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
            	// 3. 节点为二叉树则二叉树快速查找
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                	// 4.节点为链表，则链表一个一个元素匹配
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```


## ```remove```方法

1. 还是老一套，先计算Key的hash值

2. 若node数组为空或长度为0则直接返回Null

3. 检查索引位置链表的第一个元素，看是否匹配

4. 根据索引位置节点判断进行二叉树获取还是链表获取

5. 若上述步骤匹配到了元素，开始删除元素（二叉树删除、头链表、多元素链表删除）

```java
    public V remove(Object key) {
        Node<K,V> e;
        //1. 还是老一套，先计算Key的hash值
        return (e = removeNode(hash(key), key, null, false, true)) == null ?
            null : e.value;
    }

    final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        Node<K,V>[] tab; Node<K,V> p; int n, index;
        // 2. 数组为空或长度为0则直接返回Null
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {
            Node<K,V> node = null, e; K k; V v;
            // 3. 检查索引位置链表的第一个元素，看是否匹配
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;
            else if ((e = p.next) != null) {
                // 4. 若索引位置元素为二叉树，则直接利用二叉树获取元素
                if (p instanceof TreeNode)
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                else {
                    // 索引位置元素为链表，则遍历匹配对应元素
                    do {
                        if (e.hash == hash &&
                            ((k = e.key) == key ||
                             (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            // 5. 若上述步骤匹配到了元素，开始删除元素，
            if (node != null && (!matchValue || (v = node.value) == value ||
                                 (value != null && value.equals(v)))) {
                // 二叉树删除
                if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                else if (node == p) // 链表头匹配，则直接将索引位指向链表下一个元素
                    tab[index] = node.next;
                else
                    p.next = node.next; // 链表多个元素，则该表链表指向
                ++modCount;
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }
```

> 总结

* ```HashMap```默认初始容量是16，负载因子是0.75，扩容为原来的2倍

* ```HashMap```底层的数据结构为数组+链表(以及二叉树)

* ```HashMap```不是线程安全的

* ```HashMap```不允许存放重复的Key，允许Key或Value为空