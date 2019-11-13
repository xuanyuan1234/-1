# 读写锁


# 读写锁
* 读-读可共存
* 读-写不可共存
* 写-写不可共存

# ReentrantReadWriteLock

```java
class MyCache {
    private volatile Map<String, Object> map = new HashMap<>();
    private ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();

    public void put(String key, Object value) {
        rwLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t 正在写入：" + key);
            try {
                TimeUnit.MILLISECONDS.sleep(300);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + "\t 写入完成");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            rwLock.writeLock().unlock();
        }
    }

    public void get(String key) {
        rwLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t 正在读取");
            Object result = map.get(key);
            System.out.println(Thread.currentThread().getName() + "\t 读取完成：" + result);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            rwLock.readLock().unlock();
        }
    }
}

public class ReadWriteLock {
    public static void main(String[] args) {
        MyCache myCache = new MyCache();

        for(int i = 1; i <= 5; i++) {
            final int v_count = i;
          new Thread(() -> {
              myCache.put(v_count+"", v_count+"");
          }, String.valueOf(i)).start();
        }

        for(int i = 1; i <= 5; i++) {
            final int v_count = i;
          new Thread(() -> {
            myCache.get(v_count+"");
          }, String.valueOf(i)).start();
        }
    }
}
```
