#### Java线程的6种状态

[TOC]

##### 6种状态概述

```java
public static enum State {
    NEW,
    RUNNABLE,
    BLOCKED,
    WAITING,
    TIMED_WAITING,
    TERMINATED;

    private State() {
    }
}
```

###### 1.NEW

```java
public static void main(String[] args) {
        Thread thread = new Thread(()->{});
        // 输出：NEW　创建了线程而并没有调用start()方法，此时线程处于NEW状态
        System.out.println(thread.getState());
        thread.start();
    }
```

###### 2.RUNNABLE

表示当前线程正在运行中。处于RUNNABLE状态的线程在Java虚拟机中运行，也有可能在等待CPU分配资源。故**RUNNABLE**状态其实是包括了传统操作系统线程的**就绪ready**和**运行running**两个状态的。

```java
    public static void main(String[] args) {
        Thread thread = new Thread(()->{});
        // 输出：NEW　创建了线程而并没有调用start()方法，此时线程处于NEW状态
        System.out.println(thread.getState());
        thread.start();
        // 输出：RUNNABLE
        System.out.println(thread.getState());
    }
```

###### 3.BLOCKED

阻塞状态。处于BLOCKED状态的线程正等待锁的释放以进入同步区。

<Font Color="Green">通俗小栗子：</Font>

> 假如今天你下班后准备去食堂吃饭。你来到食堂仅有的一个窗口，发现前面已经有个人在窗口前了，此时你必须得等前面的人从窗口离开才行。
> 假设你是线程t2，你前面的那个人是线程t1。此时t1占有了锁（食堂唯一的窗口），t2正在等待锁的释放，所以此时t2就处于BLOCKED状态。

###### 4.WAITING

等待状态。处于等待状态的线程变成RUNNABLE状态需要其他线程唤醒。

使线程进入等待状态需要调用以下3个方法：

- Object.wait()：使当前线程处于等待状态直到另一个线程唤醒它；
- Thread.join()：等待线程执行完毕，底层调用的是Object实例的wait方法；
- LockSupport.park()：除非获得调用许可，否则禁用当前线程进行线程调度。



<Font Color="Green">通俗小栗子：</Font>

> 你等了好几分钟现在终于轮到你了，突然你们有一个“不懂事”的经理突然来了。你看到他你就有一种不祥的预感，果然，他是来找你的。
>
> 他把你拉到一旁叫你待会儿再吃饭，说他下午要去作报告，赶紧来找你了解一下项目的情况。你心里虽然有一万个不愿意但是你还是从食堂窗口走开了。
>
> 此时，假设你还是线程t2，你的经理是线程t1。虽然你此时都占有锁（窗口）了，“不速之客”来了你还是得释放掉锁。此时你t2的状态就是WAITING。然后经理t1获得锁，进入RUNNABLE状态。
>
> 要是经理t1不主动唤醒你t2（notify、notifyAll..），可以说你t2只能一直等待了。



###### 5.TIMED_WAITING

超时等待状态。

调用如下方法会使线程进入超时等待状态：

- Thread.sleep(long millis)：使当前线程睡眠指定时间；
- Object.wait(long timeout)：线程休眠指定时间，等待期间可以通过notify()/notifyAll()唤醒；
- Thread.join(long millis)：等待当前线程最多执行millis毫秒，如果millis为0，则会一直执行；
- LockSupport.parkNanos(long nanos)： 除非获得调用许可，否则禁用当前线程进行线程调度指定时间；
- LockSupport.parkUntil(long deadline)：同上，也是禁止线程进行调度指定时间；

<Font Color="Green">通俗小栗子：</Font>

