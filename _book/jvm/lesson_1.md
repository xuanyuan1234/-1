# 学习1


## 类加载器

类的加载指的是将类的.**class文件**中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在内存中创建一个java.lang.Class对象（规范并未说明Class对象位于哪里，HotSpot虚拟机将其放在了方法区中）用来封装类在方法区内的数据结构。

在Java代码中，类型的加载、连接与初始化过程都是在程序运行期间完成的。

加载：查找并加载类的二进制数据
连接：
* 验证： 确保被加载的类的正确性
* 准备： 为类的**静态变量**分配内存，并将其初始化为**默认值(如 static int a = 1，那默认值为0)**
* 解析： **把类中的符号引用转换为直接引用**（符号引用：一个类中方法引用了另外的一个类；直接引用：直接通过指针的方法指向目标对象的内存位置）  
**初始化：为类的静态变量赋予正确的初始值**


![类加载器](/assert/jvm/lesson_1/1_1.png)


### .class加载方式

* 从本地系统中直接加载
* 通用网络下载.class文件
* 从zip、jar等归档文件中加载.class文件
* 从专用数据库中提取.class文件
* 将Java源文件动态编译为.class文件（类似动态代理）


## 类的使用方式

Java程序对类的使用方法可分为两种：
* 主动使用
* 被动使用 （不初始化）

所有的Java虚拟机实现必须在每个类或接口被Java程序 **首次主动使用** 时才初始化化们。

### 主动使用（七种）

* 创建类的实例
* 访问某个类或接口的**静态变量**，或者对该静态变量赋值 （助记符getstatic、putstatic）
* 调用类的静态方法（invokestatic）
* 反射（如Class.forName("com.test.Test")）
* 初始化一个类的子类 （两个类：parent、child。child初始化是对parent的主动使用）
* Java虚拟机启动时被标明为启动类的类(Java Test、main)
* JDK1.7开始提供的动态语言支持：java.lang.invoke.MethodHandle实例的解析结果REF_getStatic、REF_putStatic、REF_invokeStatic句柄对应的类没有初始化，则初始化

除了以上七种情况，其他使用java类的方式都被看作是对类的**被动使用**，都不会导致类的**初始化**。


#### 示例1

```java
class MyParent {
    public static String str = "hello world";

    static {
        System.out.println("MyParent static block");
    }
}

class MyChild extends MyParent {
    public static String str2 = "welcome";

    static {
        System.out.println("MyChild static block");
    }
}

public class ClassLoaderDemo {
    public static void main(String[] args) {
        System.out.println(MyChild.str);
        System.out.println(MyChild.str2);
    }
}
```

* MyChild.str 返回如下
原因：对于静态字段来说，只有<font color="red">**直接定义了该字段**</font>的类才会被初始化
```java
MyParent static block
hello world
```

* MyChild.str2 返回如下
原因：对应**主动使用的第5条**，当一个类在初始化时，要求其父类全部都已经初始化完毕了
```java
MyParent static block
MyChild static block
welcome
```

* -XX:+TraceClassLoading，用于追踪类的加载信息并打印出来
```java
-XX:+<option>，表示开启option选项
-XX:-<option>，表示关闭option选项
-XX:<option>=<value>，表示将option选项的值设置为value
```


#### 示例2

```java
//本质上，调用类并没有直接引用到定义常量的类，因此并不会触发
//定义常量的类的初始化
//注意：这里指的是将常量存放到了ClassLoaderDemo的常量池中，之后ClassLoaderDemo与MyParent2
//就没有任何关系了。甚至，我们可以将MyParent2的class文件删除。
class MyParent2 {
    public static final String str = "hello world";

    static {
        System.out.println("MyParent2 static block");
    }
}

public class ClassLoaderDemo {
    public static void main(String[] args) {
        System.out.println(MyParent2.str);
    }
}
```

输出结果：
```java
hello world
```

原因：加了**final关键字**。在编译阶段，这个常量就会被存入到调用这个常量的方法所在的类的常量池当中。


#### 示例3

```java
/*
    当一个常量的值并非编译期间可以确定的，那么其值就不会被放到调用类的常量池中。
    这时在程序运行时，会导致主动使用这个常量所在的类，显然会导致这个类被初始化。
*/
public class MyTest3 {
    public static void main(String[] args) {
        System.out.println(MyParent3.str);
    }
}


//UUID在编译时，无法知道具体值。只能在运行时才会知道。
class MyParent3 {
    public static final String str = UUID.randomUUID().toString(); 

    static {
        System.out.println("MyParent3 static code");
    }
}
```

输出结果 (MyParent3类会初始化)
```java
MyParent3 static code
7bddd3a0-0cca-45e2-9dff-3ee6703aa64a
```


#### 示例4

```java
/**
 * 对于数组实例来说，其类型是由JVM在运行期动态生成的，表示为[Lcom.zhou.java.jvm.MyParent4
 * 这种形式。动态生成的类型，其父类型就是Object。
 *
 * 对于数组来说，JavaDoc经常将构成数组的元素为Component，实际上就是将数组降低一个维度后的类型。
 */
public class MyTest4 {
    public static void main(String[] args) {
        //MyParent4 myParent4 = new MyParent4();

        MyParent4[] myParent4s = new MyParent4[1];
        System.out.println(myParent4s.getClass());

        MyParent4[][] myParent4s1 = new MyParent4[1][1];
        System.out.println(myParent4s1.getClass());

        System.out.println(myParent4s.getClass().getSuperclass());
        System.out.println(myParent4s1.getClass().getSuperclass());
    }
}

class MyParent4 {
    static {
        System.out.println("MyParent4 init");
    }
}
```


#### 示例5

```java
/**
 * 当一个接口在初始化时，并不要求其父接口都完成了初始化
 * 只有在真正使用到父接口的时候（如引用接口中所定义的常量时），才会初始化
 */
public class MyTest5 {

    public static void main(String[] args) {
        System.out.println(MyChild5.b);
    }
}

interface MyParent5 {
    public static int a = 5;
}

interface MyChild5 extends MyParent5 {
    public static final int b = 6; //放置在常量池中，与MyChild5本身没有关系
    public static final int c = new Random().nextInt(); //运行期生成
}
```


#### 示例6

```java
public class MyTest6 {

    public static void main(String[] args) {
        Singleton singleton = Singleton.getInstance();

        System.out.println("counter1:" + Singleton.counter1);
        System.out.println("counter2:" + Singleton.counter2);
    }
}

/**
 * main中调用了类的静态方法，导致主动使用。会执行 准备、初始化的操作
 *
 * 1、准备：给各变量赋默认值，如counter1为0，singleton为null，counter2为0
 * 2、getInstance（）后，进行初始化操作，给counter1赋0，singleton调用构造方法，使得
 * counter1、counter2都为1。接下来，counter2赋0。
 *
 * 最终的结果为：counter1：1，counter2：0。
 */
class Singleton {
    public static int counter1;

    //public static int counter2 = 0;

    private static Singleton singleton = new Singleton();

    private Singleton() {
        counter1++;
        counter2++; //准备阶段的重要意义。counter2在后面初始化，在准备阶段赋值。程序不会报错，可正常执行。

        System.out.println("temp : " + counter2);
    }

    public static int counter2 = 0;  //将counter2的定义，从上面移到了此处

    public static Singleton getInstance() {
        return singleton;
    }
}
```