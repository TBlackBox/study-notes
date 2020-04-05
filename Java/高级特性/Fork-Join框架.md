# 简介
在jdk1.7加入了Fork/Join框架，简单来说就是在必要的情况下，将一个大的任务进行拆分(fork)成若干个小任务（拆到不能再拆时），再将一个个小的任务运算的结果进行join汇总。
```
                         -       任务    -
      -      子任务1  -                         - 子任务2-
子任务11                子任务12    子任务21                  子任务22
-------------------------------------------------------------------
结果11                   结果11     结果21                      结果22
            结果1                                 结果1
                        结果
```
说明：拆任务通过（fork）拆，任务通过递归分配成若干小任务。结果通过(join)进行合并。子任务建并行求值。


# 和传统的线程池的区别
传统线程池：如果cpu有4个核，每个核上分5个线程，如果有两个被阻塞了，另外两个运行完了就会处于空闲状态，这样不会很好的利用cpu的资源。fork/join 框架采用“工作窃取”的模式，将任务添加到一个线程队列中，这个队列是双端队列（队列两端都可以操作数据）中，当另外两个核执行完后，将随机从其他没有执行完的队列末端窃取任务执行，这样都不会处于空闲状态了，更好的利用了cpu的资源。

# Fork/Join框架的实现
计算`1`到`10000000000L`的累加值

1. 继承`RecursiveTask`类，并实现他的`compute`方法。
```
package forkjoin;

import java.util.concurrent.RecursiveTask;

public class ForkJoinTest extends RecursiveTask<Long>{

	private static final long serialVersionUID = -3349417846377101047L;

	private Long start;
	private Long end;
	
	//临界值(也就是说当分到10000，任务都不要在分了，也就是递归的结束条件)
	private static final Long THRESHOLD = 10000L; 
	
	public ForkJoinTest(Long start, Long end) {
		this.start = start;
		this.end = end;
	}

	@Override
	protected Long compute() {
		if(this.end - this.start <= THRESHOLD) {
			//达到条件了  这个就是最小的任务，进行任务逻辑处理
			long sum = 0L;
			for(long i = this.start;i < this.end;i++) {
				sum += i;
			}
			return sum;
		}else {
			//说明还能进行拆分
			long middle = (this.start + this.end)/2;
			//进行递归调用
			ForkJoinTest left = new ForkJoinTest(this.start,middle);
			//拆分  并将线程压入线程队列
			left.fork();
			ForkJoinTest right = new ForkJoinTest(middle+1,this.end);
			right.fork();
			//结果合并
			return left.join() + right.join();
		}
	}
}

```
2. 运行fork/join框架
```
package forkjoin;

import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ForkJoinTask;

public class TestForkJoin {

	public static void main(String[] args) {
		long start = System.currentTimeMillis();
		//需要用到ForkJoinPool线程池类运行线程
		ForkJoinPool pool = new ForkJoinPool();
		ForkJoinTask<Long> task = new ForkJoinTest(0L, 10000000000L);
		
		long sum = pool.invoke(task);
		System.out.println("计算结果："+sum);
		
		long end = System.currentTimeMillis();
		System.out.println("耗费的时间为: " + (end - start)); 
	}
}

```

说明：
1. 可以继承两个类，`RecursiveTask`是有返回值，`RecursiveAction`是没有返回值得。

# 总结 
这里简单说了他的运用，其实也就是递归思想，复杂的任务用它能提高效率，单小任务用它将大大减小效率，不用说也晓得，因为分配线程这些需要系统开销这些。