> 到了第二天中午，又到了饭点，你还是到了窗口前。
>
> 突然间想起你的同事叫你等他一起，他说让你等他十分钟他改个bug。
>
> 好吧，你说那你就等等吧，你就离开了窗口。很快十分钟过去了，你见他还没来，你想都等了这么久了还不来，那你还是先去吃饭好了。
>
> 这时你还是线程t1，你改bug的同事是线程t2。t2让t1等待了指定时间，此时t1等待期间就属于TIMED_WATING状态。
>
> t1等待10分钟后，就自动唤醒，拥有了去争夺锁的资格。





###### 6.TERMINATED

终止状态。此时线程已执行完毕。

```java
    public static void main(String[] args) {
        Thread thread = new Thread(()->{
            System.out.println("子线程输出state:");
        });
        // 输出：NEW　创建了线程而并没有调用start()方法，此时线程处于NEW状态
        System.out.println(thread.getState());
        
        thread.start();
        // 输出：RUNNABLE
        System.out.println(thread.getState());
        
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        // 输出：TERMINATED
        System.out.println(thread.getState());
    }
```



###### 线程状态的转换

![线程状态转换图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/thread/thread-state-and-method-18f0d338-1c19-4e18-a0cc-62e97fc39272.png)



##### 线程状态的转换

###### 1.BLOCKED与RUNNABLE状态的转换

处于BLOCKED状态的线程是因为在等待锁的释放！

```java
public class TestBLocked {
    static Thread threadA = new Thread(new Runnable() {
        @Override
        public void run() {
            testMethod("threadA");
        }
    }, "threadA");


    static Thread threadB = new Thread(new Runnable() {
        @Override
        public void run() {
            testMethod("threadB");
        }
    }, "threadB");
    public static void main(String[] args) throws InterruptedException {

        threadA.start();
        System.out.println(threadA.getName() + ":" + threadA.getState());  //　RUNNABLE
        //Thread.sleep(1000L); // 需要注意这里main线程休眠了1000毫秒，而testMethod()里休眠了2000毫秒
        threadB.start();
        System.out.println(threadA.getName() + ":" + threadA.getState()); // TIMED_WAITING
        System.out.println(threadB.getName() + ":" + threadB.getState()+" "); //　BLOCKED

    }

    //　同步方法争夺锁
    private static synchronized void testMethod(String name) {
        try {
            Thread.sleep(2000L);
            //　A 线程调用该方法输出B状态:RUNNABLE
            //　B 线程调用该方法输出B状态:RUNNABLE
            System.out.println(threadB.getName() + ":" + threadB.getState());
            System.out.println("执行完毕啦！");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

ThreadA的转换过程：RUNNABLE（`a.start()`） -> TIMED_WATING（`Thread.sleep()`）->RUNABLE（sleep()时间到）->*BLOCKED(未抢到锁)* -> TERMINATED

ThreadB的转换过程：RUNNABLE（`b.start()`) -> *BLOCKED(未抢到锁)* ->RUNABLE（抢到锁了）TIMED_WATING（`Thread.sleep()`）->TERMINATED

*注：斜体表示可能出现的状态*

###### 2.WAITING状态与RUNNABLE状态的转换

此处主要以Object.wait()和Thread.join()为例

**Object.wait()**

- 调用wait()方法**前**线程必须持有对象的锁。

- 线程调用wait()方法时，会释放当前的锁，直到有其他线程调用notify()/notifyAll()方法唤醒等待锁的线程。

<Font Color="Red">需要注意的是</Font>，其他线程调用notify()方法只会唤醒单个等待锁的线程，如有有多个线程都在等待这个锁的话不一定会唤醒到之前调用wait()方法的线程。

同样，调用notifyAll()方法唤醒所有等待锁的线程之后，也不一定会马上把时间片分给刚才放弃锁的那个线程，具体要看系统的调度。

**Thread.join()**

> 调用join()方法，会一直等待这个线程执行完毕（即这个线程转换为TERMINATED状态）。

```java
public static void main(String[] args) throws InterruptedException {
        threadA.start();
        threadA.join(); //调用join()方法，会一直等待这个线程执行完毕
        threadB.start();
        System.out.println(threadA.getName() + ":" + threadA.getState()); // 输出 TERMINATED
        System.out.println(threadB.getName() + ":" + threadB.getState()); // 输出 RUNNABLE/TIMED_WAITING 均有可能
    }
