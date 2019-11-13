# CAS

不加锁，保证一致性。但是自旋锁需多次比较，CPU开销大。

# synchronized
一致性保证，并发性下降。

CAS ---> UnSafe ---> CAS底层原理 ---> ABA ---> 原子引用更新 ---> 如何规避ABA问题

# CAS(compare and swap)
在计算机科学中，比较和交换（Conmpare And Swap）是用于实现多线程同步的原子指令。它是一条CPU并发原语。它的功能是**判断内存某个位置的值是否为预期值，如果是则更改为新的值，这个过程是原子的**。


CAS并发原语体现在sum.mics.Unsafe类中的各个方法。调用UnSafe类中的CAS方法，**JVM会帮我们实现出CAS汇编指令**。是依赖**硬件**的功能，实现原子操作。

CAS是系统原语，由若干条指令组成，原语的执行必须是连续的，不允许被中断。**不会造成数据不一致的问题**。


# 底层原理

```java
AtomicInteger atomicInteger = new AtomicInteger();
public void addMyAtomic() {
    atomicInteger.getAndIncrement();
}
```

## Unsafe类 (jdk中sun.misc)
```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;
```

### unsafe.getAndAddInt
```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}

public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

### 底层汇编

![指令重排](/assert/view/juc/cas/1_1.png)

## 自旋锁

getAndAddInt中的do while循环，线程循环检测工作内存中的值与主内存中的值是否相等，不等的话，则失败，则继续循环检测。相等，则更新主内存中的值。

# CAS缺点
* 循环时间长开销很大。（由于do while检测，如果值一直改变，则会一直循环尝试，CPU开销增大）
* 只能保证**一个**共享变量的原子操作
* 引出来ABA问题？？？

## ABA问题

多线程中，one、two线程各自从主内存取出变量A，two线程由于执行快，先将主内存中的值刷新为B，又刷新为A。当one线程CAS检测时，发现主内存中的值为A，与自已工作空间中的值相等，就将自身的值刷新回主内存。
**one线程刷新值成功，不代表中间过程就没有问题**。

## 解决手段 （原子引用）
```java
class User {
    String userName;
    int age;

    User(String userName, int age) {
        this.userName = userName;
        this.age = age;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "userName='" + userName + '\'' +
                ", age=" + age +
                '}';
    }
}

public class AtomicReferenceDemo {
    public static void main(String[] args) {

        User z3 = new User("z3", 22);
        User li4 = new User("li4", 25);


        AtomicReference<User> atomicReference = new AtomicReference<>();
        atomicReference.set(z3);

        System.out.println(atomicReference.compareAndSet(z3, li4) + " value: " + atomicReference.get().toString());
        System.out.println(atomicReference.compareAndSet(z3, li4) + " value: " + atomicReference.get().toString());
    }
}
```

## 时间戳原子引用（新增一种机制，就是类似修改版本号）

T1  100  1                   100  1

T2  100  1       101  2      100  3

```java
public class ABADemo {

    static AtomicReference<Integer> atomicReference = new AtomicReference<>(100); //原子引用
    static AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(100, 1); //时间戳原子引用

    public static void main(String[] args) {

        System.out.println("===============ABA生成=====================");

        new Thread(() -> {
            atomicReference.compareAndSet(100, 101);
            atomicReference.compareAndSet(101, 100);
        }, "t1").start();

        new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(1); //让t1线程生成一次ABA操作
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            atomicReference.compareAndSet(100, 2019);

            System.out.println(Thread.currentThread().getName() + "\t value: " +  atomicReference.get().toString());
        }, "t2").start();

        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }


        System.out.println("===============ABA解决=====================");

        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName() + "\t 第1次版本号： " + stamp);

            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            atomicStampedReference.compareAndSet(100, 101, atomicStampedReference.getStamp(), atomicStampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName() + "\t 第2次版本号： " + atomicStampedReference.getStamp());
            atomicStampedReference.compareAndSet(101, 100, atomicStampedReference.getStamp(), atomicStampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName() + "\t 第3次版本号： " + atomicStampedReference.getStamp());
        }, "t3").start();

        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName() + "\t 第1次版本号： " + stamp);
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            boolean result = atomicStampedReference.compareAndSet(100, 2019, stamp, stamp + 1);
            System.out.println(Thread.currentThread().getName() + "\t 刷新成功否： " + result + "\t 版本号实际值为：" + atomicStampedReference.getStamp());
            System.out.println("当前值为：" + atomicStampedReference.getReference().toString());
        }, "t4").start();
    }
}
```


