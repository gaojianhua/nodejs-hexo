title: 并发编程-进阶1
date: 2017-04-13 21:20:31
tags:
- java
- 并发编程
categories: java
---

### CountDownLatch
 
 有A,B,C,D,E 5个线程,等到A,B,C,D线程并发执行计算任务结束后,E线程对计算结果进行汇总,
 可以用CountDownLatch 实现该需求,
 <!-- more -->
 CountDownLatch 是java.util.concurrent并发包下的类,

 ```bash
 // 构造方法如下
 public CountDownLatch(int count) {  };  //参数count为计数值

 //以下3个是比较重要的方法

 //调用await()方法的线程会被挂起，它会等待直到count值为0才继续执行
 public void await() throws InterruptedException { }; 

 //和await()类似，只不过等待一定的时间后count值还没变为0的话就会继续执行
 public boolean await(long timeout, TimeUnit unit) throws InterruptedException { };

 //将count值减1
 public void countDown() { };  

 ```

```bash
package com.concurrent.test;

import java.util.ArrayList;
import java.util.concurrent.Callable;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.FutureTask;

/**
 * 
 * @author robin
 */
public class ExecutorsTest {
	private final int POOL_SIZE = 5;
	private final int TOTAL_THREAD = 6;
	public ExecutorsTest() throws InterruptedException{
		ExecutorService es_pool = Executors.newFixedThreadPool(POOL_SIZE);
		ArrayList<FutureTask<String>> list = new ArrayList<FutureTask<String>>();
		CountDownLatch latch = new CountDownLatch(POOL_SIZE);
		for(int i =1 ;i<TOTAL_THREAD;i++){
			ThreadCall c = new ThreadCall(i,latch);
			FutureTask<String> ft = new FutureTask<String>(c);
			es_pool.submit(ft);
			list.add(ft);
		}	
		//等待其他线程执行
		latch.await();
		System.out.println("结果获取中...");
		for(int i =0;i<list.size();i++){
			try {
				System.out.println(System.currentTimeMillis()+"返回值***="+list.get(i).get());
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			} catch (ExecutionException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		System.out.println("程序结束...");	
		es_pool.shutdown();
	}
	
	public static void main(String[] args) {
		try {
			new ExecutorsTest();
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}
class ThreadCall implements Callable<String>{
	private int id ;
	private CountDownLatch latch ;
	public ThreadCall(int id,CountDownLatch latch){
		this.id = id;
		this.latch = latch;
	}
	public String call() throws Exception {
		System.out.println("线程-"+id+"运行-->"+System.currentTimeMillis());
		Thread.sleep(300);
		System.out.println("线程-"+id+"结束==>"+System.currentTimeMillis());
		//计数减1
		latch.countDown();
		return "返回字符串="+id;
	}

}


//执行结果如下: 
线程-1运行-->1508482512740
线程-2运行-->1508482512742
线程-3运行-->1508482512745
线程-4运行-->1508482512749
线程-5运行-->1508482512752
线程-1结束==>1508482513042
线程-2结束==>1508482513042
线程-5结束==>1508482513052
线程-3结束==>1508482513054
线程-4结束==>1508482513054
结果获取中...
1508482513054返回值***=返回字符串=1
1508482513054返回值***=返回字符串=2
1508482513054返回值***=返回字符串=3
1508482513054返回值***=返回字符串=4
1508482513054返回值***=返回字符串=5
程序结束...

```
 