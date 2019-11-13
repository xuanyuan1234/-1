# 集合类


# ArrayList是线程不安全，举例说明
```java
public class ContainerNotSafeDemo {
    public static void main(String[] args) {
        List<String> list = Collections.synchronizedList(new ArrayList<>());

        for(int i = 1; i <= 30; i++) {
          new Thread(() -> {
            list.add(UUID.randomUUID().toString().substring(0, 8));
            System.out.println(list);
          }, String.valueOf(i)).start();
        }

        /**
         * 1、故障现象
         *    java.util.ConcurrentModificationException
         *
         *
         * 2、导致原因
         *    add方法，没有加锁。不保证原子性
         *
         *
         * 3、解决方案
         *    3.1 new Vector<>()
         *    3.2 Collections.synchronizedList(new ArrayList<>());
         *    3.3 new CopyOnWriteArrayList<>();
         *
         *
         * 4、优化建议（同样的错误不犯第2次）
         *
         * 
         */
    }
}
```

## CopyOnWriteArrayList 写时复制

添加数据时，并不直接待当前容器Object[] array中添加。而是将当前容器复制一份，得到一个新的容器newElements，大小加1。写完成后，再将原容器的引用指向新的容器。

好处：可以并发的读，不需要加锁，因为当前容器不会添加任何元素。体现**读写分离思想**，读和写不同的容器。

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

# HashSet线程不安全，错误原因与ArrayList一样

**底层是 CopyOnWriteArrayList**

```java
public class CopyOnWriteArraySet<E> extends AbstractSet<E>
        implements java.io.Serializable {
    private static final long serialVersionUID = 5457747651344034263L;

    private final CopyOnWriteArrayList<E> al;

    /**
     * Creates an empty set.
     */
    public CopyOnWriteArraySet() {
        al = new CopyOnWriteArrayList<E>();
    }
```


```java
1.1 Collections.synchronizedSet(new HashSet<String>());
1.2 new CopyOnWriteArraySet<>();
```

## HashSet底层原理

* HashSet底层是 HashMap

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    static final long serialVersionUID = -5024744406713321676L;

    private transient HashMap<E,Object> map;

    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();

    /**
     * Constructs a new, empty set; the backing <tt>HashMap</tt> instance has
     * default initial capacity (16) and load factor (0.75).
     */
    public HashSet() {
        map = new HashMap<>();
    }
```

* HashSet底层是HashMap，为啥 add前者一个参数，后者需要传2个参数

```java
// Dummy value to associate with an Object in the backing Map
private static final Object PRESENT = new Object();

public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}

public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

# HashMap线程不安全，原因与上相同

```java
1.1 new ConcurrentHashMap<>();
1.2 Collections.synchronizedMap(new HashMap<>());
```