# volatile


# 概念

volatile是java虚拟机提供的轻量级的同步机制
* 保证可见性 （多线程中，某个线程修改了工作内存中变量值，并同步到主内存中的变量值，该变量值的变化，对其它线程是可见的。其它线程使用该值时，会去主内存中重新获取值）
* 不保证原子性 （不可分割，完整性。要么全部执行，要么全部不执行）
* 禁止指令重排

## 保证可见性
  多线程中，主内存中变量值的变化，会通知其它线程。其它线程调用时，会重新去主内存中进行拷贝。
```java
class myData {
    volatile int number = 0;

    public void addTo60() {
        this.number = 60;
    }
}

public void showVolatile() {
    myData myData = new myData();

    new Thread(()->{
        System.out.println(Thread.currentThread().getName() + " come in");
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        myData.addTo60();
        System.out.println(Thread.currentThread().getName() + " update value :" + myData.number);
    }, "AAA").start();

    while (myData.number == 0) { //AAA线程修改number后，会刷新到主内存中，并通知main线程。跳出while循环

    }

    System.out.println(Thread.currentThread().getName() + " value:" + myData.number);
}
```

## 不保证原子性
  某个线程对工作空间的值修改，刷新到主内存中的同时。其它线程被挂起。当其它线程唤醒时，由于CPU执行快，变量值的变化还没来得及通知，其它线程就把各自工作空间中的值刷新回主内存。**导致主内存值覆盖丢失**

```java
class myData {
    volatile int number = 0;

    public void addPlusPlus() {
        number++;
    }
}

public static void main(String[] args) {
    myData myData = new myData();

    for(int i = 1; i <= 20; i++) {
      new Thread(() -> {
          for (int j = 0; j < 1000; j++) {
              myData.addPlusPlus();
              myData.addMyAtomic();
          }
      }, String.valueOf(i)).start();
    }

    while (Thread.activeCount() > 2) { //2代表，后台有一个 main线程和GC线程。大于2，代表上述线程还没有执行完成。
        Thread.yield();
    }

    System.out.println(Thread.currentThread().getName() + " Int value:" + myData.number); //值不一定为20000，多线程覆盖丢失，不保证原子性
    System.out.println(Thread.currentThread().getName() + " AtomicInteger value:" + myData.atomicInteger); //CAS原子操作，保证原子性
}
```

### 如何解决原子性
* 加sync （如synchronized）
* 使用JUC下的atomicInteger （**参见不保证原子性示例**）

```java
class myData {
    volatile int number = 0;

    AtomicInteger atomicInteger = new AtomicInteger();
    public void addMyAtomic() {
        atomicInteger.getAndIncrement();
    }
}

```

## 禁止指令重排
volatile实现**禁止指令重排优化**，从而避免多线程环境下程序出现乱序执行的现象。

计算机在执行程序时，为了提高性能，编译器和处理器常常会对指令做重排，一般分为以下3种：

![指令重排](/assert/view/juc/volatile/1_1.png)

单线程环境确保程序最终执行结果和代码顺序执行的结果一致。  
处理器进行重排序时，须考虑指令间的**数据依赖性**  
多线程环境中线程交替执行，由于编译器优化重排存在，两个线程中使用的变量能否一致是无法确定的，结果无法预测。

内存屏障（Memory Barrier），又称内存栅栏，是一个CPU指令。**通过插入内存屏障禁止在内存屏障前后的指令执行重排序优化**。另一个作用：强制刷出各种CPU的缓存数据，因些任何CPU上的线程都能读取到这些数据的最新版本。

* 对volatile变量进行写操作，**加入一条store屏障指令**，将工作内存中的共享变量值刷新回到主内存；
* 对volatile变量进行读操作，**加入一条load屏障指令**，从主内存中读取共享变量


# jvm  java虚拟机

# JMM(java Memory Model) java内存模型
JMM本身是一种抽象的概念，并不真实存在。它描述的是一级规则或规范。通过这组规范定义了程序中各个变量(包括实例字段、静态字段和构成数组对象的元素)的访问方式

## JMM关于同步的规定
* 线程解锁前，必须把共享变量的值刷新回**主内存**  
* 线程加锁前，必须读取主内存的最新值到自已的**工作内存**
* 加锁解锁是同一把锁  

### 特点
* **可见性**
* **原小性**
* **有序性**


# DCL(双端检锁机制)

## 单例模式下的 volatile分析

DCL(双端检锁)机制不一定线程安全，原因是有指令重排序的存在，加入volatile可以禁上指令重排

```java
public class SingletonDemo {

    private static SingletonDemo instance = null;

    private SingletonDemo() {
        System.out.println(Thread.currentThread().getName() + "\t 我是构造方法，我怕谁");
    }

    //DCL(Double check Lock) 双端检锁机制
    public static SingletonDemo getInstance() {
        if (instance == null) {
            synchronized (SingletonDemo.class) {
                if (instance == null) {
                    instance = new SingletonDemo();
                }
            }
        }

        return instance;
    }

    //单线程下的单例模式
    public static SingletonDemo getInstance() {
        if (instance == null) {
            instance = new SingletonDemo();
        }

        return instance;
    }

    public static void main(String[] args) {
        for(int i = 1; i <= 100; i++) {
          new Thread(() -> {
              SingletonDemo.getInstance();
          }, String.valueOf(i)).start();
        }
    }
}
```

原因在于某一个线程执行到第一次检测，读取到的instance不为null时，instance的引用对象**可能没有完成初始化**  
instance = new SingletonDemo();可以分为以下3步完成（伪代码）

* memory = allocate(); //1、分配对象内存空间
* instance(memory); //2、初始化对象
* instance = memory; //3、设置instance指向刚分配的内存地址，此时instance!=null

其中2和3**不存在数据依赖关系**，无论重排前还是重排后程序执行结果在**单线程**中并没有改变。因此这种重排优化是允许的。

* memory = allocate(); //1、分配对象内存空间
* instance = memory; //**3**、设置instance指向刚分配的内存地址，此时instance!=null。但是对象还没有初始化完成
* instance(memory); //**2**、初始化对象

<font color="red">**所有当一条线程访问instance不为null时，由于instance实例未必已初始化完成，也就造成了线程安全问题。**</font>

### 解决方案

在instance定义处，加上volatile关键字，禁止指令重排。

```java
private static volatile SingletonDemo instance = null;
```

