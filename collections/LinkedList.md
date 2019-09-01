
```LinkedList```实现了 ```List```、 ```Deque``` 接口，既有列表的特性又有队列的特性。


其数据结构如下：

![数据结构](http://wx1.sinaimg.cn/large/9c349f47gy1fxusevb1c5j20r4082weo.jpg)


底层采用的是双向链表结构，结构节点都有指针指向前一个节点、后一个节点；其内部节点的实现


```java
	private static class Node<E> {
        E item; // 当前节点数据
        Node<E> next; // 下一个节点
        Node<E> prev; // 前一个节点

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

下面看看主要方法的实现


## ```add```方法


```java
	// 普通插入
	public boolean add(E e) {
        linkLast(e); // 插入到双向链表尾部
        return true;
    }

    // 指定索引插入
    public void add(int index, E element) {
        checkPositionIndex(index); // 校验索引

        if (index == size) //插入到最后
            linkLast(element);
        else
            linkBefore(element, node(index));
    }
```


```linkLast```方法

```java
 void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null); // 根据当前的数据构造节点，上一个节点为原尾节点
        last = newNode;
        if (l == null) // 尾节点为空，说明整个list为空则新则的节点即是头又是尾
            first = newNode;
        else 
            l.next = newNode; // 原尾节点的下一个节点为当前新生成的节点
        size++; // 长度+1
        modCount++;
    }
```


```linkBefore```方法

```java
	void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ); // 替换掉index位置的节点
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }

    // 根据索引找到对应位置元素，利用双向链表特性，判断index里链表那边近，如果index接近size/2的时候效率非常高
	Node<E> node(int index) {
        // assert isElementIndex(index);

        if (index < (size >> 1)) { 
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {  
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```


## ```get```方法

```java
	 public E get(int index) {
        checkElementIndex(index);
        // 查询到索引位置的节点
        return node(index).item;
    }
```


## ```remove```方法

```java

	public E remove(int index) {
		// 校验索引
        checkElementIndex(index);
        return unlink(node(index));
    }


	public boolean remove(Object o) {
        if (o == null) { 
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }

```

```java
E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) { // 删除的节点为头节点
            first = next;// 头节点之下下一个节点元素
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) { // 删除的节点为尾节点
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
```


* **```LinkedList```插入方法都是移动指针，相当的高效**

* **```LinkedList```查询/删除方法的时候会存在遍历节点的情况，相对来说效率差一点**

* **允许插入空元素**