线程池原理、创建线程池的参数含义

先看看线程池的创建函数
```java
	public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

发现调用的是```ThreadPoolExecutor```构造方法
```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue)
```

各参数含义:
* corePoolSize 核心线程数
* maximumPoolSize 最大线程数
* keepAliveTime 线程存活时间
* unit     时间单位
* workQueue  存放任务的队列

下面说说各个参数的作用！
> 首先假设```corePoolSize```是10，```maximumPoolSize```是20,```keepAliveTime/unit```是60s，```workQueue```是```LinkedBlockingQueue```无界队列； 一开始提交任务到线程池中，当前线程池中线程数为0，小于核心线程数；直接创建线程来执行当前任务(线程在指定时间内存活并且阻塞直到任务队列中有任务则唤起空闲的线程执行任务)，一直提交直到当前线程数中线程数等于核心线程数;此时再次有任务提交了，发现线程池中线程数等于核心线程数了，就不会再创建线程了，而是将线程提交到任务队列中去，有阻塞的线程去队列中取任务执行；假设一直提交一直提交，这个时候任务队列满的的话(达到队列容量)，再提交任务到队列中是不行的了，然后就会尝试再次创建线程(发现当前线程数小于最大线程数 嗯还是可以创建成功知道当前线程数达到了最大线程数);如果线程达到了最大线程数，并且队列也满了这个时候如果还提交任务的话就会执行reject策略(用户指定的)了