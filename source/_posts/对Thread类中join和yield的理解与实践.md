---
title: 对Thread类中join和yield的理解与实践
date: 2018-07-16 22:09:10
categories: 技术
tags:
      - Java
---
### Java中thread的状态
* java中thread类的状态体现在java.lang.Thread类的State枚举里。包含以下几种状态：
1. NEW，表示线程被创建出来还没真正启动的状态，可以认为是java的内部状态
2. RUNNABLE，表示线程已经在JVM中执行，可以是在运行，也可以是在等待系统分配给它CPU片段，在就绪队列里面排队
3. BLOCKED，阻塞表示线程在等待Monitor lock，比如线程通过synchronized去获取某个锁，但是其他线程独占了，那么当前线程处于阻塞状态。或者，读取文件IO过程中的阻塞
4. WAITING，waiting表示线程正在等待其他线程采取某些操作，生产者消费者模式中，消费者等待（wait），生产者准备数据，然后调用notify通知消费线程继续工作。Thread.join()也会令线程进入等待状态。
5. TIMED_WAITING，有条件的等待。
6. TERMINATED，线程意外死亡或者完成使命，终止运行。
### 1. join()
* 解释：join方法的作用可以理解为同步，使得线程之间的并行执行变为串行执行（可以这么理解，原理是调用join的线程会让出线程锁进入等待状态，后面将分析源码）。
* 代码：
```
public class ThreadTest {

    public static void main(String[] args) throws InterruptedException {
        OneTread one = new OneTread();
        System.out.println("------begin------");
        one.start();
        /**
         * 下面加入join方法，main线程执行就切换为one这个线程来执行
         * 不加入join，则会并行执行main线程和one线程
         */
        one.join();
        System.out.println("------end------");
    }
    static class OneTread extends Thread{
        @Override
        public void run() {
            for (int i = 0; i != 5; ++i) {
                System.out.println("One thread is doing.i = " + i);
            }
            System.out.println("one thread is finished.");
        }
    }
}
```
* 输出,加入join：
```
------begin------
One thread is doing.i = 0
One thread is doing.i = 1
One thread is doing.i = 2
One thread is doing.i = 3
One thread is doing.i = 4
one thread is finished.
------end------
```
* 不加入join：
```
------begin------
------end------
One thread is doing.i = 0
One thread is doing.i = 1
One thread is doing.i = 2
One thread is doing.i = 3
One thread is doing.i = 4
one thread is finished.
```
* join(long millis)
millis代表线程等待的毫秒数，在此处则代表main线程等待one线程执行millis毫秒，然后再并行执行。
* join()源码分析（为描述方便，假设A线程调用B线程join方法）
```
//调用join其实就是调用带参数的join(0)。
public final void join() throws InterruptedException {
    join(0);
}
```
* 紧接着看带一个参数的join：
```
/**
 synchronized对B线程对象加锁，调用join方法首先要获取当前对象的锁，获取锁的原因是会调用B线程的wait()方法，而wait方法是必须持有B线程对象的锁才能进行调用，否则会抛出IllegalMonitorStateException。
*/
public final synchronized void join(long millis)
    throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }
    //如果等待时间=0，则直接调用B线程对象的wait(0)方法。循环检测B线程的状态，直到B线程被打断或者B线程执行完毕调用了notify方法唤醒对于B线程的等待队列线程。
    if (millis == 0) {
        while (isAlive()) {
            wait(0);
        }
    } else {
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
```

* 经过以上的源码阅读，我们发现，其实main线程调用one线程的join方法后，并不是将两个线程串行化执行，而是main线程获取到one线程的同步锁，然后调用了one线程的wait方法释放同步锁进入相对于one线程的等待队列当中。当one线程执行完毕，one线程在exit过程中会调用notify方法来唤醒main线程继续执行。
---
### 2.yield()
* 解释：一个调用yield()方法的线程告诉虚拟机它乐意让其他线程占用自己的位置。
* 注意事项：
1. Yield是一个静态的原生(native)方法
2. Yield告诉当前正在执行的线程把运行机会交给线程池中拥有相同优先级的线程。
3. Yield不能保证使得当前正在运行的线程迅速转换到可运行的状态
4. 它仅能使一个线程从运行状态转到可运行状态，而不是等待或阻塞状态。
* 代码：
```
public class YieldTest {

    public static void main(String[] args) {
        System.out.println("start to test the thread.yield.------");
        TwoThread threadA = new TwoThread("ThreadA");
        TwoThread threadB = new TwoThread("ThreadB");
        threadA.start();
        threadB.start();
        System.out.println("end to test the thread.yield.------");
    }
    static class TwoThread extends Thread {
        TwoThread(String name) {
            super(name);
        }
        @Override
        public void run() {
            for (int i = 0; i != 5; ++i) {
                System.out.println("the current thread:" +
                        Thread.currentThread().getName() + "." + i);
                //此处加入yield代表线程让出cpu
                Thread.yield();
            }
        }
    }
}
```

* 输出，加入yield
```
start to test the thread.yield.------
end to test the thread.yield.------
the current thread:ThreadA.0
the current thread:ThreadB.0
the current thread:ThreadA.1
the current thread:ThreadB.1
the current thread:ThreadA.2
the current thread:ThreadB.2
the current thread:ThreadA.3
the current thread:ThreadB.3
the current thread:ThreadA.4
the current thread:ThreadB.4
```
* 不加入yield
```
start to test the thread.yield.------
end to test the thread.yield.------
the current thread:ThreadA.0
the current thread:ThreadA.1
the current thread:ThreadA.2
the current thread:ThreadA.3
the current thread:ThreadA.4
the current thread:ThreadB.0
the current thread:ThreadB.1
the current thread:ThreadB.2
the current thread:ThreadB.3
the current thread:ThreadB.4
```
* 通过查看对应的输入输出我们可以发现，加入yield执行，当前运行线程会让出cpu时间片，然后cpu会选择其他线程执行。在本例当中，ThreadA与ThradB交替轮流执行。
