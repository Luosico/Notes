#  Runable、Callable、Future、FutureTask



## Runable

创建线程的两种方式：继承Thread类、实现Runnable接口，都是实现Runnable接口

```java
public interface Runnable {
    public abstract void run();
}
```

不支持返回值和抛出异常。也就是说无法获取执行结果，如果需要获取执行结果，就必须通过共享变量或使用线程间通信的方式来达到效果。

自从Java 1.5开始，提供了Callable和Future，通过它们可以在任务执行完毕之后得到任务执行结果。



## Callable

```java
public interface Callable<V> {
    V call() throws Exception;
}
```

Callable接口支持返回执行结果

一般是和 `ExecutorService`（线程池 `ThreadPoolExecutor`继承的 `AbstractExecutorService`实现了这个接口） 配合使用

![image-20210524165632290](https://raw.githubusercontent.com/Luosico/Typora/main/img/20210524165646.png)

![image-20210524165807692](https://raw.githubusercontent.com/Luosico/Typora/main/img/20210524165807.png)





## Future

Future表示 **异步** 计算的结果。 提供了一些方法来检查计算是否完成，等待其完成以及检索计算结果。

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

对于具体的Runnable或者Callable任务的执行结果进行取消、查询是否完成、获取结果。必要时可以通过get方法获取执行结果，该方法会阻塞直到任务返回结果。

**可以说，Callable用于产生结果，Future用于获取结果**



## FutureTask

FutureTask实现了 `RunnableFuture` 接口

![image-20210524170614644](https://raw.githubusercontent.com/Luosico/Typora/main/img/20210524170614.png)

提供了两个构造参数

```java
public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        this.callable = callable;
        this.state = NEW;       // ensure visibility of callable
    }

public FutureTask(Runnable runnable, V result) {
    	//这里实际上封装了一下，将Runnable对象封装成Callable对象
    	//当Runnable执行完成，会返回result
        this.callable = Executors.callable(runnable, result);
        this.state = NEW;       // ensure visibility of callable
    }
```



### 使用

FutureTask对象一般交由 `Executor.execute()` 或 `ExecutorService.submit()`，或者他们的实现类 ThreadPoolExecutor，**线程池**