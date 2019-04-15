# java多线程/并发总结

## 一、多线程

![C67CFFB4-DCDE-44DC-90E7-8B396F926AD3](.\C67CFFB4-DCDE-44DC-90E7-8B396F926AD3.png)

1、新建状态（New）：新创建了一个线程对象。
2、就绪状态（Runnable）：线程对象创建后，其他线程调用了该对象的start()方法。该状态的线程位于可运行线程池中，变得可运行，等待获取CPU的使用权。
3、运行状态（Running）：就绪状态的线程获取了CPU，执行程序代码。
4、阻塞状态（Blocked）：阻塞状态是线程因为某种原因放弃CPU使用权，暂时停止运行。直到线程进入就绪状态，才有机会转到运行状态。阻塞的情况分三种：
（一）、等待阻塞：运行的线程执行wait()方法，JVM会把该线程放入等待池中。(wait会释放持有的锁)
（二）、同步阻塞：运行的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则JVM会把该线程放入锁池中。
（三）、其他阻塞：运行的线程执行sleep()或join()方法，或者发出了I/O请求时，JVM会把该线程置为阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。（注意,sleep是不会释放持有的锁）

5、死亡状态（Dead）：线程执行完了或者因异常退出了run()方法，该线程结束生命周期。

### 1. 创建线程的方法

#### 1.1 继承Thread类创建线程类

```java
/*
 *继承 Thread 类
 *覆盖 run() 方法
 *直接调用 Thread#start() 执行
 */
public class FirstThreadTest extends Thread{

	@Override
	public void run() {
		for(int i = 0; i < 100; i++){
			System.out.println(getName()+"  "+i);

		}

	}

	public static void main(String[] args)
	{
		for (int i = 0;i< 100;i++)
		{
			System.out.println(Thread.currentThread().getName()+"  : "+i);
			if(i==20)
			{
				new FirstThreadTest().start();
				new FirstThreadTest().start();
			}
		}
	}
}

```

#### 1.2 通过Runable接口创建线程类

```java
/*
 *实现Runnable接口
 *获取实现Runnable接口的实例，作为参数，创建Thread实例
 *执行 Thread#start() 启动线程public class RunnableThreadTest implements Runnable {
 */
	@Override
	public void run() {
		for (int i = 0; i < 100; i++) {
			System.out.println(Thread.currentThread().getName()+" "+i);
		}
	}

	public static void main (String[] args) {
		for(int i = 0;i < 100;i++) {
			System.out.println(Thread.currentThread().getName()+" "+i);
			if (i == 20) {
				RunnableThreadTest rtt = new RunnableThreadTest();
				new Thread(rtt,"新线程1").start();
				new Thread(rtt,"新线程2").start();
			}
		}
	}
}

```

#### 1.3  通过Callable和FutureTask创建线程

```java
/**
 * a. 创建Callable接口的实现类，并实现call()方法；
 * b. 创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该FutureTask对象封装了该Callback对象的call()方法的返回值；
 *  c. 使用FutureTask对象作为Thread对象的target创建并启动新线程；
 *  d. 调用FutureTask对象的get()方法来获得子线程执行结束后的返回值。
 */
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class CallableThreadTest implements Callable<String> {
	public static void main(String[] args)
	{
		CallableThreadTest ctt = new CallableThreadTest();
		FutureTask<String> ft = new FutureTask<String>(ctt);
//        Thread thread = new Thread(ft,"有返回值的线程");
//        thread.start();
		for (int i = 0; i < 100; i++) {
			System.out.println(Thread.currentThread().getName()+" 的循环变量i的值"+i);
			if (i == 20) {
				new Thread(ft,"有返回值的线程").start();
			}
		}
		try {
			System.out.println("子线程的返回值："+ft.get());
		} catch (InterruptedException e) {
			e.printStackTrace();
		} catch (ExecutionException e) {
			e.printStackTrace();
		}
	}

	@Override
	public String call() throws Exception {
		int i = 0;
		for(; i < 100; i++) {
			System.out.println(Thread.currentThread().getName()+" "+i);
		}
		return i+"";
	}
}
```

#### 1.4 通过线程池创建线程

