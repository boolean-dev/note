# 详解synchronized与Lock的区别与使用

### 1. 引言：

昨天在学习别人分享的面试经验时，看到Lock的使用。想起自己在上次面试也遇到了synchronized与Lock的区别与使用。于是，我整理了两者的区别和使用情况，同时，对synchronized的使用过程一些常见问题的总结，最后是参照源码和说明文档，对Lock的使用写了几个简单的Demo。请大家批评指正。

#### 1.1 技术点：

##### 1.1.1 线程与进程：

在开始之前先把进程与线程进行区分一下，一个程序最少需要一个进程，而一个进程最少需要一个线程。关系是线程–>进程–>程序的大致组成结构。所以线程是程序执行流的最小单位，而进程是系统进行资源分配和调度的一个独立单位。以下我们所有讨论的都是建立在线程基础之上。

##### 1.1.2 Thread的几个重要方法：

我们先了解一下Thread的几个重要方法。a、start()方法，调用该方法开始执行该线程；b、stop()方法，调用该方法强制结束该线程执行；c、join方法，调用该方法等待该线程结束。d、sleep()方法，调用该方法该线程进入等待。e、run()方法，调用该方法直接执行线程的run()方法，但是线程调用start()方法时也会运行run()方法，区别就是一个是由线程调度运行run()方法，一个是直接调用了线程中的run()方法！！

看到这里，可能有些人就会问啦，那wait()和notify()呢？要注意，其实wait()与notify()方法是Object的方法，不是Thread的方法！！同时，wait()与notify()会配合使用，分别表示线程挂起和线程恢复。

这里还有一个很常见的问题，顺带提一下：wait()与sleep()的区别，简单来说wait()会释放对象锁而sleep()不会释放对象锁。这些问题有很多的资料，不再赘述。

##### 1.1.3 线程状态：



线程总共有5大状态，通过上面第二个知识点的介绍，理解起来就简单了。

新建状态：新建线程对象，并没有调用start()方法之前

就绪状态：调用start()方法之后线程就进入就绪状态，但是并不是说只要调用start()方法线程就马上变为当前线程，在变为当前线程之前都是为就绪状态。值得一提的是，线程在睡眠和挂起中恢复的时候也会进入就绪状态哦。

运行状态：线程被设置为当前线程，开始执行run()方法。就是线程进入运行状态

阻塞状态：线程被暂停，比如说调用sleep()方法后线程就进入阻塞状态

死亡状态：线程执行结束

##### 1.1.4 锁类型

可重入锁：在执行对象中所有同步方法不用再次获得锁

可中断锁：在等待获取锁过程中可中断

公平锁： 按等待获取锁的线程的等待时间进行获取，等待时间长的具有优先获取锁权利

读写锁：对资源读取和写入的时候拆分为2部分处理，读的时候可以多线程一起读，写的时候必须同步地写

### 2. synchronized与Lock的区别

#### 2.1 两者区别

| **类别** | **synchronized**                                             | **Lock**                                                     |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 存在层次 | Java的关键字，在jvm层面上                                    | 是一个类                                                     |
| 锁的释放 | 1、以获取锁的线程执行完同步代码，释放锁 <br>2、线程执行发生异常，jvm会让线程释放锁 | 在finally中必须释放锁，不然容易造成线程死锁                  |
| 锁的获取 | 假设A线程获得锁，B线程等待。<br>如果A线程阻塞，B线程会一直等待 | 分情况而定，Lock有多个锁获取的方式，具体下面会说道<br>大致就是可以尝试获得锁，线程可以不用一直等待 |
| 锁状态   | 无法判断                                                     | 可以判断                                                     |
| 锁类型   | 可重入 不可中断 非公平                                       | 可重入 可判断 可公平（两者皆可）                             |
| 性能     | 少量同步                                                     | 大量同步                                                     |

### 3. Lock详细介绍与Demo

以下是Lock接口的源码，笔者修剪之后的结果：

```java
public interface Lock {

    /**
     * Acquires the lock.
     */
    void lock();

    /**
     * Acquires the lock unless the current thread is
     * {@linkplain Thread#interrupt interrupted}.
     */
    void lockInterruptibly() throws InterruptedException;

    /**
     * Acquires the lock only if it is free at the time of invocation.
     */
    boolean tryLock();

    /**
     * Acquires the lock if it is free within the given waiting time and the
     * current thread has not been {@linkplain Thread#interrupt interrupted}.
     */
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

    /**
     * Releases the lock.
     */
    void unlock();
	
}
```


