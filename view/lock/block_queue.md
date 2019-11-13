# 阻塞队列


# 定义

![](/assert/view/lock/block_queue/1_1.png)


* 当阻塞队列是空时，从队列中**获取**元素的操作会被阻塞
* 当阻塞队列是满时，往队列中**添加**元素的操作会被阻塞

# 为什么用？有什么好处？

多线程环境下，所谓阻塞，在某些情况下会**挂起**线程（即阻塞），一旦条件满足，被挂起的线程又会自动**被响醒**

## 为什么需要BlockingQueue

好处是我们不需要关心什么时候需要阻塞线程，什么时候需要响醒线程，因为这一切BlockingQueue都给你一手包办了。

在concurrent包发布之前，在多线程环境下，我们每个程序员都必须去自已控制这些细节。尤其要兼顾效率和线程安全，带来了不小的复杂度。


# 异常组

```java
public static void main(String[] args) {
    //List list = new ArrayList<>();
    1、//由数组结构组成的有界阻塞队列
    BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
    System.out.println(blockingQueue.add("a"));
    System.out.println(blockingQueue.add("b"));

    2、//由链表结构组成的有界（但大小默认Integer.MAX_VALUE）阻塞队列
    BlockingQueue<String> blockingQueue = new LinkedBlockingQueue<>(3);

    3、//不存储元素的阻栋队列，也即单个元素消费后，才生产下个元素
    BlockingQueue<String> blockingQueue = new SynchronousQueue<>();
```

## add (插入)

```java
blockingQueue.add("a")   // IllegalStateException: Queue full
```

## remove (移除)

```java

blockingQueue.remove()  //the head of this queue.if null, then NoSuchElementException
```

## element (检查)

```java
blockingQueue.element() //the head of this queue.if null, then NoSuchElementException
```

# 特殊值

## offer （插入）

成功true,失败false

## poll (移除)

成功返回队列头部信息，失败则返回null

## peek (检查)

the head of this queue, or {@code null} if this queue is empty

# 阻塞

## put(e) （插入）

当阻塞列表满时，生产者线程继志往队列中put元素，队列会一直阻塞线程直到put数据or响应中断退出

## take() (移除)

当阻栋列表空时，消费者线程试图从队列中take元素，队列会一直阻塞消费者线程直到队列可用

# 超时

## offer(e, time, out) （插入）

time时间到后，插入成功返回true,失败返回false。不阻塞

## poll(time, unit) (移除)

time时间到后，获取成功返回头元素，失败返回null


```java
public static void main(String[] args) throws InterruptedException {
    //List list = new ArrayList<>();
    BlockingQueue<String> blockingQueue = new SynchronousQueue<>();

    new Thread(() -> {
        try {
            System.out.println(Thread.currentThread().getName() + "\t put 1");
            blockingQueue.put("1");

            System.out.println(Thread.currentThread().getName() + "\t put 2");
            blockingQueue.put("2");

            System.out.println(Thread.currentThread().getName() + "\t put 3");
            blockingQueue.put("3");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }, "AAA").start();

    new Thread(() -> {
        try {
            TimeUnit.SECONDS.sleep(3);
            System.out.println(Thread.currentThread().getName() + "\t take " + blockingQueue.take());

            TimeUnit.SECONDS.sleep(3);
            System.out.println(Thread.currentThread().getName() + "\t take " + blockingQueue.take());

            TimeUnit.SECONDS.sleep(3);
            System.out.println(Thread.currentThread().getName() + "\t take " + blockingQueue.take());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }, "BBB").start();
}
```

# BlockingQueue核心


# 架构
