## ```ArrayList```

`ArrayList`实现了```RandomAccess```,```List```接口；基于数组实现可以实现快速访问(数组是连续空间存储)，数组的扩容使用的是Arrays.copyOf实现！

`ArrayList`底层数据结构为数组,允许插入重复数据，默认初始容量为10。`ArrayList`不是线程安全的，基于数组实现可以实现快速访问(数组是连续空间存储)，数组的扩容使用的是`Arrays.copyOf`实现！


首先我们看看一些变量

```java
// 默认容量
private static final int DEFAULT_CAPACITY = 10;

// 采用数组存数据
transient Object[] elementData;
```

### ```add```方法

```java
	// 普通add方法
	public boolean add(E e) {
		// 扩容处理
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // 数组最后加入元素
        elementData[size++] = e;
        return true;
    }
    // 指定位置add
    public void add(int index, E element) {
    	// 1.验证参数
        rangeCheckForAdd(index);
        // 2.扩容处理
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        // 3.数组拷贝 向后移，空出index位置
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        // 指定索引添加元素
        elementData[index] = element;
        size++;
    }
```

主要是要关注下扩容方法

```java
	private static int calculateCapacity(Object[] elementData, int minCapacity) {
		// 空数组则去当前容量与默认容量中大的
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }

    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        // 默认扩容1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        // 采用数组拷贝快速复制到新数组中
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

**通过上面的方法，发现每次add元素的时候都会检查扩容，扩容就会存在数组拷贝，还是比较耗时的，所以在日常开发中在使用ArrayList的时候尽量要指定大小，尽量减少扩容。**


### ```remove```方法

```java
	// 指定索引位置删除
	public E remove(int index) {
		// 1.参数检查
        rangeCheck(index);

        modCount++;
        // 2.找到索引位置元素
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
        	// 3.索引位置后元素往前移动
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }

	public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
    // 匹配到第一个元素进行删除
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```

**```ArrayList```插入、删除大概率会存在数组拷贝所以效率会比较差，查询是根据索引所以非常的高效**

## ```Vector```

```Vector``` 也是实现于```List```接口，底层数据结构和```ArrayList```类似,也是一个动态数组存放数据。

```add```方法实现也差不多，不过是用了```synchronized```保持同步

```java
	public synchronized boolean add(E e) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = e;
        return true;
    }

    public void add(int index, E element) {
        insertElementAt(element, index);
    }

    public synchronized void insertElementAt(E obj, int index) {
        modCount++;
        if (index > elementCount) {
            throw new ArrayIndexOutOfBoundsException(index
                                                     + " > " + elementCount);
        }
        // 扩容
        ensureCapacityHelper(elementCount + 1);
        System.arraycopy(elementData, index, elementData, index + 1, elementCount - index);
        elementData[index] = obj;
        elementCount++;
    }

    // 扩容方法
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

```Vector```的扩容方法与```capacityIncrement```参数有关，此参数在构造函数传入，默认是0；所以Vector默认扩容2倍。
此外```Vector```还有个子类```Stack```栈