```java
/*
 *创建线程池（可以利用JDK的Executors，也可自己实现）
 *创建Callable 或 Runnable任务，提交到线程池
 *通过返回的 Future#get 获取返回的结果
 */	
public class ThreadPool 
{
	/* POOL_NUM */
	private static int POOL_NUM = 10;
	
	/**
	 * Main function
	 */
	public static void main(String[] args)
	{
		ExecutorService executorService = Executors.newFixedThreadPool(5);
		for(int i = 0; i<POOL_NUM; i++)
		{
			RunnableThread thread = new RunnableThread();
			executorService.execute(thread);
		}
	}
}	
 
class RunnableThread implements Runnable
{
	private int THREAD_NUM = 10;
	public void run()
	{
		for(int i = 0; i<THREAD_NUM; i++)
		{
			System.out.println("线程" + Thread.currentThread() + " " + i);
		} 
	}

```

#### 1.5 分类

- 继承Thread类，覆盖run方法填写业务逻辑
- 实现Callable或Runnable接口，然后通过Thread或线程池来启动线程

#### 1.6 Thread和Runnable的区别

如果一个类继承Thread，则不适合资源共享。但是如果实现了Runable接口的话，则很容易的实现资源共享。

总结：

实现Runnable接口比继承Thread类所具有的优势：

1）：适合多个相同的程序代码的线程去处理同一个资源

2）：可以避免java中的单继承的限制

3）：增加程序的健壮性，代码可以被多个线程共享，代码和数据独立

4）：线程池只能放入实现Runable或callable类线程，不能直接放入继承Thread的类

**Runnable**

​	Runnable不关心返回，所以任务自己默默的执行就可以了，也不用告诉我完成没有，我不care，您自己随便玩，所以一般使用就是

```java
new Thread(new Runnable() { public void run() {...} }).start()
```

​	换成JDK8的 lambda表达式就更简单了 `new Thread(() -> {}).start();`

**Callable**

​	相比而言，callbale就悲催一点，没法这么随意了，因为要等待返回的结果，但是这个线程的状态我又控制不了，怎么办？借助`FutrueTask`来玩，所以一般可以看到使用方式如下:

```java
FutureTask<Object> future = new FutureTask<>(() -> null);
new Thread(future).start();
Object obj = future.get(); // 这里会阻塞，直到线程返回值
```

#### 1.7 区别与应用场景

- 继承和实现接口的方式唯一区别就是继承和实现的区别，不存在共享变量的问题
- 需要获取返回结果时，结合 FutureTask和Callable来实现
- Thread和Runnable的两种方式，原则上想怎么用都可以，个人也没啥好推荐的，随意使用
- 线程池需要注意线程复用时，对ThreadLocal中变量的串用问题（本篇没有涉及，等待后续补上）

**注意**

- 利用线程池创建线程，实际上依然是借助的Runnable或者Callable，能否算一种新的方式纯看个人理解
- 采用Timer方式实现定时任务的方式，也是一种新的创建线程的方式，这里也没有多说，后续将有一篇专门说明定时任务的博文介绍其用法

### 2. 线程的调度

#### 2.1 调整线程优先级

Java线程有优先级，优先级高的线程会获得较多的运行机会。

Java线程的优先级用整数表示，取值范围是1~10，Thread类有以下三个静态常量：
static int MAX_PRIORITY
          线程可以具有的最高优先级，取值为10。
static int MIN_PRIORITY
          线程可以具有的最低优先级，取值为1。
static int NORM_PRIORITY
          分配给线程的默认优先级，取值为5。

Thread类的setPriority()和getPriority()方法分别用来设置和获取线程的优先级。
 每个线程都有默认的优先级。主线程的默认优先级为Thread.NORM_PRIORITY。
线程的优先级有继承关系，比如A线程中创建了B线程，那么B将和A具有相同的优先级。
JVM提供了10个线程优先级，但与常见的操作系统都不能很好的映射。如果希望程序能移植到各个操作系统中，应该仅仅使用Thread类有以下三个静态常量作为优先级，这样能保证同样的优先级采用了同样的调度方式。

#### 2.2 线程睡眠

Thread.sleep(long millis)方法，使线程转到阻塞状态。millis参数设定睡眠的时间，以毫秒为单位。当睡眠结束后，就转为就绪（Runnable）状态。sleep()平台移植性好。

#### 2.3 线程等待

Object类中的wait()方法，导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 唤醒方法。这个两个唤醒方法也是Object类中的方法，行为等价于调用 wait(0) 一样。

#### 2.4 线程让步

Thread.yield() 方法，暂停当前正在执行的线程对象，把执行机会让给相同或者更高优先级的线程。

#### 2.5 线程加入

join()方法，等待其他线程终止。在当前线程中调用另一个线程的join()方法，则当前线程转入阻塞状态，直到另一个进程运行结束，当前线程再由阻塞转为就绪状态。

#### 2.6 线程唤醒
