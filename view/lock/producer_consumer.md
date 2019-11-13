# 生产者+消费者


# 生产者+消费者

volatile(可见性)/CAS/atomicInteger/BlockQueue/线程交互/原子引用

```java
class Resource {
    private volatile boolean FLAG = true; //默认开启生产+消费
    private AtomicInteger atomicInteger = new AtomicInteger();

    BlockingQueue<String> blockingQueue = null;

    public Resource(BlockingQueue<String> blockingQueue) {  //永远记住，传参 传接口
        this.blockingQueue = blockingQueue;
        System.out.println(blockingQueue.getClass().getName());
    }

    public void myProd() {
        String data = null;
        boolean retval;
        while (FLAG) {
            data = atomicInteger.incrementAndGet() + "";
            try {
                retval = blockingQueue.offer(data, 2L, TimeUnit.SECONDS);
                if (retval) {
                    System.out.println(Thread.currentThread().getName() + "\t 生产数据" + data);
                } else {
                    System.out.println("boss叫停，生产暂停");
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public void myConsumer() {
        String data = null;
        while (FLAG) {
            try {
                data = blockingQueue.poll(2L, TimeUnit.SECONDS);
                if (data == null || "".equalsIgnoreCase(data)) {
                    return;
                }
                System.out.println(Thread.currentThread().getName() + "\t 消费数据" +data);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public void myStop() {
        this.FLAG = false;
    }
}

/**
 * volatile/CAS/atomicInteger/BlockQueue/线程交互/原子引用
 */
public class ProdConsumer_BlockQueueDemo {
    public static void main(String[] args) {
        Resource resource = new Resource(new ArrayBlockingQueue<>(10));
        new Thread(() -> {
            resource.myProd();
        }, "A").start();

        new Thread(() -> {
            resource.myConsumer();
        }, "B").start();

        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        resource.myStop();
    }
}
```
