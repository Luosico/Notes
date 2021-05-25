# Future和CompletableFuture

在Java中CompletableFuture用于异步编程，异步编程是编写非阻塞的代码，运行的任务在一个单独的线程，与主线程隔离，并且会通知主线程它的进度，成功或者失败。

在这种方式中，主线程不会被阻塞，不需要一直等到子线程完成。主线程可以并行的执行其他任务。

使用这种并行方式，可以极大的提高程序的性能。



## Future的局限性

1. 不能手动完成 当你写了一个函数，用于通过一个远程API获取一个电子商务产品最新价格。因为这个 API 太耗时，你把它允许在一个独立的线程中，并且从你的函数中返回一个 Future。现在假设这个API服务宕机了，这时你想通过该产品的最新缓存价格手工完成这个Future 。你会发现无法这样做。
2. Future 的结果在非阻塞的情况下，不能执行更进一步的操作 Future 不会通知你它已经完成了，它提供了一个阻塞的 `get()` 方法通知你结果。你无法给 Future 植入一个回调函数，当 Future 结果可用的时候，用该回调函数自动的调用 Future 的结果。
3. 多个 Future 不能串联在一起组成链式调用 有时候你需要执行一个长时间运行的计算任务，并且当计算任务完成的时候，你需要把它的计算结果发送给另外一个长时间运行的计算任务等等。你会发现你无法使用 Future 创建这样的一个工作流。
4. 不能组合多个 Future 的结果 假设你有10个不同的Future，你想并行的运行，然后在它们运行未完成后运行一些函数。你会发现你也无法使用 Future 这样做。
5. 没有异常处理 Future API 没有任务的异常处理结构居然有如此多的限制，幸好我们有CompletableFuture，你可以使用 CompletableFuture 达到以上所有目的。



**CompletableFuture 实现了 `Future` 和 `CompletionStage`接口，并且提供了许多关于创建，链式调用和组合多个 Future 的便利方法集，而且有广泛的异常处理支持。**



## 使用

#### 常量

```java
//有时候是需要构建一个常量的CompletableFuture
public static <U> CompletableFuture<U> completedFuture(U value);

public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<String> completableFuture = CompletableFuture.completedFuture("hello");

        System.out.println(completableFuture.get());
    	//hello
}
```



#### **无返回结果** 

`CompletableFuture.runAsync(Runnable runnable)`

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {
            try{
                System.out.println("begin sleep");
                TimeUnit.SECONDS.sleep(1);
                System.out.println("end sleep");
            }catch (InterruptedException e){
                throw new IllegalArgumentException();
            }
            System.out.println("thread run success");
        });
        System.out.println("before");
        completableFuture.get();
        System.out.println("after");
}
/**
before
begin sleep
end sleep
thread run success
after
**/

//使用线程池，从线程池获取线程执行
Executor executor = Executors.newFixedThreadPool(10);
CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        throw new IllegalStateException(e);
    }
}, executor);

```



#### **有返回结果** 

 `CompletableFuture.supplyAsync(Supplier supplier)`

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
            try{
                System.out.println("begin sleep");
                TimeUnit.SECONDS.sleep(1);
                System.out.println("end sleep");
            }catch (InterruptedException e){
                throw new IllegalArgumentException();
            }
            return "返回结果";
        });
        System.out.println(completableFuture.get());
}

//使用线程池，从线程池获取线程执行
Executor executor = Executors.newFixedThreadPool(10);
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        throw new IllegalStateException(e);
    }
    return "Result of the asynchronous computation";
}, executor);
```



#### **增加回调函数,返回结果** 

`thenApply`

```java
public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn);
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn);
public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn, Executor executor);
```



```java
CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
            try{
                TimeUnit.SECONDS.sleep(1);
                System.out.println("thread");
            }catch (InterruptedException e){
                throw new IllegalArgumentException();
            }
            return "thread";
}).thenApply((message)->{
      return "回调函数结果" + message;
});
System.out.println(completableFuture.get());
```



#### **增加回调函数，无返回结果**

`thenRun` 不可以访问CompletableFuture结果

`thenAccept` 可以访问CompletableFuture结果

```java
CompletableFuture.SupplyAsync(()-{
    return "hello";
}).thenAccept(message->{
   	System.out.println(message);
})
```



#### 组合两个CompletableFuture

`thenCompose()` **组合两个独立的future**，后一个依赖前一个

**串行执行**

```java
CompletableFuture<User> getUsersDetail(String userId) {
	return CompletableFuture.supplyAsync(() -> {
		UserService.getUserDetails(userId);
	});	
}

CompletableFuture<Double> getCreditRating(User user) {
	return CompletableFuture.supplyAsync(() -> {
		CreditRatingService.getCreditRating(user);
	});
}

//使用thenApply
CompletableFuture<CompletableFuture<Double>> result = getUserDetail(userId)
.thenApply(user -> getCreditRating(user));

//使用thenCompose
CompletableFuture<Double> result = getUserDetail(userId)
.thenCompose(user -> getCreditRating(user));

```

`thenCombine` 当两个的future都完成时执行

**并行执行**

```java
System.out.println("Retrieving weight.");
CompletableFuture<Double> weightInKgFuture = CompletableFuture.supplyAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
       throw new IllegalStateException(e);
    }
    return 65.0;
});

System.out.println("Retrieving height.");
CompletableFuture<Double> heightInCmFuture = CompletableFuture.supplyAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
       throw new IllegalStateException(e);
    }
    return 177.8;
});

System.out.println("Calculating BMI.");
CompletableFuture<Double> combinedFuture = weightInKgFuture
        .thenCombine(heightInCmFuture, (weightInKg, heightInCm) -> {
    Double heightInMeter = heightInCm/100;
    return weightInKg/(heightInMeter*heightInMeter);
});

System.out.println("Your BMI is - " + combinedFuture.get());

//当两个Future都完成的时候，传给thenCombine()的回调函数将被调用。
```



#### 组合多个CompletableFuture

`CompletableFuture.allOf`



[基础篇：异步编程不会？我教你啊！CompletableFuture（JDK1.8） (juejin.cn)](https://juejin.cn/post/6902655550031413262)

