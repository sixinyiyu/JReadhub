前面我们分析了```HashMap```,知道```HashMap```是无序的，如果要保证按插入顺序来访问Map则需要使用```LinkedHashMap```。


```LinkedHashMap``` 继承自 ```HashMap``` ，又重写了其中了一些结构方法，底层的结构还是采用的数组、链表(红黑树)的方式；只是链表是双向的每个节点都持有前后节点的指针。



![结构示意图](http://wx1.sinaimg.cn/large/9c349f47gy1fxuppt7q3cj20kc0dvgly.jpg)


```java
	static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after; // 前后元素
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }

    /**
     * The head (eldest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> head;  // 双向链表的头

    /**
     * The tail (youngest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> tail; // 双向链表的尾
```

接着我们看看```LinkedHashMap```定制的```Node```结构

```java
	Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
        LinkedHashMap.Entry<K,V> p =
            new LinkedHashMap.Entry<K,V>(hash, key, value, e);
        linkNodeLast(p);
        return p;
    }

    private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
        LinkedHashMap.Entry<K,V> last = tail;
        tail = p;
        // 将p节点插入到双向链表的尾部
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
    }
```

:tada:


### ```get```方法

1. 首先还是计算Key的hash值

2. 判断节点数组、以及根据Key计算的索引位元素是否空

3. 首先匹配索引i table[i]处链表的头是否为要匹配的值，如果不是则看链表是链表还是二叉树

4. 根据不同的情况进行处理

5. 将匹配到的元素移到链表的尾部


```java
	public V get(Object key) {
        Node<K,V> e;
        // 1. 首先还是计算Key的hash值
        if ((e = getNode(hash(key), key)) == null)
            return null;
        if (accessOrder) // accessOrder为构造函数传入，默认是false(false按照插入顺序来连接，true则为按照访问顺序来连接)
            afterNodeAccess(e); // 如果按照访问顺序来，则需要将当前访问的节点移到双向链表尾部
        return e.value;
    }


	final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {  // 2. node数组不为空且数组所在`桶`不为空
            if (first.hash == hash && // always check first node  3. 检查链表头
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) { // 链表不止一个元素
                if (first instanceof TreeNode) // 3.按二叉树处理
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do { // 4. 遍历链表
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```