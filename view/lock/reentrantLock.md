# 可重入锁（又名递归锁）


# 定义

指同一线程外层函数获取锁之后，内层递归函数仍然能获取该锁的代码  
即：**线程可以进入任何一个它已经拥有的锁所同步着的代码块**。

```java
public sync void method01() {
    method02();
}

public sync void method02() {

}
```

* ReentrantLock/Synchronized就是一个典型的可重入锁
* 可重入锁最大的作用是<font color="red">**避免死锁**</font>。

```java
class Phone implements Runnable {
    public synchronized void sendSMS() {
        System.out.println(Thread.currentThread().getName() + "\t invoke sendSMS()");
        sendEmail();
    }

    public synchronized void sendEmail() {
        System.out.println(Thread.currentThread().getName() + "\t invoke sendEmail()");
    }

    Lock lock = new ReentrantLock();

    @Override
    public void run() {
        get();
    }

    public void get() {
        lock.lock();
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t invoke get()");
            set();
        } finally {
            lock.unlock();
        }
    }

    public void set() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t invoke set()");
        } finally {
            lock.unlock();
            lock.unlock();
        }
    }
}

public class ReentrantDemo {

    public static void main(String[] args) {
        Phone phone = new Phone();

        new Thread(() -> {
            phone.sendSMS();
        }, "t1").start();

        new Thread(() -> {
            phone.sendSMS();
        }, "t2").start();

        Thread t3 = new Thread(phone, "t3");
        Thread t4 = new Thread(phone, "t4");

        t3.start();
        t4.start();
    }
}
```

其中：lock可以加多次，只要保证 lock与unlock的次数一致，都是可以正常运行。另外：如果不一致，会导致程序一直执行，无输出结果。


下面这种也是可以正常执行。
```java
public void get() {
    lock.lock();
    try {
        System.out.println(Thread.currentThread().getName() + "\t invoke get()");
        set();
    } finally {
        lock.unlock();
    }
}

public synchronized void set() {
    try {
        System.out.println(Thread.currentThread().getName() + "\t invoke set()");
    } finally {
    }
}
```

# synchronized和lock有什么区别？用新的lock有什么好处？举例说说

* 原始构成
  synchronized是关键字属于JVM层面。
  monitorenter(底层是通过monitor对象来完成，其实wait/notify等方法也依赖于monitor对象只有在同步块或方法中才能调wait/nofity等方法)
  monitorexit

  通过javap -c分析，有一次enter，2次exit。代表 正常退出，异常退出。保证不会死锁。

  lock是具体类(java.util.concurrent.locks.Lock)是api层面的锁

* 使用方法
  synchronzied不需要用户去手动释放锁，代码执行完系统让线程自动释放锁
  reentrantLock需要用户去手动释放锁，若没有主动释放锁，则有可能出现死锁。需要lock和unlock方法配合try/finally语句块来完成。

* 等待是否可中断
  synchronzied不可中断，除非抛出异常或者正常运行完成
  reentrantLock可中断，1、设置超时方法 tryLock 2、lockInterruptibly放代码块中，调用interrupt方法可中断

* 加锁是否公平
  synchronized非公平锁
  reentrantLock默认非公平锁。构造方法传参，true为公平锁，flase非公平锁

* 锁绑定多个条件Condition
  synchronized没有
  reentrantLock用来实现分组响醒需要唤醒的线程们，可以精确响醒，而不是像synchronized要么随机唤醒一个线程要么唤醒全部线程。

```java
  /**
 * 3个线程
 * <p>
 * A打印5次，B打印10次，C打印15次。
 * 总共打印10轮
 */

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 多线程编程，步骤：
 * 1、创建资源类
 * 2、判断、干活、通知
 */

class ShareResource {
    private int number = 1; //A:1, B:2, C:3
    Lock lock = new ReentrantLock();
    private Condition c1 = lock.newCondition();
    private Condition c2 = lock.newCondition();
    private Condition c3 = lock.newCondition();

    public void print(int flag) {
        lock.lock();

        try {
            if (flag == 1) {
                while (number != 1) {
                    c1.await();
                }
            }

            if (flag == 2) {
                while (number != 2) {
                    c2.await();
                }
            }

            if (flag == 3) {
                while (number != 3) {
                    c3.await();
                }
            }

            for (int i = 1; i <= 5 * number; i++) {
                System.out.println(Thread.currentThread().getName() + "\t " + i);
            }

            if (number == 1) {
                number = 2;
                c2.signal();
            } else if (number == 2) {
                number = 3;
                c3.signal();
            } else if (number == 3) {
                number = 1;
                c1.signal();
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}

public class ReentrantLockDemo {
    public static void main(String[] args) {
        ShareResource shareResource = new ShareResource();

        new Thread(() -> {
            shareResource.print(1);
        }, "A").start();

        new Thread(() -> {
            shareResource.print(2);
        }, "B").start();

        new Thread(() -> {
            shareResource.print(3);
        }, "C").start();
    }
}
```
