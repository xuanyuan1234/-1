# 线程池

后面带s的是辅助工具类。
// Array   Arrays
// Collection Collections
// Executor  Executors

# 架构

线程池的底层是 ThreadPoolExecutor

```java
public ThreadPoolExecutor(int corePoolSize,
                            int maximumPoolSize,
                            long keepAliveTime,
                            TimeUnit unit,
                            BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
            Executors.defaultThreadFactory(), defaultHandler);
}
```

# 实现

```java
public class ThreadPoolDemo {
    public static void main(String[] args) {
        //ExecutorService threadPool = Executors.newFixedThreadPool(5); //一池固定数据线程
        //ExecutorService threadPool = Executors.newSingleThreadExecutor();//一池单个
        ExecutorService threadPool = Executors.newCachedThreadPool();//缓存

        try {
            for(int i = 1; i <= 10; i++) {
                threadPool.execute(() -> { //Runnable 函数式接口，可采用lamba表达式
                    System.out.println(Thread.currentThread().getName() + "\t 办理业务");
                });

                //threadPool.submit();
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();
        }
    }
}
```

## newFixedThreadPool

```java
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

## newSingleThreadExecutor

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

## newCachedThreadPool

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                    60L, TimeUnit.SECONDS,
                                    new SynchronousQueue<Runnable>());
}
```

## newScheduledThreadPool

## newWorkStealingPool


# 七大参数意义

* corePoolSize: 核心线程数
* maximumPoolSize：最大线程数
* keepAliveTime：
* TimeUnit：
* BlockingQueue<Runnbale>：
* threadFactory：
* RejectedExecutionHandler：拒绝策略



# 线程池拒绝策略

**等待队列已经排满了**，再也塞不下新任务。同时，**线程池中的max线程也达到了**，无法继续为新任务服务。这个时候需要拒绝策略机制，来合理处理这个问题。

* AbortPolicy（默认）：直接抛出RejectedExecutionException异常阻止系统正常运行；
* DiscardPolicy：直接丢弃任务，不予处理也不抛出异常。如果允许任务丢失，这是最好的一种方案；
* CallerRunsPolicy："调用者运行"一种调节机制，不会抛弃任务，也不会抛出异常，而是将某些任务回退到调用者；
* DiscardOldestPolicy：抛弃队列中等待最久的任务，然后把当前任务加入队列中尝试再次提交当前任务

# 工作中单一的/固定的/可变的三种创建线程池的方法，你用哪个多？超线大坑

一个都不用。原因：据阿里巴巴规约

并发编程下：

固定的/单一的线程，调用了LinkedBlockingQueue队列，支持MAX_VALUE个数，容易造成OOM。

可变的，支持MAX_VALUE个数，容易造成OOM

# 自定义线程池

```java
public class ThreadPoolDemo {
    public static void main(String[] args) {
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                2,
                5,
                1L,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy()
        );

        try {
            for(int i = 1; i <= 10; i++) {
                threadPoolExecutor.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + "\t 办理业务");
                });
            }
        } finally {
            threadPoolExecutor.shutdown();
        }
    }
}
```

* AbortPolicy策略（**抛出异常**）
```java
pool-1-thread-1	 办理业务
Exception in thread "main" java.util.concurrent.RejectedExecutionException: Task
pool-1-thread-1	 办理业务
```

* CallerRunsPolicy策略（**任务回退给调用者**）
```java
pool-1-thread-1	 办理业务
main	 办理业务
main	 办理业务
pool-1-thread-1	 办理业务
```

* DiscardPolicy、DiscardOldestPolicy策略 （**数量变少，任务被丢弃**）
```java
pool-1-thread-1	 办理业务
pool-1-thread-2	 办理业务
pool-1-thread-2	 办理业务
pool-1-thread-2	 办理业务
pool-1-thread-2	 办理业务
pool-1-thread-3	 办理业务
pool-1-thread-4	 办理业务
pool-1-thread-5	 办理业务
```

# 如何合理配置线程池参数
* CPU密集型：
    配置尽可能少的线程数量（一般公式：CPU核数+1个线程的线程池）
* IO密集型：
    1、由于IO密集型任务线程并不是一直在执行任务，则应配置尽可能多的线程，如CPU核数 * 2
    2、即该任务需要大量的IO，即大量的阻塞。参考公式：CPU核数/(1-阻塞系数)  阻塞系统在0.8~0.9之间。比如8核CPU： 8/ (1-0.9) = 80个线程数