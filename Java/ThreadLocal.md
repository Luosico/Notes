# ThreadLocal



## 作用和目的

在多线程下对于临界区的访问，通常会通过加锁来保证并发的安全性。

但无论是乐观锁还是悲观锁，在并发冲突的时候都会对性能造成影响。

ThreadLocal 是一种 “**空间换时间**” 的方式，相当于为每个线程创建一个局部变量，这样自然就不会出现竞争资源的情况



## 使用

```java
public static void main(String[] args) throws InterruptedException {
    // lambda表达式 Supplier 初始化
    ThreadLocal<String> stringThreadLocal = ThreadLocal.withInitial(() -> "init");

    // 匿名内部类方式处理化
    ThreadLocal<SimpleDateFormat> formatThreadLocal = new ThreadLocal<>(){
        @Override
        protected SimpleDateFormat initialValue() {
            return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        }
    };

    Thread t1 = new Thread(()->{
        stringThreadLocal.set("1");
        stringThreadLocal.get();
        Date now = new Date();
        System.out.println(formatThreadLocal.get().format(now));
    });

    Thread t2 = new Thread(()->{
        stringThreadLocal.set("2");
    });

    t1.setName("子线程1");
    t1.start();
    t2.setName("子线程2");
    t2.start();

    Thread.sleep(1000);
}
```



## 原理

**一个线程对应一个 ThreadLocalMap 对象，一个线程可拥有多个 ThreadLocal 对象，这些 ThreadLocal 对象和 value 存在 ThreadLocalMap.Entry 数组中**

### 初始化

可通过设置初始化方式，为每个线程的 Entry.value 设置初始值

```java
	// lambda表达式 Supplier 初始化
    ThreadLocal<String> stringThreadLocal = ThreadLocal.withInitial(() -> "init");

    // 匿名内部类方式处理化
    ThreadLocal<SimpleDateFormat> formatThreadLocal = new ThreadLocal<>(){
        @Override
        protected SimpleDateFormat initialValue() {
            return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        }
    };
```

设置初始值并不是创建对象时进行，而是在调用 get 方法时，获取不到值才设置



### 获取值

```java
/** ThreadLocal **/
public T get() {
    	// 获取当前 Thread
        Thread t = Thread.currentThread();
    	// 获取当前 Thead 的 TheadLocalMap
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
    	// 返回初始值
        return setInitialValue();
    }
```





### 设置值

```java
public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            map.set(this, value);
        } else {
            createMap(t, value);
        }
    }
```

最终将值存在 Entry.value 中，并将该 Entry作为 Entry数组的一个元素

在寻找在数组中的下标位置时，采用hash来获取，若该下标被占用，就判断下一个位置，直到找到未被占用的位置

![image-20210905133103857](https://raw.githubusercontent.com/Luosico/Typora/main/img/20210905133108.png)



## 内存泄露问题

虽然ThreadLocalMap中的key是**弱引用**，当不存在外部强引用的时候，就会自动被回收，但是Entry中的value依然是强引用

这个value的引用链条如下：

![image-20210905122819087](https://raw.githubusercontent.com/Luosico/Typora/main/img/20210905122842.png)

由于 Entry数组属于线程的变量，存在在线程的整个生命周期，所以**只有在 Thread 结束被回收时，value 才有被回收的机会**。



ThreadLocalMap 检测到 弱引用的 ThreadLocal 被回收后，会调用 expungeStaleEntry() 方法，来释放 value

这个方法不仅仅只释放该 ThreadLocal 对应的value，会继续遍历 Entry 数组，直到获取到的 Entry==null

获取的 Entry 的弱引用 key 为 null, 则释放强引用 value，并将该 Entry[i] 置为null

```java
	private int expungeStaleEntry(int staleSlot) {
            Entry[] tab = table;
            int len = tab.length;

            // expunge entry at staleSlot
            tab[staleSlot].value = null;
            tab[staleSlot] = null;
            size--;

            // Rehash until we encounter null
            Entry e;
            int i;
            for (i = nextIndex(staleSlot, len);
                 (e = tab[i]) != null;
                 i = nextIndex(i, len)) {
                ThreadLocal<?> k = e.get();
                if (k == null) {
                    e.value = null;
                    tab[i] = null;
                    size--;
                } else {
                    int h = k.threadLocalHashCode & (len - 1);
                    if (h != i) {
                        tab[i] = null;

                        // Unlike Knuth 6.4 Algorithm R, we must scan until
                        // null because multiple entries could have been stale.
                        while (tab[h] != null)
                            h = nextIndex(h, len);
                        tab[h] = e;
                    }
                }
            }
            return i;
        }
```



ThreadLocal 的 set()、get()、remove() 都会间接调用到这个方法

```java
	private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
            Entry[] tab = table;
            int len = tab.length;

            while (e != null) {
                ThreadLocal<?> k = e.get();
                if (k == key)
                    return e;
                // 弱引用 key 被回收
                if (k == null)
                    // 回收 key 
                    expungeStaleEntry(i);
                else
                    i = nextIndex(i, len);
                e = tab[i];
            }
            return null;
        }
```

从这里可以看到，ThreadLocal为了避免内存泄露，也算是花了一番大心思。不仅使用了弱引用维护key，还会在每个操作上检查key是否被回收，进而再回收value。

**但是 ThreadLocal 的这种方式并不能保证不会发生内存泄露**

存在一种极端情况，get()方法总是访问固定几个存在的 ThreadLocal，清理动作根本不会执行，没有机会调用set()和remove()，那么这个内存泄漏依然会发生。

**当你需要这个ThreadLocal变量时，主动调用remove()，这样对整个系统是有好处的**





# InheritableThreadLocal

**可以被继承的ThreadLocal**

子线程并不能获取父线程的 ThreadLocal 对象

如果我们希望子线程可以看到父线程的ThreadLocal，那么就可以使用InheritableThreadLocal。顾名思义，这就是一个支持线程间父子继承的ThreadLocal

但是依然要注意以下几点：

1. 变量的传递是发生在线程创建的时候，如果不是新建线程，而是用了线程池里的线程，就不灵了
2. 变量的赋值就是从主线程的map复制到子线程，它们的value是同一个对象，如果这个对象本身不是线程安全的，那么就会有线程安全问题




