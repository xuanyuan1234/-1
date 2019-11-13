# 死锁

指两个或两个以上的进程在执行过程中，因争夺资源而造成的一种互相等待的现象，若无外力干涉那它们都将无法推进下去。如果系统资源充足，进程的资源请求都能够得到满足，死锁出现的可能性就很低。


## 原因
* 系统资源不足
* 进程运行推进的顺序不对
* 资源分配不当

## 例子

```java
//线程资源类
class ThreadLockDemo implements Runnable {

    private String lockA;
    private String lockB;

    public ThreadLockDemo(String lockA, String lockB) {
        this.lockA = lockA;
        this.lockB = lockB;
    }

    @Override
    public void run() {
        synchronized (lockA) {
            System.out.println(Thread.currentThread().getName() + "\t 持有" + lockA + " 尝试获取" + lockB);

            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            synchronized (lockB) {
                System.out.println(Thread.currentThread().getName() + "\t 持有" + lockB + " 尝试获取" + lockA);
            }
        }
    }
}


/**
 * 死锁例子
 */
public class DeadLockDemo {
    public static void main(String[] args) {
        String lockA = "lockA";
        String lockB = "lockB";

        new Thread(new ThreadLockDemo(lockA, lockB), "AAA").start();
        new Thread(new ThreadLockDemo(lockB, lockA), "BBB").start();
    }
}
```

### 分析是否为死锁

#### idea中Terminal输入jps -l查看当前运行进程

* jps 类似于linux下的 ps -ef | grep xxx，它是windows下的，只是用于查看java的。

```java
F:\2019H2\java_code>jps -l
12368 org.jetbrains.idea.maven.server.RemoteMavenServer
15744 sun.tools.jps.Jps
8368
9120 org.jetbrains.jps.cmdline.Launcher
6484 com.zhou.java.lock.DeadLockDemo
10924 org.jetbrains.jps.cmdline.Launcher
```

#### 查看6484进程的执行情况，使用jstack 6484

```java
Java stack information for the threads listed above:
===================================================
"BBB":
        at com.zhou.java.lock.ThreadLockDemo.run(DeadLockDemo.java:27)
        - waiting to lock <0x000000074051d908> (a java.lang.String)
        - locked <0x000000074051d940> (a java.lang.String)
        at java.lang.Thread.run(Thread.java:748)
"AAA":
        at com.zhou.java.lock.ThreadLockDemo.run(DeadLockDemo.java:27)
        - waiting to lock <0x000000074051d940> (a java.lang.String)
        - locked <0x000000074051d908> (a java.lang.String)
        at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```

