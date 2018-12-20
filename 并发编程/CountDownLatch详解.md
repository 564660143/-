# CountDownLatch简介

首先来看下JDK中对这个类的描述:

> 一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。
>
> 用给定的*计数* 初始化 `CountDownLatch`。由于调用了 [`countDown()`](../../../java/util/concurrent/CountDownLatch.html#countDown())  方法，所以在当前计数到达零之前，[`await`](../../../java/util/concurrent/CountDownLatch.html#await())  方法会一直受阻塞。之后，会释放所有等待的线程，[`await`](../../../java/util/concurrent/CountDownLatch.html#await())  的所有后续调用都将立即返回。这种现象只出现一次——计数无法被重置。如果需要重置计数，请考虑使用 [`CyclicBarrier`](../../../java/util/concurrent/CyclicBarrier.html)。    
>
> `CountDownLatch` 是一个通用同步工具，它有很多用途。将计数 1 初始化的  `CountDownLatch` 用作一个简单的开/关锁存器，或入口：在通过调用 [`countDown()`](../../../java/util/concurrent/CountDownLatch.html#countDown())  的线程打开入口前，所有调用 [`await`](../../../java/util/concurrent/CountDownLatch.html#await())  的线程都一直在入口处等待。用 *N* 初始化的 `CountDownLatch` 可以使一个线程在 *N*  个线程完成某项操作之前一直等待，或者使其在某项操作完成 N 次之前一直等待。  
>
> `CountDownLatch` 的一个有用特性是，它不要求调用 `countDown`  方法的线程等到计数到达零时才继续，而在所有线程都能通过之前，它只是阻止任何线程继续通过一个 [`await`](../../../java/util/concurrent/CountDownLatch.html#await())。  

# CountDownLatch API

CountDownLatch类中的主要方法就是countDown方法,await方法

## countDown方法

>public void countDown()递减锁存器的计数，如果计数到达零，则释放所有等待的线程。 
>如果当前计数大于零，则将计数减少。如果新的计数为零，出于线程调度目的，将重新启用所有的等待线程。 
>
>如果当前计数等于零，则不发生任何操作。

## await方法

- public void await()方法

  **JDK中描述:**

  >public void await()
  >           throws InterruptedException
  >使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断。 
  >如果当前计数为零，则此方法立即返回。 
  >
  >如果当前计数大于零，则出于线程调度目的，将禁用当前线程，且在发生以下两种情况之一前，该线程将一直处于休眠状态： 
  >
  >由于调用 countDown() 方法，计数到达零；或者 
  >其他某个线程中断当前线程。 
  >如果当前线程： 
  >
  >在进入此方法时已经设置了该线程的中断状态；或者 
  >在等待时被中断， 
  >则抛出 InterruptedException，并且清除当前线程的已中断状态。 
  >
  >抛出： 
  >InterruptedException - 如果当前线程在等待时被中断

- public boolean await(long timeout, TimeUnit unit)方法

  **JDK中描述:**

  >public boolean await(long timeout,
  >                     TimeUnit unit)
  >              throws InterruptedException使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断或超出了指定的等待时间。 
  >如果当前计数为零，则此方法立刻返回 true 值。 
  >
  >如果当前计数大于零，则出于线程调度目的，将禁用当前线程，且在发生以下三种情况之一前，该线程将一直处于休眠状态： 
  >
  >由于调用 countDown() 方法，计数到达零；或者 
  >其他某个线程中断当前线程；或者 
  >已超出指定的等待时间。 
  >如果计数到达零，则该方法返回 true 值。 
  >
  >如果当前线程： 
  >
  >在进入此方法时已经设置了该线程的中断状态；或者 
  >在等待时被中断， 
  >则抛出 InterruptedException，并且清除当前线程的已中断状态。 
  >如果超出了指定的等待时间，则返回值为 false。如果该时间小于等于零，则此方法根本不会等待。 
  >
  >
  >参数：
  >timeout - 要等待的最长时间
  >unit - timeout 参数的时间单位。 
  >返回：
  >如果计数到达零，则返回 true；如果在计数到达零之前超过了等待时间，则返回 false 
  >抛出： 
  >InterruptedException - 如果当前线程在等待时被中断

  

# CountDownLatch使用样例

- 通过CountDownLatch使用并发对多个数组分别求和,最后对所有数组之和进行汇总

```java
package thread.concurrentutil ;

import java.util.ArrayList ;
import java.util.List ;
import java.util.concurrent.CountDownLatch ;

/**
 * CountDownLatch并发工具类demo
 * 多线程并发求和
 * @author Administrator
 *
 */
public class CountDownLatchDemo {
	
	private int [] nums ;
	
	public CountDownLatchDemo(Integer length) {
		nums = new int [length] ;
	}
	
	/**
	 * 计算每一行的数字之和,存放到nums数组的指定位置
	 * @param line,1,2,3以逗号分割的字符串
	 */
	public void lineSum(String line, int index, CountDownLatch latch) {
		String [] nos = line.split(",") ;
		int total = 0 ;
		for (String num : nos) {
			total += Integer.parseInt(num) ;
		}
		System.out.println(Thread.currentThread().getName() + "线程执行计算" + line + "结果为:" + total) ;
		nums[index] = total ;
		/* 每调用一次CountDownLatch的countDown方法,计数器将会减1,计数器减为0时,
		 * 将唤醒使用CountDownLatch多的await方法进入休眠的线程,进入后续操作 */
		latch.countDown() ;
	}
	
	/**
	 * 汇总求和
	 */
	public void sum() {
		System.out.println("汇总线程开始执行----") ;
		int sum = 0 ;
		for (int i : nums) {
			sum += i ;
		}
		System.out.println("汇总结果为: " + sum) ;
	}
	
	public static void main(String [] args) {
		/************初始化测试数据************/
		List<String> list = new ArrayList<>() ;
		list.add("1,2,3,4,5") ;
		list.add("11,22,34,34,33") ;
		list.add("12,23,34,45,56") ;
		list.add("666,666") ;
		/************初始化测试数据************/
		// 生成一个与数组个数对应的CountDownLatch计数器
		CountDownLatch latch = new CountDownLatch(list.size()) ;
		CountDownLatchDemo demo = new CountDownLatchDemo(list.size()) ;
		// 多线程计算
		for (int i = 0; i < list.size(); i++) {
			final int j = i ;
			new Thread() {
				
				@Override
				public void run() {
					demo.lineSum(list.get(j), j, latch) ;
				}
			}.start() ;
		}
		
		try {
			// 调用await方法使main线程处于等待状态,当CountDownLatch计数为0使,才会重新唤醒main进程
			latch.await() ;
		} catch (InterruptedException e) {
			e.printStackTrace() ;
		}
		// 汇总求和
		demo.sum() ;
	}
	
}
```

**输出结果:**

>Thread-0线程执行计算1,2,3,4,5结果为:15
>Thread-1线程执行计算11,22,34,34,33结果为:134
>Thread-3线程执行计算666,666结果为:1332
>Thread-2线程执行计算12,23,34,45,56结果为:170
>汇总线程开始执行----
>汇总结果为: 1651

# CountDownLatch原理详解

## 实现原理

CountDownLatch基于AQS(AbstractQueuedSynchronizer)实现的,由于锁存器计数是各个线程之间共享的,所以在CountDownLatch中通过Sync辅助类实现了AQS(AbstractQueuedSynchronizer)中的tryAcquireShared方法和tryReleaseShared实现一个共享锁

## JDK源码分析

### 辅助类Sync分析

```java
 	/**
     * Sync辅助类继承AbstractQueuedSynchronizer实现了一个共享锁
     * 重写AQS的tryAcquireShared,tryReleaseShared方法
     */
    private static final class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 4982264981922014374L;

        Sync(int count) {
            // 使用AQS的state属性,记录锁存器技术
            setState(count);
        }

        int getCount() {
            return getState();
        }

        /**
         *
         *
         */
        protected int tryAcquireShared(int acquires) {
            return (getState() == 0) ? 1 : -1;
        }
		
        /**
        * 等同于Lock类中的unlock方法,CountDownLatch的countDown方法会调用这个方法
        * 调用这个方法实现锁存器计数减一
        * 
        */
        protected boolean tryReleaseShared(int releases) {
            // 死循环保证最终这个状态值能设置成功
            for (;;) {
                // 获取当前状态值,CountDownLatch中的锁存器计数
                int c = getState();
                // 如果状态为0,表示CountDownLatch中的锁存器计数为0,就直接返回
                if (c == 0)
                    return false;
                // 如果状态不为0,则将状态减一
                int nextc = c-1;
                /*
                 * 更新state值,即锁存器计数值,通过CAS保证线程安全
                 * CAS操作有3个操作数，内存值M，预期值E，新值U，如果M==E，则将内存值修改为B，否则啥都不做
                 * compareAndSetState(c, nextc)方法:
                 * 		c:表示预期值
                 *       nextc:要更新的值
                 *   在此处,表示c==getState()时返回true,并更新state值为nextc
                 *   如果c!=getState()时,表示已经有其他线程更新了state值,
                 *   所以这里不进行更新,直接返回false,通过死循环再重新获取最新的getState()值
                 */
                if (compareAndSetState(c, nextc))
                    // 返回状态是否为0判断,为0表示锁存器计数为0,可以唤醒await的进程了
                    // 这里唤醒进程的操作也是通过AQS进行实现的
                    return nextc == 0;
            }
        }
    }

```





### 构造函数

```java
    /**
     * Constructs a {@code CountDownLatch} initialized with the given count.
     *
     * @param count the number of times {@link #countDown} must be invoked
     *        before threads can pass through {@link #await}
     * @throws IllegalArgumentException if {@code count} is negative
     */
    public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        // 实现一个共享锁
        this.sync = new Sync(count);
    }
```

### countDown方法

    /**
     * 通过调研sync的releaseShared方法将锁存器计数减一
     */
    public void countDown() {
        sync.releaseShared(1);
    }
### await方法

```java
    /**
     * 通过AQS的acquireSharedInterruptibly方法实现await
     */
    public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
```

