# ThreadLocal简介

ThreadLocal是一个线程的局部变量,在多线程环境下,每个线程针对ThreadLocal的操作,都只会影响到自己本身的值,并不会对其他线程产生影响,从而解决一部分线程安全性问题,下面是jdk对这个工具类的描述

> 该类提供了线程局部 (thread-local) 变量。这些变量不同于它们的普通对应物，因为访问某个变量（通过其 `get` 或  `set` 方法）的每个线程都有自己的局部变量，它独立于变量的初始化副本。`ThreadLocal` 实例通常是类中的  private static 字段，它们希望将状态与某一个线程（例如，用户 ID 或事务 ID）相关联。

# ThreadLocal API

* protected T initialValue()

  > 用于返回此线程局部变量**当前线程**的初始值,通常需要重写这个方法给线程赋初始值

* public T get()

  > 返回此线程局部变量的**当前线程**副本中的值。如果变量没有用于当前线程的值，则先将其初始化为调用 initialValue()  方法返回的值。

* public void set(T value)

  > 将此线程局部变量的**当前线程**副本中的值设置为指定值

* public void remove()

  > 移除此线程局部变量当前线程的值。如果此线程局部变量随后被当前线程读取，且这期间当前线程没有设置其值，则将调用其 initialValue()  方法重新初始化其值。这将导致在当前线程多次调用 initialValue 方法。



# ThreadLocal简单样例

通过ThreadLocal实现一个每个现在之间相互独立的计数器

```java
package thread.concurrentutil ;

/**
 * ThreadLocal使用测试
 * @author Administrator
 *
 */
public class ThreadLocalDemo {
	
	private ThreadLocal<Integer> threadLocal = new ThreadLocal<Integer>() {
		/**
		 * 重写这个方法给Value赋初始值
		 */
		@Override
		protected Integer initialValue() {
			return new Integer(0) ;
		}
	} ;
	
	/**
	 * 每个线程调用getNext方法时,获取的都是从0开始递增计数器,且彼此之间互不影响
	 * @return
	 */
	public Integer getNext() {
		int value = threadLocal.get() ;
		threadLocal.set(value + 1) ;
		return value ;
	}
	
	public static void main(String [] args) {
		ThreadLocalDemo demo = new ThreadLocalDemo() ;
		// 通过匿名内部类的方式实现两个线程
		new Thread() {
			
			@Override
			public void run() {
				while (true) {
					System.out.println(Thread.currentThread().getName() + ":" + demo.getNext()) ;
					try {
						Thread.sleep(1000) ;
					} catch (InterruptedException e) {
						e.printStackTrace() ;
					}
				}
			}
		}.start() ;
		
		new Thread() {
			
			@Override
			public void run() {
				while (true) {
					System.out.println(Thread.currentThread().getName() + ":" + demo.getNext()) ;
					try {
						Thread.sleep(2000) ;
					} catch (InterruptedException e) {
						e.printStackTrace() ;
					}
				}
			}
		}.start() ;
		
	}
	
}
```

**运行结果:**

>Thread-0:0
>Thread-1:0
>Thread-0:1
>Thread-1:1
>Thread-0:2
>Thread-0:3
>Thread-1:2
>Thread-0:4

从上面的运行结果可以看出,每个线程在使用getNext方法获取下一个计数时,都是自身的计数器,且线程之间互相不影响

# ThreadLocal实现原理

ThreadLocal的实现原理其实很简单,针对每一个线程都会有一个ThreadLocalMap与之绑定,这个ThreadLocalMap以当前的ThreadLocal为key,当前线程的value值为value,这些可以通过看下jdk源码

##首先看下ThreadLocal的get方法的实现

```java
 public T get() {
     	// 获取当前线程
        Thread t = Thread.currentThread();
     	// 获取当前线程绑定的ThreadLocalMap,这里获取的是t.threadLocals
        ThreadLocalMap map = getMap(t);
     	// 根据当前的ThreadLocalMap获取当前线程的value
        if (map != null) {
            // this这里表示当前的ThreadLocal对象
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
     	// 如果ThreadLocalMap为空的话,会调用setInitialValue返回初始值,并生成一个ThreadLocalMap与当前的线程t进行绑定
        return setInitialValue();
    }
```

## setInitialValue方法详解

```java
private T setInitialValue() {
    	// initialValue这个方法需要我们重写,用来给ThreadLocal设置初始值,以避免在调用get()方法时报错
        T value = initialValue();
        Thread t = Thread.currentThread();
    	// 获取当前线程绑定的ThreadLocalMap
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            /*
             * 如果当前线程没有绑定的ThreadLocalMap,就通过createMap创建一个map并绑定到当前线程上
             * t.threadLocals = new ThreadLocalMap(this, firstValue); //此处新建一个		 ThreadLocalMap并绑定到当前线程上
             */
            createMap(t, value);
        return value;
    }
```

## set方法详解

```java
    /**
     * set方法其实很简单,ThreadLocalMap如果不存在就新建,并将这个值设进去
     * ThreadLocalMap如果存在,就通过map.set(this, value);更新这个值
     * map.set(this, value)这个方法就等同于HashMap中的put(K,V)
     */
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

## remove方法

```java
/**
 * 移除当前的ThreadLocal里面的值
 * 如果没有重新set的话,下次调用get方法获取到的就是初始值
 */
	public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }
```



