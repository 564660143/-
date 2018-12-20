# CyclicBarrier简介

- 一个同步辅助类，它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。因为该 barrier 在释放等待线程后可以重用，所以称它为循环 的 barrier。 
- CyclicBarrier 支持一个可选的 Runnable 命令，在一组线程中的最后一个线程到达之后（但在释放所有线程之前），该命令只在每个屏障点运行一次。若在继续所有参与线程之前更新共享状态，此屏障操作 很有用。

> 简单来说,CyclicBarrier就是一个同步屏障,通过await方法使线程处于等待状态,当等待的线程达到一定的数量时(这个就是公共屏障点),所有的线程都会被唤醒,这个数量通过构造方法传入.通过reset方法可以重置屏障

# CyclicBarrier API

CyclicBarrier主要方法包括await方法,reset方法,以及两个构造器方法

## 构造方法

> **public CyclicBarrier(int parties):**
>
> ```
> public CyclicBarrier(int parties)
> 创建一个新的 CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，但它不会在启动 barrier 时执行预定义的操作。
> ```

**public CyclicBarrier(int parties,Runnable barrierAction):**

> public CyclicBarrier(int parties,Runnable barrierAction)
> 创建一个新的 CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，并在启动 barrier  时执行给定的屏障操作，该操作由最后一个进入 barrier 的线程执行。 

## await方法

>public int await()
>          throws InterruptedException,
>                 BrokenBarrierException在所有参与者都已经在此 barrier 上调用 await 方法之前，将一直等待。 
>如果当前线程不是将到达的最后一个线程，出于调度目的，将禁用它，且在发生以下情况之一前，该线程将一直处于休眠状态： 
>
>最后一个线程到达；或者 
>其他某个线程中断当前线程；或者 
>其他某个线程中断另一个等待线程；或者 
>其他某个线程在等待 barrier 时超时；或者 
>其他某个线程在此 barrier 上调用 reset()。 
>如果当前线程： 
>
>在进入此方法时已经设置了该线程的中断状态；或者 
>在等待时被中断 
>则抛出 InterruptedException，并且清除当前线程的已中断状态。 
>如果在线程处于等待状态时 barrier 被 reset()，或者在调用 await 时 barrier 被损坏，抑或任意一个线程正处于等待状态，则抛出 BrokenBarrierException 异常。 
>
>如果任何线程在等待时被 中断，则其他所有等待线程都将抛出 BrokenBarrierException 异常，并将 barrier 置于损坏状态。 
>
>如果当前线程是最后一个将要到达的线程，并且构造方法中提供了一个非空的屏障操作，则在允许其他线程继续运行之前，当前线程将运行该操作。如果在执行屏障操作过程中发生异常，则该异常将传播到当前线程中，并将 barrier 置于损坏状态。 
>
>
>返回：
>到达的当前线程的索引，其中，索引 getParties() - 1 指示将到达的第一个线程，零指示最后一个到达的线程 
>抛出： 
>InterruptedException - 如果当前线程在等待时被中断 
>BrokenBarrierException - 如果另一个 线程在当前线程等待时被中断或超时，或者重置了 barrier，或者在调用 await 时 barrier 被损坏，抑或由于异常而导致屏障操作（如果存在）失败。

**出了上面这个await方法,await还存在一个带超时时间的方法,允许设置一个超时时间public int await(long timeout,TimeUnit unit)的方法,表示在所有线程到达之前或超出指定超时时间之前线程一直等待**



##reset方法

public void reset()将屏障重置为其初始状态。如果所有参与者目前都在屏障处等待，则它们将返回，同时抛出一个 BrokenBarrierException。注意，在由于其他原因造成损坏之后，实行重置可能会变得很复杂；此时需要使用其他方式重新同步线程，并选择其中一个线程来执行重置。与为后续使用创建一个新 barrier 相比，这种方法可能更好一些。 

# 使用样例

**1.使用CyclicBarrier模仿倚天屠龙记中六大门派围攻光明顶**

