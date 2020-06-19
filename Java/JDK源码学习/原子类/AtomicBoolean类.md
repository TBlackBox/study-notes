# 简介
AtomicBoolean是Java.util.concurrent.atomic包下的原子变量。主要用于比较和赋值需要一个原子操作的地方 。

# 例子
通过例子说明，AtomicBoolean的部分作用

1. 单线程执行下面的代码，不会有问题
```
public class NormalBoolean implements Runnable{

	private static boolean exit = false;
	
	@Override
	public void run() {
			
		if(!exit) {
			
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			
			System.out.println(Thread.currentThread().getName() + "进入 if1;");
			
			exit = true;
			
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			
			System.out.println(Thread.currentThread().getName() + "进入 if2;");
			
			exit = false;
			
			System.out.println(Thread.currentThread().getName() + "进入 if3;");
		}else {
			System.out.println(Thread.currentThread().getName() + "进入 else;");
		}
	}
	
	public static void main(String[] args) {
		Thread.currentThread().setName("主线程");
		new NormalBoolean().run();
	}
}
```
输出
```
主线程进入 if1;
主线程进入 if2;
主线程进入 if3;
```
2. 加入我们用2个线程执行
```
public static void main(String[] args) {
		ExecutorService ExecutorService = Executors.newFixedThreadPool(2);
		
		ExecutorService.submit(new NormalBoolean());
		ExecutorService.submit(new NormalBoolean());
		
		ExecutorService.shutdown();
	}
```
输出：
```
pool-1-thread-1进入 if1;
pool-1-thread-2进入 if1;
pool-1-thread-1进入 if2;
pool-1-thread-1进入 if3;
pool-1-thread-2进入 if2;
pool-1-thread-2进入 if3;
```
总结：结果很明显，我们想要的效果是if里面只要一个线程执行，但两个线程都进入了，我们可以通过AmoticBoolean来解决。

3. `AtomicBoolean`处理问题
```
public class AtomicBooleanTest implements Runnable{

	private static AtomicBoolean exit = new AtomicBoolean(false);
	
	@Override
	public void run() {
			
		if(exit.compareAndSet(false, true)) {
			
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			
			System.out.println(Thread.currentThread().getName() + "进入 if1;");
			
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			
			System.out.println(Thread.currentThread().getName() + "进入 if2;");
			
			exit.set(false);
		}else {
			System.out.println(Thread.currentThread().getName() + "进入 else;");
		}
	}
	
	public static void main(String[] args) {
		
		ExecutorService ExecutorService = Executors.newFixedThreadPool(2);
		
		ExecutorService.submit(new AtomicBooleanTest());
		ExecutorService.submit(new AtomicBooleanTest());
		ExecutorService.shutdown();
	}
}
```

输出：
```
pool-1-thread-2进入 else;
pool-1-thread-1进入 if1;
pool-1-thread-1进入 if2;

```
这样就能保证只有一个线程执行if里面的信息，可以看出仅仅一个线程进行工作，因为exist.compareAndSet(false, true)提供了原子性操作，比较和赋值操作组成了一个原子操作, 中间不会提供可乘之机。

# 总结
AtomicBoolean里面的方法来保证原子操作，灵活使用可以解决很多多线程标识问题。