```

若没有调用join方法，main线程不管a线程是否执行完毕都会继续往下走

###### 3.TIMED_WAITING与RUNNABLE状态转换

TIMED_WAITING状态需要指定所等待的时间

**Thread.sleep(long)**

> 使当前线程睡眠指定时间，<Font Color = "Red">需要注意这里的“睡眠”只是暂时使线程停止执行，并不会释放锁。</Font>
>
> 时间到后，线程会重新进入RUNNABLE状态。

**Object.wait(long)**

> 与wait()相同的地方：都可以通过其他线程调用notify()或notifyAll()方法来唤醒
>
> 不同的地方：有参方法wait(long)就算其他线程不来唤醒它，经过指定时间long后它会自动唤醒，拥有去争夺锁的资格。

**Thread.join(long)**

```java
    public static void main(String[] args) throws InterruptedException {
        threadA.start();
        threadA.join(1000L); //这里参数为1000,是因为指定了具体a线程执行的时间的，并且执行时间是小于a线程sleep的时间(2000)，所以a线程状态输出TIMED_WAITING
        // threadA.join(5000L); //如若参数为5000,时间>a线程执行的时间，故a线程执行完了，a线程输出状态为TERMINATED
        threadB.start();
        System.out.println(threadA.getName() + ":" + threadA.getState()); // 输出 TIMED_WAITING
        System.out.println(threadB.getName() + ":" + threadB.getState()); // 输出 RUNNABLE/BLOCKED
    }
```



###### 线程中断

通过中断操作并不能直接终止一个线程，而是通知需要被中断的线程自行处理。

关于线程中断的几个方法：

- Thread.interrupt()：中断线程。这里的中断线程并不会立即停止线程，而是设置线程的中断状态为true（默认是flase）；
- Thread.currentThread().isInterrupted()：测试当前线程是否被中断。线程的中断状态受这个方法的影响，意思是调用一次使线程中断状态设置为true，连续调用两次会使得这个线程的中断状态重新转为false；
- Thread.isInterrupted()：测试当前线程是否被中断。与上面方法不同的是调用这个方法并不会影响线程的中断状态。

------



###### **关于start()的两个引申问题**

1. 反复调用同一个线程的start()方法是否可行？
2. 假如一个线程执行完毕（此时处于TERMINATED状态），再次调用这个线程的start()方法是否可行？

答：**都是不可行**，在调用一次start()之后，threadStatus的值会改变（threadStatus !=0），此时再次调用start()方法会抛出IllegalThreadStateException异常。比如，threadStatus为2代表当前线程状态为TERMINATED。

```java
//　改变状态时，调用的源码
public static Thread.State toThreadState(int var0) {
        if ((var0 & 4) != 0) {
            return State.RUNNABLE;
        } else if ((var0 & 1024) != 0) {
            return State.BLOCKED;
        } else if ((var0 & 16) != 0) {
            return State.WAITING;
        } else if ((var0 & 32) != 0) {
            return State.TIMED_WAITING;
        } else if ((var0 & 2) != 0) {
            return State.TERMINATED;
        } else {
            return (var0 & 1) == 0 ? State.NEW : State.RUNNABLE;
        }
    }
```



###### 操作系统中的线程状态切换

操作系统线程主要有以下三个状态：

- 就绪状态(ready)：线程正在等待使用CPU，经调度程序调用之后可进入running状态。
- 执行状态(running)：线程正在使用CPU。
- 等待状态(waiting): 线程经过等待事件的调用或者正在等待其他资源（如I/O）。

![系统进程/线程转换图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/thread/thread-state-and-method-f60caaad-ad47-4edc-8d0a-ab736c2e8500.png)
