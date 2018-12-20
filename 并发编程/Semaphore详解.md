# Semaphore简介

​	Semaphore是一个计数信号量,是由JDK提供的一个并发工具类,在java.util.concurrent包下.下面是jdk中对Semaphore的描述:

>一个计数信号量。从概念上讲，信号量维护了一个许可集。如有必要，在许可可用前会阻塞每一个 acquire()，然后再获取该许可。每个 release() 添加一个许可，从而可能释放一个正在阻塞的获取者。但是，不使用实际的许可对象，Semaphore 只对可用许可的号码进行计数，并采取相应的行动。 
>
>Semaphore 通常用于限制可以访问某些资源（物理或逻辑的）的线程数目



# API简介

##Semaphore有两个构造方法

- Semaphore(int permits): permits参数用来指定许可数,实现的是一个非公平信号量计数器
- Semaphore(int permits,  boolean fair):permits参数指定许可数,fair参数表示是否是公平

**关于公平锁和非公平锁引用一下别人的描述:**

>公平锁是指多个线程在等待同一个锁时，必须按照申请锁的先后顺序来一次获得锁。
>
>公平锁的好处是等待锁的线程不会饿死，但是整体效率相对低一些；非公平锁的好处是整体效率相对高一些，但是有些线程可能会饿死或者说很早就在等待锁，但要等很久才会获得锁。其中的原因是公平锁是严格按照请求所的顺序来排队获得锁的，而非公平锁时可以抢占的，即如果在某个时刻有线程需要获取锁，而这个时候刚好锁可用，那么这个线程会直接抢占，而这时阻塞在等待队列的线程则不会被唤醒。

## acquire方法

