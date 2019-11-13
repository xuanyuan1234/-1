# Callable接口

* Runnable
* Thread
* Callable<V>

# 与 Runnable区别？

* Runnable没有返回值，Callable有返回值
* Runnable不报异常，Callable抛出异常
* Runnable执行run,Callable执行call

# 实现

```java
//第1种
class MyThread implements Runnable {
    @Override
    public void run() {

    }
}

//第2种
class MyThread2 extends Thread {
    @Override
    public void run() {

    }
}

//第3种
class MyThread3 implements Callable<Integer> {

    @Override
    public Integer call() throws Exception {
        System.out.println(Thread.currentThread().getName() + "\t  come in");
        return 1024;
    }
}


public class CallableDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //传统的java.lang.Thread的入参，只有Runnable。需要找一个继承Runnable、Callable的方法。
        FutureTask<Integer> futureTask = new FutureTask<>(new MyThread3());

        //使用Thread执行
        new Thread(futureTask, "AA").start();

        //BB与AA执行同样的方法，不会去执行。come in打印只有一条。
        new Thread(futureTask, "BB").start();

        int result = 100;

        //get建议放在靠后的位置。当futureTask执行时间长时，会导致阻塞。
        int result02 = futureTask.get();

        System.out.println("返回结果：" + (result + result02));
    }
}
```
