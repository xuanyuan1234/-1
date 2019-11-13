# 助记符


javap -c xxx.class 查看

* ldc: 表示将int、float或者String类型的常量值从常量池中推送至栈顶
```java
0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
3: ldc           #4                  // String hello world
5: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
8: return
```

* bipush：表示将单字节(-128~127)的常量值推送至栈顶
```java
0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
3: bipush        7
5: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
8: return
```

* sipush：表示将一个短整型常量值(-32768~32767)推送至栈顶
```java
0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
3: sipush        128
6: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
9: return
```

* iconst_1：表示将int类型1推送至栈顶 (iconst_m1 - iconst_5)
```java
    0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
    3: iconst_1
    4: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
    7: return
```

* anewarray: 表示创建一个引用类型的（如类、接口、数组）数组，并将其引用值压入栈顶
```java
0: iconst_1
1: anewarray     #2                  // class com/zhou/java/jvm/MyParent4
4: astore_1
```

* newarray: 表示创建一个指定的原始类型（如int、float、char等）的数组，并将其引用值压入栈顶
```java
58: iconst_1
59: newarray       int
```