```java
package thread.concurrentutil ;

import java.util.concurrent.CyclicBarrier ;

/**
 * 门派线程,用来表示六大门派
 */
class MenPai implements Runnable {
	
	// 名称
	private String			name ;
	// 到达光明顶所需要的时间
	private long			time ;
	// CyclicBarrier
	private CyclicBarrier	cyclicBarrier ;
	
	public MenPai(String name, long time, CyclicBarrier cyclicBarrier) {
		super() ;
		this.name = name ;
		this.time = time ;
		this.cyclicBarrier = cyclicBarrier ;
	}
	
	@Override
	public void run() {
		System.out.println(name + "出发前往光明顶") ;
		try {
			Thread.sleep(time) ;
			System.out.println(name + "到达光明顶") ;
			cyclicBarrier.await() ;
		} catch (Exception e) {
			e.printStackTrace() ;
		}
		
	}
	
}

/**
 * CyclicBarrier使用demo
 * 通过CyclicBarrier模仿六大门派围攻光明顶
 *
 */
public class CyclicBarrierDemo {
	
	public static void main(String [] args) {
		// 定义CyclicBarrier,传入6表示需要六大门派到期
		CyclicBarrier cyclicBarrier = new CyclicBarrier(6, new Runnable() {
			
			@Override
			public void run() {
				System.out.println("六大门派到期,开始围攻光明顶") ;
			}
		}) ;
		new Thread(new MenPai("少林", 3000, cyclicBarrier)).start() ;
		new Thread(new MenPai("武当", 5000, cyclicBarrier)).start() ;
		new Thread(new MenPai("峨眉", 2000, cyclicBarrier)).start() ;
		new Thread(new MenPai("昆仑", 6000, cyclicBarrier)).start() ;
		new Thread(new MenPai("崆峒", 3000, cyclicBarrier)).start() ;
		new Thread(new MenPai("华山", 3500, cyclicBarrier)).start() ;
		
	}
	
}

```

**运行结果:**

>少林出发前往光明顶
>峨眉出发前往光明顶
>昆仑出发前往光明顶
>武当出发前往光明顶
>华山出发前往光明顶
>崆峒出发前往光明顶
>峨眉到达光明顶
>崆峒到达光明顶
>少林到达光明顶
>华山到达光明顶
>武当到达光明顶
>昆仑到达光明顶
>六大门派到期,开始围攻光明顶

这个样例演示了,CyclicBarrier作为同步屏障的用法,下面通过CyclicBarrier模仿一个棋牌室的打麻将的场景,展示一下CyclicBarrier作为循环屏障的用法

```java
package thread.concurrentutil;

import java.util.Random ;
import java.util.concurrent.CyclicBarrier ;

/**
 * 玩家类,表示打麻将的人
 */
class Player implements Runnable {
	// 姓名
	private String name;
	// 到达麻将馆所需时间
	private long time;
	// 到达麻将馆所需时间
	CyclicBarrier cyclicBarrier;
	
	public Player(String name, long time, CyclicBarrier cyclicBarrier) {
		super() ;
		this.name = name ;
		this.time = time ;
		this.cyclicBarrier = cyclicBarrier ;
	}


	@Override
	public void run() {
		System.out.println(name + "出发前往麻将馆") ;
		try {
			// 休眠一段时间,表示路上所需时间
			Thread.sleep(time);
			// 需要等待集齐四个人
			cyclicBarrier.await();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}


/**
 * CyclicBarrier使用demo
 * 通过CyclicBarrier模仿打麻将的场景,每到期四个人就开一桌麻将
 *
 */
public class CyclicBarrierDemo2 {
	
	public static void main(String [] args) throws Exception {
		// 定义CyclicBarrier,传入5表示加上当前main线程在内,需要有5个线程处于等待状态时,才会唤醒所有线程
		CyclicBarrier cyclicBarrier = new CyclicBarrier(5);
		// 人员编号, 桌子编号
		int playerIndex = 1, deskIndex = 1;
		while(true){
			new Thread(new Player("玩家" + playerIndex++, new Random().nextInt(5000), cyclicBarrier)).start();
			new Thread(new Player("玩家" + playerIndex++, new Random().nextInt(5000), cyclicBarrier)).start();
			new Thread(new Player("玩家" + playerIndex++, new Random().nextInt(5000), cyclicBarrier)).start();
			new Thread(new Player("玩家" + playerIndex++, new Random().nextInt(5000), cyclicBarrier)).start();
			// 在人员到期之前,让主线程处于等待状态
			cyclicBarrier.await();
			System.out.println("人员已到齐,第" +  (deskIndex++) + "桌开始游戏") ;
		}
		
	}
	
}
```

