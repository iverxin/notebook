[TOC]

# java高并发

主要知识点：

synchronizer  / 同步容器 / ThreadPool 、executor

## synchronized() 互斥锁，原子快

多线程访问同一个资源，需要对资源加锁

加锁方法：

```java
public class T {
    private int count = 10;
    
    /*方法1*/
    private  Object lock_obj = new Object();  //申请一个object,在堆中存的对象

    public void fun1(){
        //synchronized锁定的是对象
        synchronized (lock_obj) { // 对lock_obj加锁，互斥锁。任何调用fun1的必须要取确定lock_obj是否被加锁。
            count --;
            System.out.println(Thread.currentThread().getName()+"count" +count);
        }
    }

/*方法2*/
    //锁定整个T对象
    public void fun2(){
        synchronized (this) {//简写，不用new一个对象，而是直接锁定自身这个类。
            System.out.println(Thread.currentThread().getName()+"is using");
        }
    }
    //等同于
    public synchronized void fun3() {
        System.out.println();
    }
    
    //如果是static方法，锁定的是T.class。因为静态方法没有对象可以访问
    public synchronized static void fun4(){

    }
    //等同于
    public static void fun5(){
        synchronized (T.class){

        }
    }
}
```



同一个类里，有一个函数synchronized修饰，其他的没有。当有一个线程执行加锁的程序，其他线程是否可以执行其他未加锁的？

- 可以并行。因为其他线程不需要锁。如果同时还有另一个函数是需要加锁的，则不能同时运行两个都需要加锁的函数，除非他们锁定的对象不同



## 读脏位

对于读写，如果只对写加锁，对读不加锁，可能产生脏读。实际考虑是否允许脏读



## 一个同步方法是否可以调用另外一个同步方法？

可以。函数1申请锁后，运行过程中调用函数2，函数2也需要同一把锁，按理说不能申请，因为函数1正在占用。不过由于他们来自同一个线程，是可以申请的。只不过锁从序号1变成序号2.

## 死锁：

线程1： 先锁定A，再锁定B

线程2： 先锁定B，再锁定A

当他们同时运行，线程1锁定A，申请B的锁时，B已经被线程2锁定。需要等待线程2释放。然而，线程2此时正在等待A的锁解除，出现了两个线程对峙的状态。	



循环死锁是好几个线程



## 子类同步方法可以调用父类同步方法。

```java
public T{
    public synchronzied void fun1(){
        
    }
    
    public static void main(String [] argv){
        new TT().m;
    }
}

public TT extends T{
    @Override
    public synchronzied void fun1(){
        super.m();//调用父类
    }
}
```





## 程序执行过程，出现异常锁会被释放

多进程处理同一个数据，如果出现异常一定要处理，否则直接释放锁，会造成数据写入一半





## volatile多个线程共用该变量

例如，一个变量flag为true。线程1运行while(flag)，一直运行，如果这个flag是一个volatile声明的，则线程2可以把flag这只为flase, 从而线程1也会停止。



flag内容会从主内存拷贝到线程的缓冲区内存。

如果不加volatile，线程很忙时，就不会去更新缓冲区里flag的值。如果有sleep等，cpu空闲可能取刷一下内存

如果加了volatile，当主内存的内容改变之后，会通知各个线程，你的flag过期了。需要更新。保证线程对变量可见性



## volatile和synchronized区别

volatile比synchronzied性能高很多。无锁同步。但是volatile不能取代synchronzied。只能保证可见，不能保证原子



## 多个线程同时处理volatile

多个线程同时累加一个volatile值。总和会少。有重复回写

## 原子方法

例如AtomicInteget a=new AtomicInteget(0);

a.incrementAndGet(); //a++

性能高

两个原子性的方法之间，有可能被其他线程进入改写，需要注意

AtomXXX

## synchronized锁的代码量尽量少

代码量少，粒度小。性能高



## 锁定一个object，如果这个object被重新赋值，锁失效。

```java
public Object o = new Object()
func1(){
    synchronized(o){
    xxxxxx
}   
}

//第一个线程运行，线程2是不能开始的。
//如果t.o=new Object();
//线程2可以使用，因为锁的对象换了。

```



## 不要以字符串常量作为锁定对象

```java
String s1="hello";
String s2="hello";

synchronized(s1) 和 synchronized(s2)是同一把锁。可能产生死锁。
```



## java锁位置

java锁只能锁在堆上，在堆上的对象的数据做一个记录。栈用来存放局部变量，对于对象，实际存在堆中，栈只是记录，相当于指针。指向堆的内存。





## 多线程实例

1. 实现一个容器，add和size两个方法。两个线程，线程1加十个元素放到容器中，线程2监控线程1，加到第5个时候，告诉线程1停止。

```java
public class MyContainer1 {
    volatile  List lists = new ArrayList();

    public void add(Object o) {
        lists.add(o);
    }

    public int size() {
        return lists.size();
    }

    public static void main(String[] args) {
        MyContainer1 c = new MyContainer1();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                c.add(new Object());
                System.out.println("add" + i);

                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "t1").start();

        new Thread(() -> {
            while (true) {
                if (c.size() == 5) {   //c中的list必须声明为volatile，否则检测永远是5.可能没时间刷新c，t2不能得到通知
                    break;
                }
            }
            System.out.println("t2结束");
        }, "t2").start();
    }
}
```



