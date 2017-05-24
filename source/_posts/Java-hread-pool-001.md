---
title: Java多线程Thread,Runnable, Callable<>和线程池(一)
date: 2017-05-24 21:34:10
tags: [Java, 线程池]
category: Java
---

这一篇主要关注于我们自己实现和管理多线程，后面会介绍使用线程池实现多线程。
Java里面实现多线程有三种方式,继承 Thead类，或者实现Runnable和Callable<>接口。下面详细介绍一下这三种实现方式。

# 1. Thread实现多线程
使用Thread实现，我们只需要继承Thread类，重写(overwirte)run方法。
```java
class ThreadDemo extends Thread {
	private String threadFlag;
	private long sleepTime = 100;

	public ThreadDemo(String threadFlag) {
		System.out.println("Construct " + threadFlag);
		this.threadFlag = threadFlag;
	}

	public ThreadDemo(String threadFlag, long sleepTime) {
		System.out.println("Construct " + threadFlag);
		this.threadFlag = threadFlag;
		this.sleepTime = sleepTime;
	}
	//重写run方法
	public void run() {
		System.out.println("Running before: " + this.threadFlag);
		try {
			for (int i = 0; i < 6; i++) {
				System.out.println(this.threadFlag + i);
				Thread.sleep(this.sleepTime);
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("Running after:" + this.threadFlag);
	}
}
```
<!-- more -->
ThreadDemo类继承自Thread类,重写了Thread类的run()方法。当ThreadDemo调用start()方法的时候，run()方法会被调用。
下面是测试方法：
```java
public class ConcurrentThread {

	public static void main(String[] args) throws InterruptedException, ExecutionException {

		long start = System.currentTimeMillis();
		Thread tA = new ThreadDemo("Thread-A");
		tA.start();

		Thread tB = new ThreadDemo("Thread-B");
		tB.start();
		long end = System.currentTimeMillis();
		System.out.println("During time: " + (end - start));

	}
}
```
测试方法里面创建了两个线程tA,tB，下面是执行的结果，可以看到tA和tB是没有先后顺序的，是两个独立的线程。**"During Time:2"**是主线程里面的语句，但是它在tA和tB没有执行完成的时候，已经开始执行。
说明这两个线程并没有阻塞主线程。
![测试结果](/images/java_001.jpg)

# 2. Runnable实现多线程
另外一种实现多线程的方法是implement Runnable接口
```java
class RunnableImpl implements Runnable {

	private String threadFlag;
	private long sleepTime = 100;

	public RunnableImpl(String threadFlag) {
		System.out.println("Construct " + threadFlag);
		this.threadFlag = threadFlag;
	}

	public RunnableImpl(String threadFlag, long sleepTime) {
		System.out.println("Construct " + threadFlag);
		this.threadFlag = threadFlag;
		this.sleepTime = sleepTime;
	}

	@Override
	public void run() {
		System.out.println("Running before: " + this.threadFlag);
		try {
			for (int i = 0; i < 6; i++) {
				System.out.println(this.threadFlag + i);
				Thread.sleep(this.sleepTime);
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("Running after:" + this.threadFlag);
	}

}
```
RunnableImpl类实现了Runnable接口，实现了run()方法，是用start的方法去启动改线程。下面是测试方法和结果：
```java
public class ConcurrentThread {

	public static void main(String[] args) throws InterruptedException, ExecutionException {

		long start = System.currentTimeMillis();
		Thread tA = new Thread(new RunnableImpl("Thread-A"));
		tA.start();

		Thread tB = new Thread(new RunnableImpl("Thread-B", 400));
		tB.start();
		long end = System.currentTimeMillis();
		System.out.println("During time: " + (end - start));
	}
}
```
![这里写图片描述](/images/java_002.jpg)

使用RunnableImpl和Thread去创建两个线程tA和tB，从结果中看到tA和tB也是并发执行的，他们是随机的获取CPU的执行时间。**"During time:4"**在两个线程没有执行完成的时候已经输出了。说明这两个线程也没有阻塞主线程。
# 3. Callable<>实现多线程
观察Thread和Runnable实现的线程，你会发现这两种实现，你是无法知道线程的执行状态的，例如：线程是否执行完成了，线程有没有被cancel等等。如果你需要基于线程的状态做些事情，就需要很多而外的工作。为了解决这些问题，Callable<>就应运而生了，它和FutureTask结合可以很好的解决这个问题。FutureTask一般有两个用途。
1.获取异步的结果和取消任务。
2. 高并发的情况下确保任务只会执行一次。
```java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

class CallableDemo implements Callable<String> {

	private String threadFlag;
	private long sleepTime = 400;

	public CallableDemo(String threadFlag) {
		System.out.println("Construct " + threadFlag);
		this.threadFlag = threadFlag;
	}

	public CallableDemo(String threadFlag, long sleepTime) {
		System.out.println("Construct " + threadFlag);
		this.threadFlag = threadFlag;
		this.sleepTime = sleepTime;
	}

	@Override
	public String call() throws Exception {
		System.out.println("Running before: " + this.threadFlag);
		Thread.sleep(this.sleepTime);
		System.out.println("Running after:" + this.threadFlag);
		return this.threadFlag + " success in this time";
	}

}
```
这里CallableImpl实现了Callable接口，实现了里面的call()方法，这个接口是泛型化的接口，接口的类型就是call()方法的返回值类型。下面是测试代码：

```java
public class ConcurrentThread {

	public static void main(String[] args) throws InterruptedException, ExecutionException {

		long start = System.currentTimeMillis();
		FutureTask<String> futureA = new FutureTask<String>(new CallableDemo("Thread-A", 5000));
		Thread tA = new Thread(futureA);
		tA.start();
		System.out.println("Thread is Done: " + futureA.isDone());
		System.out.println(futureA.get());
		System.out.println("Thread is Done: " + futureA.isDone());
		FutureTask<String> futureB = new FutureTask<String>(new CallableDemo("Thread-B", 400));
		Thread tB = new Thread(futureB);
		tB.start();
		long end = System.currentTimeMillis();
		System.out.println("During time: " + (end - start));

	}
}
```
这里也是实现了两个线程tA和tB,和上面Thread和Runnable的不同之处在于，这里使用futureA和futureB来获取两个线程的执行状态。futureA.isDone()。这两需要着重说的是futureA.get().这个方法的意思是：主线程等待tA执行完成。这个方法是会阻塞主线程的运行。下面是测试结果:
![这里写图片描述](/images/java_003.png)
可以看到被标记的就是返回值，这是通过get()方法得到的，主线程会成tA执行完成之后，才会继续向下执行，而tB没有调用futureB.get(),所以tB没有阻塞主线程，和主线程是并发的。可以看到**'During time: 5003'**在tB没有执行完成的时候已经被执行。
# 4. 总结
这篇主要介绍了自己实现多线程的三种方式:

1. extends Thread,重写run()方法
2. implements Runnable， 实现run()方法
3. implements Callable<>，实现call()方法，可以获取线程的执行状态信息，isDone(),isCancelled()

下一篇我们会介绍，线程并发时候的共享资源问题。