从Lock接口中我们可以看到主要有个方法，这些方法的功能从注释中可以看出：

- lock()：获取锁，如果锁被暂用则一直等待

- unlock():释放锁

- tryLock(): 注意返回类型是boolean，如果获取锁的时候锁被占用就返回false，否则返回true

- tryLock(long time, TimeUnit unit)：比起tryLock()就是给了一个时间期限，保证等待参数时间

- lockInterruptibly()：用该锁的获得方式，如果线程在获取锁的阶段进入了等待，那么可以中断此线程，先去做别的事

通过 以上的解释，大致可以解释在上个部分中“锁类型(lockInterruptibly())”，“锁状态(tryLock())”等问题，还有就是前面子所获取的过程我所写的“大致就是可以尝试获得锁，线程可以不会一直等待”用了“可以”的原因。

下面是Lock一般使用的例子，注意ReentrantLock是Lock接口的实现。
lock()：

```java

package com.brickworkers;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class LockTest {
	private Lock lock = new ReentrantLock();

	//需要参与同步的方法
	private void method(Thread thread){
		lock.lock();
		try {
			System.out.println("线程名"+thread.getName() + "获得了锁");
		}catch(Exception e){
			e.printStackTrace();
		} finally {
			System.out.println("线程名"+thread.getName() + "释放了锁");
			lock.unlock();
		}
	}
	
	public static void main(String[] args) {
		LockTest lockTest = new LockTest();
		
		//线程1
		Thread t1 = new Thread(new Runnable() {
			
			@Override
			public void run() {
				lockTest.method(Thread.currentThread());
			}
		}, "t1");
		
		Thread t2 = new Thread(new Runnable() {
			
			@Override
			public void run() {
				lockTest.method(Thread.currentThread());
			}
		}, "t2");
		
		t1.start();
		t2.start();
	}
}
//执行情况：线程名t1获得了锁
//         线程名t1释放了锁
//         线程名t2获得了锁
//         线程名t2释放了锁

```

tryLock():

```java
package com.brickworkers;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class LockTest {
	private Lock lock = new ReentrantLock();

	//需要参与同步的方法
	private void method(Thread thread){
/*		lock.lock();
		try {
			System.out.println("线程名"+thread.getName() + "获得了锁");
		}catch(Exception e){
			e.printStackTrace();
		} finally {
			System.out.println("线程名"+thread.getName() + "释放了锁");
			lock.unlock();
		}*/
		
		
		if(lock.tryLock()){
			try {
				System.out.println("线程名"+thread.getName() + "获得了锁");
			}catch(Exception e){
				e.printStackTrace();
			} finally {
				System.out.println("线程名"+thread.getName() + "释放了锁");
				lock.unlock();
			}
		}else{
			System.out.println("我是"+Thread.currentThread().getName()+"有人占着锁，我就不要啦");
		}
	}
	
	public static void main(String[] args) {
		LockTest lockTest = new LockTest();
		
		//线程1
		Thread t1 = new Thread(new Runnable() {
			
			@Override
			public void run() {
				lockTest.method(Thread.currentThread());
			}
		}, "t1");
		
		Thread t2 = new Thread(new Runnable() {
			
			@Override
			public void run() {
				lockTest.method(Thread.currentThread());
			}
		}, "t2");
		
		t1.start();
		t2.start();
	}
}

//执行结果： 线程名t2获得了锁
//         我是t1有人占着锁，我就不要啦
//         线程名t2释放了锁


```

看到这里相信大家也都会使用如何使用Lock了吧，关于tryLock(long time, TimeUnit unit)和lockInterruptibly()不再赘述。前者主要存在一个等待时间，在测试代码中写入一个等待时间，后者主要是等待中断，会抛出一个中断异常，常用度不高，喜欢探究可以自己深入研究。

前面比较重提到“公平锁”，在这里可以提一下ReentrantLock对于平衡锁的定义，在源码中有这么两段：

```java

 /**
     * Sync object for non-fair locks
     */
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        /**
         * Performs lock.  Try immediate barge, backing up to normal
         * acquire on failure.
         */
        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }

    /**
     * Sync object for fair locks
     */
    static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            acquire(1);
        }

        /**
         * Fair version of tryAcquire.  Don't grant access unless
         * recursive call or no waiters or is first.
         */
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
```
从以上源码可以看出在Lock中可以自己控制锁是否公平，而且，默认的是非公平锁，以下是ReentrantLock的构造函数：

```java
   public ReentrantLock() {
        sync = new NonfairSync();//默认非公平锁
    }
```

