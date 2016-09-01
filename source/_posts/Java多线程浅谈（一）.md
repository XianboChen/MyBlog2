---
title: Java多线程浅谈（一）
date: 2016-07-26 22:41:46
tags:
  - java
---
实现java多线程主要有两种方法：继承Thread类和实现runnable接口。
## 1、继承java.lang.Thread类
1.创建一个继承于Thread的子类；
2.重写Thread类的run（）方法,方法内实现子线程要完成的功能；
3.创建一个子类的对象；
4.调用其start（）方法，启动线程，调用run（）方法。
demo：
``` java
public class TestThread {
	public static void main(String[] args) {
		//3.创建一个子类的对象
		SubThread st1 = new SubThread();
		SubThread st2 = new SubThread();
		//4.调用其start（）方法，启动线程，调用run（）方法
		st1.start();
		st2.start();	
		//st.start();
		for(int i = 1; i <= 100; i++) {
			System.out.println(Thread.currentThread().getName() + ":" + i);
		}
	}
}
//1.创建一个继承于Thread的子类
class SubThread extends Thread {
	//2.重写Thread类的run（）方法,方法内实现子线程要完成的功能
	public void run() {
		for(int i = 1; i <= 100; i++) {
			System.out.println(Thread.currentThread().getName() + ":" + i);
		}
	}
}
```
运行结果：
> main:1
main:2
Thread-1:1
Thread-0:1
Thread-1:2
main:3
Thread-1:3
Thread-0:2
Thread-1:4
main:4
Thread-1:5
Thread-1:6
Thread-0:3
Thread-1:7
main:5
Thread-1:8
Thread-0:4
Thread-1:9
main:6
main:7
Thread-1:10
Thread-0:5
Thread-1:11
main:8
Thread-1:12
Thread-0:6
Thread-1:13
main:9
Thread-1:14
Thread-0:7
Thread-1:15
...（以下省略）

**这里要注意：** 

 - 一个线程只能执行一次start()方法，在java源码中start()方法被关键字synchronized修饰，同时对threadStatus进行了判断，非0抛IllegalThreadStateException()异常；
 - 不能通过Thread实现类对象的run()方法去启动一个线程。
  
### Thread的常用方法：
- start():启动线程并执行相应的run()方法;
- run()：子线程要执行的代码放入run()方法中;
- currentThread():静态的，取到当前的线程；
- getName():取到线程的名字；
- setName():设置此线程的名字
- yield(): 调用此方法的线程释放当前CPU的执行权，重回可执行状态，**但并不意味着CPU接下来不会执行该线程，该线程仍然可能抢到CPU**；
- join():在A线程中调用B线程的join()方法，表示当执行到此方法时，A线程停止执行，直至B线程执行完毕,A线程再接着jion()之后的代码执行；
- isAlive():判断当前线程是否还存活；
- sleep(long l):显式的让当前进程睡眠1毫秒；
- 线程通信：wait() notify() notifyAll()
- 设置进程的优先级：
 - getPriority():返回进程优先值 （0-10），static final 的值 , 10最大
 - setPriority(int newPriority):改变进程的优先级

demo
``` java
public class TestThread {
	public static void main(String[] args) {
		//3.创建一个子类的对象
		SubThread st1 = new SubThread();
		SubThread st2 = new SubThread();
		//4.调用其start（）方法，启动线程，调用run（）方法
		st1.setName("子线程");
		st1.setPriority(Thread.MAX_PRIORITY);
		st1.start();
		Thread.currentThread().setName("========主线程");
		//st.start();
		for(int i = 1; i <= 100; i++) {
			System.out.println(Thread.currentThread().getName() + ":" +
					Thread.currentThread().getPriority() + " " + i);
//			if(i % 10 == 0) {
//				Thread.currentThread().yield();
//			}
//			if(i == 20) {
//				try {
//					st1.join();
//				} catch (InterruptedException e) {
//					// TODO Auto-generated catch block
//					e.printStackTrace();
//				}
//			}
		}
		System.out.println(st1.isAlive());
	}
}

//1.创建一个继承于Thread的子类
class SubThread extends Thread {
	//2.重写Thread类的run（）方法,方法内实现子线程要完成的功能
	public void run() {
		for(int i = 1; i <= 100; i++) {
//			try {
//				Thread.currentThread().sleep(1000);
//			} catch (InterruptedException e) {
//				// TODO Auto-generated catch block
//				e.printStackTrace();
//			}
			System.out.println(Thread.currentThread().getName() + ":" +
					Thread.currentThread().getPriority() + " " + i);
		}
	}
}
```

## 2、实现runnable接口
1.创建一个实现了Runnable借口的类；
2.实现接口的抽象方法；
3.创建一个Runnable接口的实现类；
4.将此对象作为行参传递给Thread类的构造器中，创建Thread的对象，此对象即为一个线程；
5.调用start()方法，启动线程并执行run()。
demo
``` java
public class TsetThread1 {
	public static void main(String[] args) {
		//3.创建一个Runnable接口的实现类
		PrintNum1 p = new PrintNum1();
		//p.run();
		//p.start();
		//要想启动一个多线程，必须调用start()
		//4.将此对象作为行参传递给Thread类的构造器中，创建Thread的对象，此对象即为一个线程
		Thread t1 = new Thread(p);
		//5.调用start()方法，启动线程并执行run()
		t1.start();
		for (int i = 1; i <= 100; i++) {
			if (i % 2 != 0) {
				System.out.println(Thread.currentThread().getName() + ":" + i);
			}
		}
	}
}
//1.创建一个实现了Runnable借口的类
class PrintNum1 implements Runnable {
    //2.实现接口的抽象方法
	@Override
	public void run() {
		// TODO Auto-generated method stub
		for (int i = 1; i <= 100; i++) {
			if (i % 2 ==0) {
				System.out.println(Thread.currentThread().getName() + ":" + i);
			}
		}
	}
}
```
运行结果：
> main:1
Thread-0:2
Thread-0:4
Thread-0:6
Thread-0:8
Thread-0:10
main:3
main:5
main:7
Thread-0:12
main:9
main:11
main:13
main:15
main:17
main:19
main:21
main:23
main:25
main:27
main:29
main:31
main:33
main:35
main:37
...(下略）
**这里要注意：**

- 要想启动一个线程，必须调用start()方法；
- 在java源码中	
         Thread t1 = new Thread(p);
		  t1.start();
 实际上是把P给了源码中target,再调用了target的run方法。

## 两种方式的比较
实现的方式较好：

- 避免了java单继承的局限性
- 如果多个线程要操作同一份资源，更适合采用实现runnable接口的方式。










 

