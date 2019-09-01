# 设计模式の单例模式

单例模式，也叫单子模式，是一种常用的软件设计模式。当需要确保某个类只要一个对象，或创建一个类需要消耗的资源过多，如访问IO和数据库操作等这个时候就需要考虑单例模式了，大家应该都知道`Spring`的bean基本上都是单例的。

单例模式有以下特点：

* 单例类只能有一个实例
* 单例类必须自己创建自己的唯一实例
* 单例类必须给所有其他对象提供这一实例

### 单例模式的几种写法

#### 饥汉模式

```java
public class Singleton {

    private static Singleton instance = new Singleton();

    private Singleton(){
    }

    public static Singleton getInstance() {
        return instance;
    }

}
```

饥汉模式由私有构造方法和静态方法构成。饥汉顾名思义即第一次引用类的时候就创建实例，不管是否需要但无法做到延迟加载。

#### 懒汉模式

```java
public class Singleton {

    private static Singleton singleton = null;

    private Singleton(){}

    public static Singleton getSingleton() {
        if(singleton == null) singleton = new Singleton();
        return singleton;
    }
}
```

这种懒汉模式能在工厂方法里对`singleton`进行了Null判断，如果为Null才创建实例，这样能做到延迟加载的效果，不过这种方法不是线程安全的。假设有两个线程同时调用`getSingleton`那么就有可能都会进入if判断从而会出现多个`Singleton`实例，这也就违背了只有一个实例的特点。

#### 线程安全的懒汉模式


说到线程安全基本上就涉及到锁。
```java
public class Singleton {
    // 利用volatile关键字实现内存可预见性
    private volatile static Singleton instance;

    private Singleton(){
    }

    public static Singleton getInstance() {
        synchronized (Singleton.class) {
            if (null == instance)
                instance = new Singleton();
        }
        return instance;
    }

}
```

上面的写法虽然考虑了线程安全不过效率太低，在`getInstance`加的`synchronized`使其每次调用都得为获取锁排队。

#### 双重验证

于是就有了下面的兼顾线程、效率的方法。
```java
public class Singleton {

    private volatile static Singleton instance;

    private Singleton(){
    }

    public static Singleton getInstance() {
        if (null == instance){
            synchronized (Singleton.class) {
                if (null == instance)
                    instance = new Singleton();
            }
        }
        return instance;
    }

}
```

写法跟上面差不多，进行了两次空检查，所以也叫双重锁检查、双重验证方法；相比而言效率提高了很多因为真正执行new的逻辑的情况还是很少的。

#### 更优雅的写法静态内部类

静态内部类(推荐使用)
```java
public class Singleton {


    private Singleton(){
    }

    public static Singleton getInstance(){
        return SingletonHolder.instance;
    }

    private static class SingletonHolder{
        private  static Singleton instance = new Singleton();
    }

}
```

jvm第一次加载Singleton的时候不会初始化实例，而是在第一次调用`getInstance`的方法时，虚拟机会加载`SingletonHolder`类从而初始化了`instance`。做到了既线程安全又兼顾了延迟加载的效果推荐使用这种方式。