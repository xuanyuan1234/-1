# 信号量

* 用于多个共享资源的互斥使用
* 用于并发线程数的控制

# Semaphore

```java
public class SemaphoreDemo {
    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);

        for(int i = 1; i <= 6; i++) {
          new Thread(() -> {
              try {
                  semaphore.acquire();
                  System.out.println(Thread.currentThread().getName() + "\t 抢到车位");
                  try {
                      TimeUnit.SECONDS.sleep(3);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
                  System.out.println(Thread.currentThread().getName() + "\t 停车3秒后离开车位");
              } catch (InterruptedException e) {
                  e.printStackTrace();
              } finally {
                  semaphore.release();
              }

          }, String.valueOf(i)).start();
        }
    }
}
```
