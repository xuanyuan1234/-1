# 可重用屏障/栅栏

CyclicBarrier支持一个可选的Runnable命令，每个屏障点运行一次，在派对中的最后一个线程到达之后，但在任何线程释放之前。 在任何一方继续进行之前，此屏障操作对更新共享状态很有用。 

# CyclicBarrier 

```java
public class CyclicBarrierDemo {
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> {
            System.out.println("召唤神龙");
        });

        for(int i = 1; i <= 7; i++) {
            final int v_count = i;
          new Thread(() -> {
              System.out.println(Thread.currentThread().getName() + "\t 找到第 " + v_count +  " 颗龙珠");
              try {
                  cyclicBarrier.await();
              } catch (InterruptedException e) {
                  e.printStackTrace();
              } catch (BrokenBarrierException e) {
                  e.printStackTrace();
              }
          }, String.valueOf(i)).start();
        }
    }
}
```