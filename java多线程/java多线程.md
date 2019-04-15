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
Object类中的notify()方法，唤醒在此对象监视器上等待的单个线程。如果所有线程都在此对象上等待，则会选择唤醒其中一个线程。选择是任意性的，并在对实现做出决定时发生。线程通过调用其中一个 wait 方法，在对象的监视器上等待。 直到当前的线程放弃此对象上的锁定，才能继续执行被唤醒的线程。被唤醒的线程将以常规方式与在该对象上主动同步的其他所有线程进行竞争；例如，唤醒的线程在作为锁定此对象的下一个线程方面没有可靠的特权或劣势。类似的方法还有一个notifyAll()，唤醒在此对象监视器上等待的所有线程。

1. wait、notify机制通常用于**等待**机制的实现，调用wait进入等待状态，需要的时候调用notify或notifyAll唤醒等待的线程继续执行；wait会释放对象锁。（wait()、notify()、notifyAll()必须放在synchronized block中）

```java
 public static void waitAndNotifyAll() {
        System.out.println("主线程运行");
        Thread thread = new WaitThread();
        thread.start();
        long startTime = System.currentTimeMillis();
        try {
            synchronized (sLockObject) {
                System.out.println("主线程等待");
                sLockObject.wait();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        long timeMs = System.currentTimeMillis() - startTime;
        System.out.println("等待耗时：" + timeMs);
    }

    static class WaitThread extends Thread {
        @Override
        public void run() {
            super.run();
            System.out.println("子线程正在运行... " + Thread.currentThread().getName());
            synchronized (sLockObject) {
                try {
                    Thread.sleep(3000);
                    sLockObject.notifyAll();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

waitAndNotifyAll函数中sLockObject的wait函数使得主线程进入了等待状态，WaitThread的run函数睡眠了3秒钟后调用了sLockObject的notifyAll函数，此时正在等待的主线程就会被唤醒。
 注：wait方法配合while循环（循环条件控制是否真的唤醒）使用可以避免notify以及notifyAll的错误唤醒，是正确以及推荐的做法。



![img](https:////upload-images.jianshu.io/upload_images/626583-9592436c1f5f6442.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

wait执行日志.png

1. Thread的静态函数sleep()是用于实现线程**睡眠**的，sleep()不会释放对象锁。
2. 上面的部分十分直观，join的作用就不那么直观了；join函数的意思是阻塞当前调用join函数所在的线程，直到接收线程执行完毕之后再继续。

```java
/**
     * join()
     * 阻塞当前调用join函数时所在的线程
     * 直到接收到线程执行完毕之后再继续
     */
    private void initJoin() {
        WaitThread waitThread1 = new WaitThread();
        WaitThread waitThread2 = new WaitThread();

        waitThread1.start();
        System.out.println("启动线程1");
        try {
            // 调用waitThread1线程的join函数，主线程会阻塞到waitThread1执行完成
            waitThread1.join();
            System.out.println("启动线程2");
            waitThread2.start();
            // 调用waitThread2线程的join函数，主线程会阻塞到waitThread2执行完成
            waitThread2.join();
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println("主线程继续执行");
    }
//--------------------
 class WaitThread extends Thread {
        @Override
        public void run() {
            super.run();
            System.out.println("子线程正在运行... " + Thread.currentThread().getName());
            synchronized (sLockObject) {
                try {
                    Thread.sleep(3000);
                    sLockObject.notifyAll();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

上面的逻辑为，启动线程1等待线程1执行完成、启动线程2、等待线程2执行完成、继续执行主线程的代码。

![img](https:////upload-images.jianshu.io/upload_images/626583-d5333c39c5e7d349.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

join执行日志.png

1. yield()作用是主动让出当前线程的执行权，其他线程能否优先执行就看各个线程的状态了。
2. 当一个线程运行时，另一个线程调用该线程的interrupt()方法来中断他，注意interrupt方法只是在目标线程中设置一个标志，表示线程已经中断，需要注意如果只单纯调用interrupt方法，线程是不会中断的，应该这样：

~~~java
public class StopRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("StopRunnable 开始...");
        try {
            Thread.sleep(10000);
            System.out.println("StopRunnable try中..."); 
        } catch (InterruptedException e) {
            e.printStackTrace();
            //处理完中断异常后，返回到run()方法入口
            System.out.println("StopRunnable 强制停止...");
            //如果没有return,线程不会实际被中断，它会继续打印下面的信息 
            return;
        }
        System.out.println("StopRunnable 结束...");
    }
}
2.1 调整线程优先级：Java线程有优先级，优先级高的线程会获得较多的运行机会。

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

2.2 线程睡眠：Thread.sleep(long millis)方法，使线程转到阻塞状态。millis参数设定睡眠的时间，以毫秒为单位。当睡眠结束后，就转为就绪（Runnable）状态。sleep()平台移植性好。

2.3 线程等待：Object类中的wait()方法，导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 唤醒方法。这个两个唤醒方法也是Object类中的方法，行为等价于调用 wait(0) 一样。

2.4 线程让步：Thread.yield() 方法，暂停当前正在执行的线程对象，把执行机会让给相同或者更高优先级的线程。

2.5 线程加入：join()方法，等待其他线程终止。在当前线程中调用另一个线程的join()方法，则当前线程转入阻塞状态，直到另一个进程运行结束，当前线程再由阻塞转为就绪状态。

2.6 线程唤醒：Object类中的notify()方法，唤醒在此对象监视器上等待的单个线程。如果所有线程都在此对象上等待，则会选择唤醒其中一个线程。选择是任意性的，并在对实现做出决定时发生。线程通过调用其中一个 wait 方法，在对象的监视器上等待。 直到当前的线程放弃此对象上的锁定，才能继续执行被唤醒的线程。被唤醒的线程将以常规方式与在该对象上主动同步的其他所有线程进行竞争；例如，唤醒的线程在作为锁定此对象的下一个线程方面没有可靠的特权或劣势。类似的方法还有一个notifyAll()，唤醒在此对象监视器上等待的所有线程。

1. wait、notify机制通常用于**等待**机制的实现，调用wait进入等待状态，需要的时候调用notify或notifyAll唤醒等待的线程继续执行；wait会释放对象锁。（wait()、notify()、notifyAll()必须放在synchronized block中）

```java
 public static void waitAndNotifyAll() {
        System.out.println("主线程运行");
        Thread thread = new WaitThread();
        thread.start();
        long startTime = System.currentTimeMillis();
        try {
            synchronized (sLockObject) {
                System.out.println("主线程等待");
                sLockObject.wait();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        long timeMs = System.currentTimeMillis() - startTime;
        System.out.println("等待耗时：" + timeMs);
    }

    static class WaitThread extends Thread {
        @Override
        public void run() {
            super.run();
            System.out.println("子线程正在运行... " + Thread.currentThread().getName());
            synchronized (sLockObject) {
                try {
                    Thread.sleep(3000);
                    sLockObject.notifyAll();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

waitAndNotifyAll函数中sLockObject的wait函数使得主线程进入了等待状态，WaitThread的run函数睡眠了3秒钟后调用了sLockObject的notifyAll函数，此时正在等待的主线程就会被唤醒。
 注：wait方法配合while循环（循环条件控制是否真的唤醒）使用可以避免notify以及notifyAll的错误唤醒，是正确以及推荐的做法。



![img](https:////upload-images.jianshu.io/upload_images/626583-9592436c1f5f6442.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

wait执行日志.png

1. Thread的静态函数sleep()是用于实现线程**睡眠**的，sleep()不会释放对象锁。
2. 上面的部分十分直观，join的作用就不那么直观了；join函数的意思是阻塞当前调用join函数所在的线程，直到接收线程执行完毕之后再继续。

```java
/**
     * join()
     * 阻塞当前调用join函数时所在的线程
     * 直到接收到线程执行完毕之后再继续
     */
    private void initJoin() {
        WaitThread waitThread1 = new WaitThread();
        WaitThread waitThread2 = new WaitThread();

        waitThread1.start();
        System.out.println("启动线程1");
        try {
            // 调用waitThread1线程的join函数，主线程会阻塞到waitThread1执行完成
            waitThread1.join();
            System.out.println("启动线程2");
            waitThread2.start();
            // 调用waitThread2线程的join函数，主线程会阻塞到waitThread2执行完成
            waitThread2.join();
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println("主线程继续执行");
    }
//--------------------
 class WaitThread extends Thread {
        @Override
        public void run() {
            super.run();
            System.out.println("子线程正在运行... " + Thread.currentThread().getName());
            synchronized (sLockObject) {
                try {
                    Thread.sleep(3000);
                    sLockObject.notifyAll();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

上面的逻辑为，启动线程1等待线程1执行完成、启动线程2、等待线程2执行完成、继续执行主线程的代码。
~~~

## 二、 同步机制

### 1. 线程安全性

#### 1.1 什么是线程安全

- 1.当多线程访问某个类的时候，这个类始终表现出预期中正确的行为，那么这个类就可以被称为线程安全的类。
- 2.线程安全的类中封装了必要的同步机制，客户端在调用的时候不需要进一步采取同步措施
- 3.当一个类既不包含域又不包含任何其他类中域的引用可以称为无状态的类，这种类一定是线程安全的。

#### 1.2 原子性

> 当一个对象被多线程使用了，而我们用一个int计算这个对象被调用了多少次，此时这个对象就是非线程安全。因为int的递增操作并不是原子性的，可能int在一个线程递增了一半，该对象就切换到了另一个线程中运行了，此时该对象就会产生与我们预期不符的行为。

- 1.竞态条件：当一个操作的正确性，取决于多个线程交替执行的时序的时候，这就叫竞态条件。如我们上面举的例子，int的递增的正确性取决于某个线程是否会在另一个线程操作到一半的时候就进行操作。常见的竞态条件就是**先检查后执行**，如某个线程的操作插入到另一个线程的**检查和执行操作**之间。
- 2.延迟初始化中的竞态条件： 
  - 1.延迟初始化是一种**先检查后执行**的竞态条件，如单例在多线程下在用到的时候才初始化：**先检查单例是否为null，在初始化单例**。
  - 2.前面举的int递增的例子是另一种竞态条件：**读取-修改-写入**，只要在某个环节被调度到其他线程，类就会产生与预期不相符的行为。
- 3.复合操作:我们前面举的竞态条件，说白了就是**一系列前后关联的操作**————复合操作，一旦这一系列操作被打断，就是线程不安全。所以我们需要将这一系列操作通过java的机制改成原子操作。

#### 1.3 加锁机制

> 我们都知道java中有原子变量，那么是不是说对于一个类，我们只需要把所有的域都变成原子变量，这个类就是线程安全的呢？**很显然并不是，我们在2中提到了竞态条件，当一系列原子变量的操作是前后关联的，那么这一系列的操作就是竞态条件，此时我们如果不进行处理，那么这个类就不是线程安全的**

- 1.内置锁： 

  - 1.每个对象和Class对象都有一个内置锁，一个锁只有一个线程能够持有。
  - 2.对象锁用于非static方法，Class对象锁用于static方法。
  - 3.线程1进入到synchronized块1中，线程1就持有了synchronized块1中传入的锁，线程1运行出synchronized块1就会释放锁，线程2如果运行到传入同样锁的synchronized块2，就会停下来等待线程1运行出synchronized块1。
  - 4.说白了整个synchronized块就是一个**原子操作**，所以我们可以将竞态条件放入synchronized块中，这样一来该类这一系列操作就变成线程安全的了。

- 2.重入：

  当一个线程多次试图进入同一个锁的synchronized块会出现什么情况呢？

  - 1.如果没有重入的话，那么该线程就将等待自己完成synchronized块中的操作，此时这个线程就会产生死锁（即**该线程永远也出不了synchronized块，而且其他线程试图获取这个锁的时候也会被阻塞停止，整个程序就会崩溃**）
  - 2.重入即当一个线程获取了某个锁之后，在没有释放锁的前提下又获取锁是可行的。可以这样理解当锁无线程获取，内部计数为0，某个线程获取了，计数为1，若该线程重复获取该锁，获取一次计数加一，当该线程跑出一个synchronized块计数减一。

### 2. 死锁

#### 2.1 条件： 

- 1.互斥条件：一个资源每次只能被一个进程使用。
- 2.请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
- 3.不剥夺条件:进程已获得的资源，在末使用完之前，不能强行剥夺。
- 4.循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。

#### 2.2 死锁的避免和诊断

- 1.支持定时锁：使用Lock代替内置锁，可以指定一个获取锁等待的时限
- 2.通过线程转储来分析死锁

#### 2.3 其他活跃性危险

- 1.饥饿：一个线程很长时间访问不到其需要的资源，如对线程优先级设置不对，导致一个线程很长时间获取不到cpu时间片
- 2.活锁

### 3. Synchronized

![20170418221917277](.\20170418221917277.png)

Contention List：竞争队列，所有请求锁的线程首先被放在这个竞争队列中；

Entry List：Contention List中那些有资格成为候选资源的线程被移动到Entry List中；

Wait Set：哪些调用wait方法被阻塞的线程被放置在这里；

OnDeck：任意时刻，最多只有一个线程正在竞争锁资源，该线程被成为OnDeck；

Owner：当前已经获取到所资源的线程被称为Owner；

!Owner：当前释放锁的线程。

**注:**

- synchronized关键字不能继承。
- 对于父类中的 synchronized 修饰方法，子类在覆盖该方法时，默认情况下不是同步的，必须显示的使用 synchronized 关键字修饰才行。
- 在定义接口方法时不能使用synchronized关键字。
- 构造方法不能使用synchronized关键字，但可以使用synchronized代码块来进行同步。
- **所以对象锁和类锁是独立的，互不干扰。**

3.1 线程执行互斥代码过程：

1. 获得同步锁；
2. 清空工作内存；
3. 从主内存拷贝对象副本到工作内存；
4. 执行代码(计算或者输出等)；
5. 刷新主内存数据；
6. 释放同步锁。

所以，synchronized既保证了多线程的并发有序性，又保证了多线程的内存可见性。

#### 3.2 概念

synchronized 是 Java 中的关键字，是利用锁的机制来实现同步的。

锁机制有如下两种特性：

- 互斥性：即在同一时间只允许一个线程持有某个对象锁，通过这种特性来实现多线程中的协调机制，这样在同一时间只有一个线程对需同步的代码块(复合操作)进行访问。互斥性我们也往往称为操作的原子性。
- 可见性：必须确保在锁被释放之前，对共享变量所做的修改，对于随后获得该锁的另一个线程是可见的（即在获得锁时应获得最新共享变量的值），否则另一个线程可能是在本地缓存的某个副本上继续操作从而引起不一致。

#### 3.3 对象锁和类锁

##### 3.3.1  对象锁

在 Java 中，每个对象都会有一个 monitor 对象，这个对象其实就是 Java 对象的锁，通常会被称为“内置锁”或“对象锁”。类的对象可以有多个，所以每个对象有其独立的对象锁，互不干扰。

##### 3.3.2  类锁

在 Java 中，针对每个类也有一个锁，可以称为“类锁”，类锁实际上是通过对象锁实现的，即类的 Class 对象锁。每个类只有一个 Class 对象，所以每个类只有一个类锁。

#### 3.4 synchronized 的用法分类

synchronized 的用法可以从两个维度上面分类：

##### 3.4.1  根据修饰对象分类

synchronized 可以修饰方法和代码块

- 修饰代码块
  - synchronized(this|object) {}
  - synchronized(类.class) {}
- 修饰方法
  - 修饰非静态方法
  - 修饰静态方法

##### 3.4.2 根据获取的锁分类

- 获取对象锁
  - synchronized(this|object) {}
  - 修饰非静态方法
- 获取类锁
  - synchronized(类.class) {}
  - 修饰静态方法

1. 当一个线程访问“某对象”的“synchronized方法”或者“synchronized代码块”时，其他线程对**“该对象”的该“synchronized方法”或者“synchronized代码块”的访问**将被阻塞。
2. 当一个线程访问“某对象”的“synchronized方法”或者“synchronized代码块”时，其他线程仍然**可以访问“该对象”的非同步代码块**。
3. 当一个线程访问“某对象”的“synchronized方法”或者“synchronized代码块”时，其他线程对**“该对象”的其他的“synchronized方法”或者“synchronized代码块”的访问**将被阻塞。

**实例锁** -- 锁在某一个实例对象上。如果该类是单例，那么该锁也具有全局锁的概念。
               实例锁对应的就是synchronized关键字。
**全局锁** -- 该锁针对的是类，无论实例多少个对象，那么线程都共享该锁。
               全局锁对应的就是static synchronized（或者是锁在该类的class或者classloader对象上）。

关于“实例锁”和“全局锁”有一个很形象的例子：

```java
pulbic class Something {
    public synchronized void isSyncA(){}
    public synchronized void isSyncB(){}
    public static synchronized void cSyncA(){}
    public static synchronized void cSyncB(){}
}
```

假设，Something有两个实例x和y。分析下面4组表达式获取的锁的情况。
(01) x.isSyncA()与x.isSyncB()                     不可以同步
(02) x.isSyncA()与y.isSyncA()		     可以同步
(03) x.cSyncA()与y.cSyncB()                       不可以同步
(04) x.isSyncA()与Something.cSyncA()       可以同步
