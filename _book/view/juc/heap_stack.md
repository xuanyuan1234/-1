# 堆栈


# 变量传值

```java
public class TestTransferValue {

    public void changeValue1(int age) {
        age = 30;
    }

    public void changeValue2(Person person) {
        person.setPersonName("xxx");
    }

    public void changeValue3(String str) {
        str = "xxx";
    }

    public static void main(String[] args) {
        TestTransferValue test = new TestTransferValue();
        int age = 20;
        test.changeValue1(age);
        System.out.println("age---" +age);   ---  age---20

        Person person = new Person("abc");
        test.changeValue2(person);
        System.out.println("personName---" + person.getPersonName());   personName---xxx

        String str = "abc";
        test.changeValue3(str);
        System.out.println("string---" + str);   string---abc
    }
}
```

![指令重排](/assert/view/juc/heap_stack/1_1.png)