**运行结果:**

>玩家3出发前往麻将馆
>玩家4出发前往麻将馆
>玩家2出发前往麻将馆
>玩家1出发前往麻将馆
>人员已到齐,第1桌开始游戏
>玩家5出发前往麻将馆
>玩家6出发前往麻将馆
>玩家7出发前往麻将馆
>玩家8出发前往麻将馆
>人员已到齐,第2桌开始游戏

# CyclicBarrier源码详解

## CyclicBarrier中属性

```java
  	/** 一个可重入锁,用于保证同一时间只有一个线程能进入await方法 */
    private final ReentrantLock lock = new ReentrantLock();
	
	 /** Condition简单解释:
	  * Condition 将 Object 监视器方法（wait、notify 和 notifyAll）分解成截然不同的对象，
	  * 以便通过将这些对象与任意 Lock 实现组合使用，为每个对象提供多个等待 set（wait-set）。
	  * 其中，Lock 替代了 synchronized 方法和语句的使用，
	  * Condition 替代了 Object 监视器方法的使用。
	  */
    /** 通过使用Condition的await()方法,保证在所有线程到达屏障点之前,其他线程处于等待状态 */
    private final Condition trip = lock.newCondition();
    /** 屏障点,简单来说当处于wait状态的数量达到这个值时,所有线程都会被唤醒 */
    private final int parties;
    /* 所有线程到达屏障点之后,执行的线程,这个是一个可选参数 */
    private final Runnable barrierCommand;
    /** 当前的屏障点实例,generation是CyclicBarrier的一个静态内部类 */
    private Generation generation = new Generation();
    /**
     * 当前还在等待的线程数量,从parties开始递减,当达到0时,所有线程被唤醒
     */
    private int count;
```

**Generation源码:**

```java
    private static class Generation {
        // 表示线程是否被中断过
        boolean broken = false;
    }
```

## 构造方法及初始化

    /**
     * parties:传入屏障点数量,当等待的线程达到这个数量时,唤醒所有线程
     */
    public CyclicBarrier(int parties) {
        this(parties, null);
    }
    
    /**
    * parties:传入屏障点数量,当等待的线程达到这个数量时,唤醒所有线程
    * barrierAction : 传入一个Runnable的实例,当所有线程到达屏障点时,执行barrierAction的run方法
    */
    public CyclicBarrier(int parties, Runnable barrierAction) {
    	if (parties <= 0) throw new IllegalArgumentException();
    	this.parties = parties;
    	this.count = parties;
    	this.barrierCommand = barrierAction;
    }
## 核心await方法

await方法有两个,一个无参的,一个有参的方法,两个方法都是通过调用内部的dowait方法实现的.

```java
    public int await() throws InterruptedException, BrokenBarrierException {
        try {
            return dowait(false, 0L);
        } catch (TimeoutException toe) {
            throw new Error(toe); // cannot happen
        }
    }

    public int await(long timeout, TimeUnit unit)
        throws InterruptedException,
               BrokenBarrierException,
               TimeoutException {
        return dowait(true, unit.toNanos(timeout));
    }
```

**dowait方法详解:**

