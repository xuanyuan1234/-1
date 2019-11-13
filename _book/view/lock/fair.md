# 公平锁与非公平锁


# 定义

```java
public ReentrantLock() {
    sync = new NonfairSync();
}

public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

* 公平锁      指多个线程按照申请锁的顺序来获取锁，类似排队打饭，先来后到
* 非公平锁    指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程以先申请的线程优先获取锁。**在高并发情况下，有可能会造成优先级反转或饥饿现象**

# 区别

* 公平锁    按照申请锁的顺序来获取锁。FIFO等待队列
* 非公平锁  先直接尝试占有锁，如果失败，就再采用**类似公平锁那种方式**

**非公平锁的优点在于吞吐量比公平锁大。**

对于synchronized而言，也是一种**非公平锁**。
