# 倒计时器

CountDownLatch用给定的计数初始化。 await方法阻塞，直到由于countDown()方法的调用而导致当前计数达到零。

# CountDownLatch

在java.util.concurrent.CountDownLatch中定义

```java
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(6);

        for (int i = 1; i <= 6; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "\t 离开教室");
                countDownLatch.countDown();
            }, String.valueOf(i)).start();
        }

        System.out.println("当前还剩人数：" + countDownLatch.getCount());
        countDownLatch.await();

        System.out.println(Thread.currentThread().getName() + "\t 班长离开教室");
    }
}
```

# 枚举

```java
public enum CountryEnum {
    ONE(1, "齐"), TWO(2, "楚"), THREE(3, "韩"), FOUR(4, "魏"), FIVE(5, "赵"), SIX(6, "燕");

    private Integer retCode;
    private String retMessage;

    public Integer getRetCode() {
        return retCode;
    }

    public String getRetMessage() {
        return retMessage;
    }

    CountryEnum(Integer retCode, String retMessage) {
        this.retCode = retCode;
        this.retMessage = retMessage;
    }

    public static String forEach_CountryEnum(int index) {
        CountryEnum[] values = CountryEnum.values();
        for (CountryEnum element : values) {
            if (index == element.getRetCode()) {
                return element.getRetMessage();
            }
        }
        return null;
    }
}
```

```java
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(6);

        for (int i = 1; i <= 6; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "\t 国，被灭");
                countDownLatch.countDown();
            }, CountryEnum.forEach_CountryEnum(i)).start();
        }

        System.out.println("当前还剩人数：" + countDownLatch.getCount());
        countDownLatch.await();

        System.out.println(Thread.currentThread().getName() + "\t 秦帝国，一统华夏");
    }
}
```