```java
    /**
     * 主要的屏障代码,各种情况都在这里进行处理.
     */
    private int dowait(boolean timed, long nanos)
        throws InterruptedException, BrokenBarrierException,
               TimeoutException {
        // 获取当前屏障的可重入锁
        final ReentrantLock lock = this.lock;
        // 获取锁,保证下面的代码同一时间只有一个线程能执行,保证CyclicBarrier的各个属性的线程全性       
        lock.lock();
        try {
            // 获取当前屏障
            final Generation g = generation;
		   // broken表示当前线程是否被打断过,如果被打断过,就抛出BrokenBarrierException异常
            if (g.broken)
                throw new BrokenBarrierException();
		   // 判断当前线程是否被打断了	
            if (Thread.interrupted()) {
                // 如果当前线程被打断了,就将当前屏障状态g.broken设置为true
                // 重置count并唤醒所有等待中的线程
                breakBarrier();
                throw new InterruptedException();
            }
		   
            // 每个线程调用await方法时,都将count减1
            int index = --count;
            // 表示count减为0了,表示所有线程都已到达屏障点,可以唤醒所有线程了
            if (index == 0) {  // tripped
                // 标识是否已经执行了barrierCommand中的操作
                boolean ranAction = false;
                try {
                    // 获取当前的CyclicBarrier的,barrierCommand
                    final Runnable command = barrierCommand;
                    // 判断有没有传入额外的Runnable实例,如果传入了,就调用Runnable的run方法
                    if (command != null)
                        command.run();
                    // barrierCommand为空或run方法已经执行之后,就将ranAction方法设置为空
                    ranAction = true;
                    // 重置屏障点,这里实现CyclicBarrier的循环屏障,nextGeneration方法会唤醒所有处于等待状态的线程
                    nextGeneration();
                    return 0;
                } finally {
                    // 如果最终ranAction为false,表示有线程被打断了,调用breakBarrier方法中断当前屏障
                    if (!ranAction)
                        breakBarrier();
                }
            }

            // 死循环,在所有线程到达屏障点之前,让线程都处于等待状态
            for (;;) {
                try {
                    // 判断是否设置了超时时间
                    if (!timed)
                        // 如果没有设置超时时间,就直接使用Condition的await方法让当前线程处于等待状态
                        // await是个非阻塞方法
                        trip.await();
                    else if (nanos > 0L)
                        // 如果设置了超时时间,则调用Condition的超时等待方法
                        nanos = trip.awaitNanos(nanos);
                } catch (InterruptedException ie) {
                    // 如果期间线程被打断了,且屏障broken状态不是被打断状态,就更改为打断状态,并唤醒所有等待线程
                    if (g == generation && ! g.broken) {
                        breakBarrier();
                        throw ie;
                    } else {
                        // 打断线程
                        Thread.currentThread().interrupt();
                    }
                }

                if (g.broken)
                    throw new BrokenBarrierException();

                if (g != generation)
                    return index;
			   // 如果等待时间超时,则抛出超时异常	
                if (timed && nanos <= 0L) {
                    breakBarrier();
                    throw new TimeoutException();
                }
            }
        } finally {
            // dowait方法执行完毕,释放可重入锁
            lock.unlock();
        }
    }
```

## 其他方法

###nextGeneration方法

```java
    /**
     * 唤醒所有等待中的线程,开启下一个屏障
     * 开启下一个屏障
     */
    private void nextGeneration() {
        // Condition的signalAll方法,唤醒所有在当前condition上等待的线程
        trip.signalAll();
        // 重置count计数为屏障初始值
        count = parties;
        // 开启下一个新的屏障
        generation = new Generation();
    }
```

### breakBarrier方法

```java
    /**
     * 中断当前屏障
     */
    private void breakBarrier() {
        // 将当前屏障状态broken状态设置为true,表示当前屏障被中断了
        generation.broken = true;
        // 重置count计数
        count = parties;
        // 唤醒所有等待中的线程
        trip.signalAll();
    }
```

### reset方法

````java
    public void reset() {
        // 获取当前屏障锁的引用
        final ReentrantLock lock = this.lock;
        // 获取锁
        lock.lock();
        try {
            // 中断当前屏障
            breakBarrier();   
            // 开启一个新的屏障
            nextGeneration(); 
        } finally {
            lock.unlock();
        }
    }
````

