# 简介
这个生成随机数的工具类是通过ThreadLocalRandom里面的方式类生成的，方便程序调用，ThreadLocalRandom是线程安全的，每个线程有自己独有的随机种子，不像传统的Random存在阻塞的情况.

```java

package com.test.core.utils;

import java.util.concurrent.ThreadLocalRandom;


public class MyRandom{
	
	
	/**
	 * 获取两个整数之间的随机值 min 闭  max 开
	 * @param min
	 * @param max
	 * @return
	 */
	public static int getMyRandomInt(int min,int max){
		if (min == max){
			return max;
		}else if (min > max){
			int tmp = max;
			max = min;
			min = tmp;
		}
		
		return ThreadLocalRandom.current().nextInt(min, max);
	}
	
	
	/**
	 * 获取两个long 之间的随机值 min 闭  max 开
	 * @param min
	 * @param max
	 * @return
	 */
	public static long getMyRandomLong(long min,long max){
		if (min == max){
			return max;
		}else if (min > max){
			long tmp = max;
			max = min;
			min = tmp;
		}
		
		return ThreadLocalRandom.current().nextLong(min, max);
	}
	
	
	/**
	 * 产生两个double 之间的随机数 min 闭  max 开
	 * @param min
	 * @param max
	 * @return
	 */
	public static double getMyRandomFloat(double min,double max){
		if (min>= max && min <= max){
			return max;
		}else if (min > max){
			double tmp = max;
			max = min;
			min = tmp;
		}
		
		return ThreadLocalRandom.current().nextDouble(min, max);
	}
	

	
	public static void main(String[] args) {
//		System.out.println(getMyRandomFloat(0,3));
//		long startTime = System.currentTimeMillis();
		for(int i = 0; i < 1000000;i++) {
			System.out.println(getMyRandomInt(0, 3));
		}
//		
//		System.out.println("用时:" + (System.currentTimeMillis() - startTime)/1000);	
	}
}


```