> * acquire()
>
>   从此信号量获取一个许可，在提供一个许可前一直将线程阻塞，否则线程被中断。
>
> * acquire(int permits)
>
>   从此信号量获取给定数目的许可，在提供这些许可前一直将线程阻塞，或者线程已被[中断](../../../java/lang/Thread.html#interrupt())。
>
> * acquireUninterruptibly()
>
>   从此信号量中获取许可，在有可用的许可前将其阻塞。
>
> * acquireUninterruptibly(int permits)
>
> 		从此信号量获取给定数目的许可，在提供这些许可前一直将线程阻塞。

## release方法

> public void release()
>
> ​	释放一个许可，将其返回给信号量。
>
> ​	释放一个许可，将可用的许可数增加 1。如果任意线程试图获取许可，则选中一个线程并将刚刚释放的许可给予它。然后针对线程安排目的启用（或再启用）该线程。  
>
> ​	**不要求释放许可的线程必须通过调用 acquire()  来获取许可**。通过应用程序中的编程约定来建立信号量的正确用法。 

>**public void release(int permits): **
>
>​	释放给定数目的许可，将可用的许可数增加该量。如果任意线程试图获取许可，则选中某个线程并将刚							    	刚释放的许可给予该线程。如果可用许可的数目满足该线程的请求，则针对线程安排目的启用（或再启用）该线程；否则在有足够的可用许可前线程将一直等待。如果满足此线程的请求后仍有可用的许可，则依次将这些许可分配给试图获取许可的其他线程。 
>
>​	**不要求释放许可的线程必须通过调用获取来获取该许可**。通过应用程序中的编程约定来建立信号量的正确用法。 
>
>
>参数：
>permits - 要释放的许可数 
>抛出： 
>IllegalArgumentException - 如果 permits 为负

需要注意的是,"**不要求释放许可的线程必须通过调用获取来获取该许可**"这句话表示,如果没有调用acquire()方法去获取许可,也可直接调用release方法进行释放许可,具体看下下面的代码就知道了

```java
    // 初始化一个计数信号量,设置许可集数量为10
    Semaphore semaphore = new Semaphore(10);
    // 这里打印的结果为 10,availablePermits方法返回可用的许可数量
    System.out.println(semaphore.availablePermits());
    // 没有使用acquire方法获取许可,直接进行许可释放,执行完成之后,许可计数信号量+1
    semaphore.release();
    // 这里打印结果为:11
    System.out.println(semaphore.availablePermits());
```

# 使用样例

了解Semaphore的简介,及常用的Api,那就使用一个简单的样例来展示一下Semaphore的简单用法吧.

样例场景如下:

​	有一个游戏厅,里面有N台机器可以供玩家玩游戏,玩家到游戏厅来玩游戏,如果游戏厅还有空余的机器的话,就直接给玩家分配一台机器,如果机器已经全部被使用了,就需要等待有人游戏结束,让出机器,才能获取机器开始游戏.在这个例子中,**游戏厅的机器就等于Semaphore计数信号量中的许可**

**代码如下:**

```java
import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

/**
 * 游戏厅类,通过构造方法传入游戏厅拥有的机器数量
 *
 */
class PlayRoom {
  // 直接用Semaphore持有的许可数量,来表示机器的数量,这里游戏机就代表"许可"
  private Semaphore semaphore;

  public PlayRoom(int permits) {
    super();
    // 初始化计数信号量semaphore
    this.semaphore = new Semaphore(permits);
    // this.semaphore = new Semaphore(permits, true);
  }

  /**
   * 开始游戏方法
   */
  public void startGame() {

    try {
      // 尝试获取台机器,如果获取不到,则进程处于等待状态
      semaphore.acquire();
      // 休眠2秒,表示一个玩家的游戏时长
      TimeUnit.SECONDS.sleep(2);
    } catch (InterruptedException e) {
      e.printStackTrace();
    } finally {
      // 游戏结束,归还机器,即归还技术信号量的许可
      semaphore.release();
    }

  }

  // 返回当前游戏厅剩余的机器数量
  public int remain() {
    // availablePermits方法返回一个计数信号量的可用许可数量
    return semaphore.availablePermits();
  }


}


/**
 * 玩家类
 *
 */
class Player implements Runnable {
  // 玩家必须要找到一个游戏厅,才能开始玩游戏,游戏厅类
  private PlayRoom playRoom;

  public Player(PlayRoom playRoom) {
    super();
    this.playRoom = playRoom;
  }

  @Override
  public void run() {
    System.out.println("玩家: " + Thread.currentThread().getName() + " 开始游戏,游戏厅剩余机器:" + playRoom.remain());
    // 直接调用游戏厅的startGame开始游戏
    playRoom.startGame();
  }
}


public class SemaphoreDemo {
  public static void main(String[] args) {
    PlayRoom playRoom = new PlayRoom(10);
    while (true) {
      new Thread(new Player(playRoom)).start();
      try {
        Thread.sleep(100);
      } catch (Exception e) {
        e.printStackTrace();
      }
    }
  }
}

```

**输出结果:**

>玩家: Thread-0 开始游戏,游戏厅剩余机器:10
>玩家: Thread-1 开始游戏,游戏厅剩余机器:9
>玩家: Thread-2 开始游戏,游戏厅剩余机器:8
>玩家: Thread-3 开始游戏,游戏厅剩余机器:7
>玩家: Thread-4 开始游戏,游戏厅剩余机器:6
>玩家: Thread-5 开始游戏,游戏厅剩余机器:5
>玩家: Thread-6 开始游戏,游戏厅剩余机器:4
>玩家: Thread-7 开始游戏,游戏厅剩余机器:3
>玩家: Thread-8 开始游戏,游戏厅剩余机器:2
>玩家: Thread-9 开始游戏,游戏厅剩余机器:1
>玩家: Thread-10 开始游戏,游戏厅剩余机器:0
>玩家: Thread-11 开始游戏,游戏厅剩余机器:0
>玩家: Thread-12 开始游戏,游戏厅剩余机器:0
>玩家: Thread-13 开始游戏,游戏厅剩余机器:0
>玩家: Thread-14 开始游戏,游戏厅剩余机器:0
>玩家: Thread-15 开始游戏,游戏厅剩余机器:0

由于在新增现程的时候增加了**Thread.sleep(100)**休眠操作,所以看起来像是公平的,按顺序获取机器,如果将休眠的这个代码注释掉的话,执行的结果就是下面这种情况了:

>玩家: Thread-57367 开始游戏,游戏厅剩余机器:0
>玩家: Thread-57370 开始游戏,游戏厅剩余机器:0
>玩家: Thread-57369 开始游戏,游戏厅剩余机器:0
>玩家: Thread-57371 开始游戏,游戏厅剩余机器:0
>玩家: Thread-57374 开始游戏,游戏厅剩余机器:0
>玩家: Thread-57375 开始游戏,游戏厅剩余机器:0
>玩家: Thread-57373 开始游戏,游戏厅剩余机器:0
>玩家: Thread-57377 开始游戏,游戏厅剩余机器:0
>玩家: Thread-57378 开始游戏,游戏厅剩余机器:0
>玩家: Thread-57379 开始游戏,游戏厅剩余机器:0

这种说明表示在只传递一个许可数量参数进行示例话计数信号量时,默认是基于非公平的实现.



# 源码分析

## 构造方法

```java
    /**
     * 默认是非公平的实现
     * 通过AQS中的state实现对许可集数量的控制
     * @param permits 许可集数量
     */
    public Semaphore(int permits) {
        sync = new NonfairSync(permits);
    }

    /**
     * permits
     * permits and the given fairness setting.
     *
     * @param permits 许可集数量
     * @param fair : 是否为公平模式
     *        	true : 以公平模式实现计数信号量,不允许抢占,获取信号顺序和等待顺序保持一致
     *          false : 以非公平模式实现,等同于不加fair参数的构造方法Semaphore(int permits)
     */
    public Semaphore(int permits, boolean fair) {
        sync = fair ? new FairSync(permits) : new NonfairSync(permits);
    }
```

**关于公平锁与非公平锁的概念,这里不细讲,这个是基于AQS(AbstractQueuedSynchronizer)框架实现的一个可重入锁,AQS是java并发中一个很核心的东西,关于这一块不了解的可以自行上网查询一下.**

## acquire方法

如果了解AQS的话,这个代码就很简单了,就是调用AQS中的acquireSharedInterruptibly去获取锁

```java
    /**
     * 通过调用sync方法去获取锁
     * @throws 线程被打断则抛出异常
     */
    public void acquire() throws InterruptedException {
        // acquireSharedInterruptibly是AQS中以可中断方式获取锁
        sync.acquireSharedInterruptibly(1);
    }


    /**
     * 通过调用sync方法去获取锁
     */
    public void acquireUninterruptibly() {
        sync.acquireShared(1);
    }
	
    /**
     * 通过调用sync方法去获取锁
     * @throws 线程被打断则抛出异常
     */
    public void acquire(int permits) throws InterruptedException {
        if (permits < 0) throw new IllegalArgumentException();
        sync.acquireSharedInterruptibly(permits);
    }

    /**
     * 通过调用sync方法去获取锁
     */
    public void acquireUninterruptibly(int permits) {
        if (permits < 0) throw new IllegalArgumentException();
        sync.acquireShared(permits);
    }
```

### AQS的acquireSharedInterruptibly方法

```java
    /**
     * 以共享模式获取对象，如果被中断则中止。
     * 在Semaphore中,允许多个线程同时访问许可,所以这里要实现AQS的tryAcquireShared(int)和 tryReleaseShared(int) 
     * 通过先检查中断状态，然后至少调用一次 tryAcquireShared(int) 来实现此方法，并在成功时返回。
     * 否则在成功或线程被中断之前，一直调用 tryAcquireShared(int) 将线程加入队列，线程可能重复被阻塞或不被阻塞。 
     * tryAcquireShared方法是一个抽象方法,此处是由Semaphore的内部类实现具体功能
     * NonfairSync类实现了一个非公平的版本
     * FairSync提供了一个公平的版本
     * @throws InterruptedException 当前线程被打断,则抛出异常
     */
    public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        /**
         * 试图使用共享模式获取对象,如果获取失败,则将当前线程加入等待队列
         * tryAcquireShared方法去获取对象,返回值大于等于0表示获取成功,小于0表示获取失败
         */
        if (tryAcquireShared(arg) < 0)
            // 将当前线程加入等待队列
            doAcquireSharedInterruptibly(arg);
    }


    /**
     * 共享模式下,将当前线程加入等待队列
     */
    private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
        // 把当前线程以共享模式创建一个Node节点
        // 把创建的节点加入到等待队列中
        // 如果队列为空,则新建一个队列,加入进去
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```





## release方法

```java
    /**
     * release方法一样是直接调用sync的方法进行实现.
     */
    public void release() {
        sync.releaseShared(1);
    }

    /**
     * 同样是调用sync的方法进行实现.
     * @param 要释放或者说归还给信号量的许可数量
     * @throws permits非法则抛出IllegalArgumentException异常
     */
    public void release(int permits) {
        if (permits < 0) throw new IllegalArgumentException();
        sync.releaseShared(permits);
    }
```

从上面的代码可以看出来,实现这些功能的核心就是Sync类,根据设置分为公平的FairSync和非公平的NonfairSync,那就简单来看下Sync的具体实现吧

### Sync源码解析

```java
    /**
     * 信号量的同步实现,通过AQS的state状态表示许可的数量permits
     * 这是一个抽象类,下面分别有基于公平锁的实现FairSync
     * 和基于非公平锁的实现NonfairSync,默认情况下使用的是NonfairSync
     */
    abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 1192457210091910933L;
	     	
        Sync(int permits) {
            // 初始化State,将状态设置为Semaphore许可的数量permits
            setState(permits);
        }

        final int getPermits() {
            return getState();
        }
	    
        /**
         * 以非公平的方式尝试获取锁
         * Semaphore的非公平版本的
         */
        final int nonfairTryAcquireShared(int acquires) {
            for (;;) {
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }

        protected final boolean tryReleaseShared(int releases) {
            for (;;) {
                int current = getState();
                int next = current + releases;
                if (next < current) // overflow
                    throw new Error("Maximum permit count exceeded");
                if (compareAndSetState(current, next))
                    return true;
            }
        }

        final void reducePermits(int reductions) {
            for (;;) {
                int current = getState();
                int next = current - reductions;
                if (next > current) // underflow
                    throw new Error("Permit count underflow");
                if (compareAndSetState(current, next))
                    return;
            }
        }

        final int drainPermits() {
            for (;;) {
                int current = getState();
                if (current == 0 || compareAndSetState(current, 0))
                    return current;
            }
        }
    }
```

-- 未完待续--