输出

```
add0
add1
add2
add3
add4
t2结束
add5
add6
add7
add8
add9

```

问题：

1. 线程2中，while死循环，浪费cpu。
2. 得到通知后，仍然再加1，停止的可能不精确。





### 使用wait和notify

调用被锁定对象的wait方法和被锁定对象的notify方法、

线程1先锁定对象，调用对象的wait方法，线程1进入等待状态，释放锁别线程2可以进来。线程2调用对象的notify或者 notifyAll方法叫醒在这个对象等待的线程1或者所有线程



思路：线程2启动，申请所，线程2进入等待并释放锁，线程1得到锁累加，加到第5个唤醒线程2并且线程1进入等待释放锁，线程2被唤醒得到锁处理后，唤醒县线程1结束线程2并释放锁。线程1拿到锁继续。

```java

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeUnit;

public class MyContainer1 {
    volatile  List lists = new ArrayList();

    public void add(Object o) {
        lists.add(o);
    }

    public int size() {
        return lists.size();
    }

    public static void main(String[] args) {
        MyContainer1 c = new MyContainer1();

        final Object lock = new Object();

        //线程2启动并等待
        new Thread(()-> {
            synchronized (lock) {
                System.out.println("t2启动");
                if (c.size() != 5) {
                    try {
                        System.out.println("t2 is waiting");
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println("t2 结束");

                //唤醒等待的t1
                try{
                    lock.notify();
                } catch (Exception e) {
                    e.printStackTrace();
                }

            }
        },"t2").start();
		
        
        //线程1启动
        new Thread(()->{
            System.out.println("t1 启动");
            synchronized (lock){
                for (int i=0; i<10; i++){
                    c.add(new Object());
                    System.out.println("add"+i);
                    if(c.size()==5) {
                        //notify不能指定线程，由系统制定
                        lock.notify(); //notify不会释放锁. sleep 也不释放锁，只有wait释放锁
                        //必须要加让该线程wait,否则t2无法获得锁
                        try{
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    try{
                        TimeUnit.SECONDS.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        },"t1").start();
    }
}
```



不涉及同步，只涉及进程通讯的时候，使用synchronized和wait+notify太重了，可以考虑**countdownlatch/cyclicbarrier/semaphore** ——门栓

```java
    public static void main(String[] args) {
        MyContainer1 c = new MyContainer1();

        //CountDownLatch 往下数，如果1变为0，门就开了。
        CountDownLatch latch = new CountDownLatch(1);

        new Thread(()->{
            System.out.println("t2启动");
            if(c.size()!=5){
                try{
                    latch.await(); //门栓等待，当门栓没了就能开了。
                    //也可以制定等待时间
                    //latch.await(5000,TimeUnit.MILLISECONDS);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("t2结束");
        },"t2").start();

        new Thread(()->{
            System.out.println("t1 start");
            for(int i=0; i<10; i++){
                c.add(new Object());
                System.out.println("add"+i);

                if(c.size() == 5){
                    //打开门栓，让t2执行
                    latch.countDown(); //调用1次减1.门栓打开了之后t1继续运行.
                }
                try{
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"t1").start();
```



## ReentrantLock 手动所

用于代替synchronized

手动锁，必须要手动释放 

synchronized出现异常jvm会处理释放。但是使用reentrantLock必须手动释放

使用

```java
Lock lock=new ReentrantLock(); 

try{
    lock.lock();   //synchronized(this);
}catch{
    
}catch{
    
}finally{
    lock.unlock();
}
```

区别：

使用ReentranLock可以进行“尝试锁定”，这样无法锁定或者在一定时间内无法锁定时，线程可以决定是否继续等待。boolean flag=lock.tryLock();

lock.tryLock(5,TimeUnit.SECONDS); //等待5秒

synchronized就死等，等不到一直等。且synchronized是非公平锁。结束之后随便抢，不一定谁。这样jvm调度效率高

公平锁：private static ReentranLock lock = new ReentranLock(true);  没有参数是非公平，有true是公平。谁等时间长谁上。



ReentranLock比较灵活。

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ReentranLock1 {
    public static void main(String [] args){

        Lock lock = new ReentrantLock();

        Thread t1= new Thread(()->{
            try {
                lock.lock();
                System.out.println("t1 start");
                TimeUnit.SECONDS.sleep(Integer.MAX_VALUE);
                System.out.println("t1 end");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        },"t1");
        t1.start();

        Thread t2 = new Thread(()->{
            try{
                //lock.lock(); //会一直等待，其他线程不能打断，必须等待
                //lock.tryLock(); //尝试锁
                lock.lockInterruptibly();//可打断的，等待锁时可以被其他线程打断
                System.out.println("t2 start");

            } catch (InterruptedException e) {
                e.printStackTrace();
                System.out.println("interputed!");
            }finally {
                lock.unlock();
            }
        });
        t2.start();

        try{
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        t2.interrupt();
    }
}
```

