# Java并发之CyclicBarrier

​	**该博客转载自[掘金](www.jianshu.com) 的[Java并发之CyclicBarrier](https://juejin.im/entry/596a05fdf265da6c4f34f2f9)**

  barrier(屏障)与互斥量、读写锁、自旋锁不同，它不是用来保护临界区的。相反，它跟条件变量一样，是用来协同多线程一起工作的。
  条件变量是多线程间传递状态的改变来达到协同工作的效果。屏障是多线程各自做自己的工作，如果某一线程完成了工作，就等待在屏障那里，直到其他线程的工作都完成了，再一起做别的事。举个通俗的例子：
  1.对于条件变量。在接力赛跑里，1号队员开始跑的时候，2，3，4号队员都站着不动，直到1号队员跑完一圈，把接力棒给2号队员，2号队员收到接力棒后就可以跑了，跑完再给3号队员。这里这个接力棒就相当于条件变量，条件满足后就可以由下一个队员(线程)跑。
  2.对于屏障：在百米赛跑里，比赛没开始之前，每个运动员都在赛场上自由活动，有的热身，有的喝水，有的跟教练谈论。比赛快开始时，准备完毕的运动员就预备在起跑线上，如果有个运动员还没准备完(除去特殊情况)，他们就一直等，直到运动员都在起跑线上，裁判喊口号后再开始跑。这里的起跑线就是屏障，做完准备工作的运动员都等在起跑线，直到其他运动员也把准备工作做完。

  `java.util.concurrent.CyclicBarrie`r类是一个同步机制。它可以通过一些算法来同步线程处理的过程。换言之，就是所有的线程必须等待对方，直到所有的线程到达屏障，然后继续运行。之所以叫做“循环屏障”，是因为这个屏障可以被重复使用。

  `CyclicBarrier`有两个构造参数，分别是：

  `CyclicBarrier(int parties)`

  创建一个新的` CyclicBarrier`，它将在给定数量的参与者（线程）处于等待状态时启动，但它不会在启动 barrier 时执行预定义的操作。
  `CyclicBarrier(int parties, Runnable barrierAction)`

  创建一个新的 `CyclicBarrier`，它将在给定数量的参与者（线程）处于等待状态时启动，并在启动 barrier 时执行给定的屏障操作，该操作由最后一个进入 `barrier` 的线程执行。

# 让线程在CyclicBarrier中等待

  有两个方法可以让线程在CyclicBarrier处等待：

  `barrier.await()`;
  `barrier.await(10, TimeUnit.SECONDS);`
  第二个方法指线程等待的超时时间，当出现等待超时的时候，当前线程会被释放,但会像其他线程传播出`BrokenBarrierException`异常。
  所有线程在`CyclicBarrier`等待，是指：

```sh
   • 最后一个线程到达（调用await方法）

   • 一个线程被被另外一个线程中断（另外一个线程调用了这个现场的interrupt()方法）

   • 其中一个等待的线程被中断

   • 其中一个等待的线程超时

   • 一个外部的线程调用了CyclicBarrier.reset()方法。
```

  下面以5个线程模拟5个运动员。运动员在赛跑的时候都会准备一段时间，当裁判发现所有的运动员都准备完毕的时候，就举起发令枪，比赛开始。

```java
package thread;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
/**
* 模拟运动员
**/
public class MyThread extends Thread {
    private CyclicBarrier cyclicBarrier;
    private String name;

    public MyThread(CyclicBarrier cyclicBarrier, String name) {
        super();
        this.cyclicBarrier = cyclicBarrier;
        this.name = name;
    }

    @Override
    public void run() {
        System.out.println(name + "开始准备");
        try {
            Thread.currentThread().sleep(5000);
            System.out.println(name + "准备完毕！等待发令枪");
            try {
                cyclicBarrier.await();
            } catch (BrokenBarrierException e) {            
                e.printStackTrace();
            }
        } catch (InterruptedException e) {

            e.printStackTrace();
        }
    }
}
//测试类
public class Test {
    public static void main(String[] args) {
        CyclicBarrier barrier = new CyclicBarrier(5, new Runnable() {

            @Override
            public void run() {
                System.out.println("发令枪响了，跑！");

            }
        });
        for (int i = 0; i < 5; i++) {
            new MyThread(barrier, "运动员" + i + "号").start();

        }

    }

}
```

  当执行测试类的时候，输出如下的结果（顺序每次执行可能会不太一样）：

```sh
运动员1号开始准备
运动员3号开始准备
运动员2号开始准备
运动员0号开始准备
运动员4号开始准备
运动员1号准备完毕！等待发令枪
运动员4号准备完毕！等待发令枪
运动员0号准备完毕！等待发令枪
运动员3号准备完毕！等待发令枪
运动员2号准备完毕！等待发令枪
发令枪响了，跑!
```

  从输出可以看到,当给定数量的参与者（线程）调用了await()方法之后，屏障放开，CyclicBarrier中的屏障动作被触发了。如果没有达到指定的数量，就会一直被阻塞。

# Barrier被破坏

`BrokenBarrierException`如果在参与者（线程）在等待的过程中，Barrier被破坏，就会抛出`BrokenBarrierException`。可以用**isBroken**()方法检测Barrier是否被破坏。
  **1.如果有线程已经处于等待状态，调用reset方法会导致已经在等待的线程出现`BrokenBarrierException`异常。并且由于出现了`BrokenBarrierException`，将会导致始终无法等待。**

  比如，五个运动员，其中一个在等待发令枪的过程中错误地接收到裁判传过来的指令，导致这个运动员以为今天比赛取消就离开了赛场。但是其他运动员都领会的裁判正确的指令，剩余的运动员在起跑线上无限地等待下去，并且裁判看到运动员没有到齐，也不会打发令枪。

```java
package thread;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class MyThread extends Thread {
    private CyclicBarrier cyclicBarrier;
    private String name;
    private int ID;

    public MyThread(CyclicBarrier cyclicBarrier, String name,int ID) {
        super();
        this.cyclicBarrier = cyclicBarrier;
        this.name = name;
        this.ID=ID;

    }
    @Override
    public void run() {
        System.out.println(name + "开始准备");
        try {
            Thread.sleep(ID*1000);  //不同运动员准备时间不一样，方便模拟不同情况
            System.out.println(name + "准备完毕！在起跑线等待发令枪");
            try {
                cyclicBarrier.await();
                System.out.println(name + "跑完了路程！");
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
                System.out.println(name+"看不见起跑线了");
            }
            System.out.println(name+"退场！");
        } catch (InterruptedException e) {

            e.printStackTrace();
        }

    }

}
public class Test {

    public static void main(String[] args) throws InterruptedException {
        CyclicBarrier barrier = new CyclicBarrier(5, new Runnable() {
            @Override
            public void run() {
                System.out.println("发令枪响了，跑！");

            }
        });

        for (int i = 0; i < 5; i++) {
            new MyThread(barrier, "运动员" + i + "号", i).start();
        }
        Thread.sleep(1000);
        barrier.reset();
    }

}
```

输出结果：

```sh
运动员0号开始准备
运动员1号开始准备
运动员2号开始准备
运动员3号开始准备
运动员4号开始准备
运动员0号准备完毕！在起跑线等待发令枪
运动员1号准备完毕！在起跑线等待发令枪
java.util.concurrent.BrokenBarrierException
运动员0号看不见起跑线了
运动员0号退场！
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at thread.MyThread.run(MyThread.java:27)
运动员2号准备完毕！在起跑线等待发令枪
运动员3号准备完毕！在起跑线等待发令枪
运动员4号准备完毕！在起跑线等待发令枪
```

  从输出可以看到，运动员0号在等待的过程中，主线程调用了reset方法，导致抛出`BrokenBarrierException`异常。但是其他线程并没有受到影响，它们会一直等待下去，从而一直被阻塞。

  **2.如果在等待的过程中，线程被中断，也会抛出`BrokenBarrierException`异常，并且这个异常会传播到其他所有的线程。**

```java
package thread;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.CyclicBarrier;

public class Test {
static   Map<Integer,Thread>   threads=new HashMap<>();
    public static void main(String[] args) throws InterruptedException {
        CyclicBarrier barrier = new CyclicBarrier(5, new Runnable() {
            @Override
            public void run() {
                System.out.println("发令枪响了，跑！");

            }
        });

        for (int i = 0; i < 5; i++) {
        MyThread t = new MyThread(barrier, "运动员" + i + "号", i);
            threads.put(i, t);
            t.start();
        }
        Thread.sleep(3000);
        threads.get(1).interrupt();
    }

}
```

输出：

```sh
运动员0号开始准备
运动员2号开始准备
运动员3号开始准备
运动员1号开始准备
运动员0号准备完毕！在起跑线等待发令枪
运动员4号开始准备
运动员1号准备完毕！在起跑线等待发令枪
运动员2号准备完毕！在起跑线等待发令枪
运动员3号准备完毕！在起跑线等待发令枪
java.lang.InterruptedException
运动员3号看不见起跑线了
运动员3号退场！
运动员2号看不见起跑线了
运动员2号退场！
运动员0号看不见起跑线了
运动员0号退场！
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.reportInterruptAfterWait(AbstractQueuedSynchronizer.java:2014)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2048)
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:234)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at thread.MyThread.run(MyThread.java:27)
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at thread.MyThread.run(MyThread.java:27)
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at thread.MyThread.run(MyThread.java:27)
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at thread.MyThread.run(MyThread.java:27)
运动员4号准备完毕！在起跑线等待发令枪
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:207)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at thread.MyThread.run(MyThread.java:27)
运动员4号看不见起跑线了
运动员4号退场！
```

  从输出可以看到，其中一个线程被中断，那么所有的运动员都退场了。

  **3.如果在执行屏障操作过程中发生异常，则该异常将传播到当前线程中，其他线程会抛出BrokenBarrierException，屏障被损坏。**

  这个就好比运动员都没有问题，而是裁判出问题了。裁判权力比较大，直接告诉所有的运动员，今天不比赛了，你们都回家吧！

```java
package thread;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.CyclicBarrier;

public class Test {
    static Map<Integer, Thread> threads = new HashMap<>();

    public static void main(String[] args) throws InterruptedException {
        CyclicBarrier barrier = new CyclicBarrier(5, new Runnable() {
            @Override
            public void run() {
                String str = null;
                str.substring(0, 1);
                System.out.println("发令枪响了，跑！");

            }
        });

        for (int i = 0; i < 5; i++) {
            MyThread t = new MyThread(barrier, "运动员" + i + "号", i);
            threads.put(i, t);
            t.start();
        }

    }

}
```

输出：

```sh
运动员0号开始准备
运动员3号开始准备
运动员2号开始准备
运动员1号开始准备
运动员4号开始准备
运动员0号准备完毕！在起跑线等待发令枪
运动员1号准备完毕！在起跑线等待发令枪
运动员2号准备完毕！在起跑线等待发令枪
运动员3号准备完毕！在起跑线等待发令枪
运动员4号准备完毕！在起跑线等待发令枪
Exception in thread "Thread-4" java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at thread.MyThread.run(MyThread.java:27)
运动员0号看不见起跑线了
运动员0号退场！
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at thread.MyThread.run(MyThread.java:27)
运动员3号看不见起跑线了
运动员3号退场！
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at thread.MyThread.run(MyThread.java:27)
运动员1号看不见起跑线了
运动员1号退场！
java.lang.NullPointerException
    at thread.Test$1.run(Test.java:15)
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:220)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at thread.MyThread.run(MyThread.java:27)
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:250)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:362)
    at thread.MyThread.run(MyThread.java:27)
运动员2号看不见起跑线了
运动员2号退场！
```

  可以看到，如果在执行屏障动作的过程中出现异常，那么所有的线程都会抛出BrokenBarrierException异常。

  **4.如果超出指定的等待时间，当前线程会抛出 TimeoutException 异常，其他线程会抛出BrokenBarrierException异常。**

```java
package thread;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

public class MyThread extends Thread {
    private CyclicBarrier cyclicBarrier;
    private String name;
    private int ID;

    public MyThread(CyclicBarrier cyclicBarrier, String name, int ID) {
        super();
        this.cyclicBarrier = cyclicBarrier;
        this.name = name;
        this.ID = ID;

    }

    @Override
    public void run() {
        System.out.println(name + "开始准备");
        try {
            Thread.sleep(ID * 1000);
            System.out.println(name + "准备完毕！在起跑线等待发令枪");
            try {
                try {
                    cyclicBarrier.await(ID * 1000, TimeUnit.MILLISECONDS);
                } catch (TimeoutException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
                System.out.println(name + "跑完了路程！");
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
                System.out.println(name + "看不见起跑线了");
            }
            System.out.println(name + "退场！");
        } catch (InterruptedException e) {

            e.printStackTrace();
        }

    }

}
```

输出：

```sh
运动员0号开始准备
运动员2号开始准备
运动员3号开始准备
运动员1号开始准备
运动员0号准备完毕！在起跑线等待发令枪
运动员4号开始准备
java.util.concurrent.TimeoutException运动员0号跑完了路程！
运动员0号退场！

    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:257)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:435)
    at thread.MyThread.run(MyThread.java:29)
运动员1号准备完毕！在起跑线等待发令枪
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:207)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:435)
    at thread.MyThread.run(MyThread.java:29)
运动员1号看不见起跑线了
运动员1号退场！
运动员2号准备完毕！在起跑线等待发令枪
java.util.concurrent.BrokenBarrierException
运动员2号看不见起跑线了
运动员2号退场！
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:207)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:435)
    at thread.MyThread.run(MyThread.java:29)
运动员3号准备完毕！在起跑线等待发令枪
java.util.concurrent.BrokenBarrierException
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:207)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:435)
    at thread.MyThread.run(MyThread.java:29)
运动员3号看不见起跑线了
运动员3号退场！
运动员4号准备完毕！在起跑线等待发令枪
java.util.concurrent.BrokenBarrierException
运动员4号看不见起跑线了
运动员4号退场！
    at java.util.concurrent.CyclicBarrier.dowait(CyclicBarrier.java:207)
    at java.util.concurrent.CyclicBarrier.await(CyclicBarrier.java:435)
    at thread.MyThread.run(MyThread.java:29)
```

  从输出可以看到，如果其中一个参与者抛出TimeoutException，其他参与者会抛出BrokenBarrierException。

