[TOC]

# java高并发



主要知识点：

synchronizer  / 同步容器 / ThreadPool 、executor

# Synchronizer

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



## ReentrantLock 手动锁

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

## Java生产工作消费者模式

写一个容器，有put和get的方法。getCount记录对象。要求容器是同步容器，多线程能够同时访问不能出问题。 

如果容器已经满，put线程需要等待；

如果容器已经空了，get的线程需要等待；



解决方案：

1. 使用wait notify组合
   - 使用while而不是if判断是否满了。如果容器已经满了，有两个生产中者在等待。当有一个空闲出现时，生产者线程2争取到权限执行之后填满，线程1仍然要检查一下是否满了，不可以醒过来直接添加。
   - 使用notifyAll而不是notify，如果满了，这时候只notify唤醒的是等待的生产中而不是消费者，这个生产者上来直接wait，那么就尴尬了。整个程序停止了。




```java
package javaM;

import java.util.LinkedList;

public class MyContainer2<T> {
    final private LinkedList<T> lists = new LinkedList<>();
    final private int MAX = 10 ; //不可以被修改
    private int count = 0;

    public synchronized void put(T a){
        //注意这里要使用
        while(lists.size()==MAX){ //满了就wait
            try{
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        lists.add(a);
        count++;
        this.notifyAll(); //通知消费者
    }

    public synchronized T get(){
        T t = null;
        while(lists.size()==0) {
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        t=lists.removeFirst();
        count--;
        this.notifyAll(); //通知生产者
        return t ;
    }

    public static void main(String [] args){
        MyContainer2<String> c = new MyContainer2<>();
        //启动消费者程序
        for(int i=0; i<10; i++){
            new Thread(()->{
                for(int j=0; j<5; j++)
                    System.out.println("c "+c.get());
            },"c"+i).start();
        }

        for(int i=0; i<10; i++){
            new Thread(()->{
                for(int j=0; j<5; j++){
                    //System.out.println("add");
                    c.put(Thread.currentThread().getName()+" "+j);
                }
            },"p"+i).start();
        }

    }

}

```

2. 使用condition。使用ReentrainLock

   ```java
   package javaM;
   
   import java.util.LinkedList;
   import java.util.concurrent.locks.Condition;
   import java.util.concurrent.locks.Lock;
   import java.util.concurrent.locks.ReentrantLock;
   
   public class MyContainer3<T> {
       final private LinkedList<T> lists = new LinkedList<>();
       final private int MAX = 10; //不可以被修改
       private int count = 0;
   
       private Lock lock = new ReentrantLock();
       private Condition producer = lock.newCondition();
       private Condition consumer = lock.newCondition();
   
       public void put(T a) {
           try {
               lock.lock();
               while (lists.size() == MAX) {
                   producer.await(); //producer这个条件等待
               }
   
               lists.add(a);
               count++;
               consumer.signalAll();  //consumer这个条件进程唤醒
           } catch (InterruptedException e) {
               e.printStackTrace();
           } finally { // 手动锁不要忘记解锁
               lock.unlock();
           }
       }
   
       public T get() {
           T t = null;
           try {
               lock.lock();
               while (lists.size() == 0) {
                   consumer.await();
               }
               t = lists.removeFirst();
               count--;
               producer.signalAll();//唤醒所有生产者进程
           } catch (InterruptedException e) {
               e.printStackTrace();
           } finally {
               lock.unlock();
   
           }
           return t;
       }
       public static void main(String [] args){
           MyContainer2<String> c = new MyContainer2<>();
           //启动消费者程序
           for(int i=0; i<10; i++){
               new Thread(()->{
                   for(int j=0; j<5; j++)
                       System.out.println("c "+c.get());
               },"c"+i).start();
           }
   
           for(int i=0; i<10; i++){
               new Thread(()->{
                   for(int j=0; j<5; j++){
                       //System.out.println("add");
                       c.put(Thread.currentThread().getName()+" "+j);
                   }
               },"p"+i).start();
           }
   
       }
   
   }
   ```

   



## final关键字：

1. 修饰类

   该类不能够被继承。该类的成员变量可以根据需要设置final，但是方法都被隐式设置为final

2. 修饰方法

   final修饰的方法不能够被重写。private方法隐式制定为final

3. 修饰成员变量

   - 必须初始化值，要么赋值初始化，要么构造初始化
   - 如果是基本类型，如int、char等，存放在栈中，表示这个变量不能改变
   - 如果是引用类型，如String，Integer等类，地址存放在栈中，引用地址不可改变，但是指向的内容是可以改变的。
   - 

## 线程局部变量 ThreadLocal



例如：

```java
static ThreadLocal<Person> tl = new ThreadLocal<>();
```

每个线程有自己的变量，互相不影响。  相当于拷贝一份。可能导致内存泄露





# 同步容器

## 单例模式：

无论如何只有一个对象

4. 内部类





<img src="/home/spade/Documents/markdown/java_pics/1568523718714.png" alt="1568523718714" style="zoom:150%;" />



可能卖重了、买多了等



<img src="/home/spade/Documents/markdown/java_pics/1568523816266.png" alt="1568523816266" style="zoom:150%;" />

容器换成Vector后，由于vector是同步容器，所以不需要加锁了。但是，虽然size（）和remove()都是原子的, 但是判断和执行分离了。两个原子操作之间分离，可能被别的线程插入。



<img src="/home/spade/Documents/markdown/java_pics/1568524042116.png" alt="1568524042116" style="zoom:150%;" />

判断和操作都加锁，可以。但是效率太低。



**使用同步容器**

<img src="/home/spade/Documents/markdown/java_pics/1568524146866.png" alt="1568524146866" style="zoom:150%;" />

tickets.poll 从头拿出来，如果没拿到返回null

虽然判断和取分离了。在这里仍然没问题，因为现在先拿出来然后再判断。

## ConcurrentHashMap等

> Map可以换成Set  只有key

加锁的

- ConcurrentHashMap 并发HashMap，默认分成16段，1.8后改了。不同段之间可以并发加入。  600多
- ConcurrentSkipListMap 高并发并且排序。插入效率低一些，但是查询块。1000
- Hashtable 默认上锁的，但是效率有点低，现在很少使用  所有线程使用都上锁。748

不加锁的

- HashMap 默认不加锁。可以往上加锁。Collections.synchronizedXXX()   传入容器，进行包装之后返回一个带锁的。List、Map等
- <img src="/home/spade/Documents/markdown/java_pics/1568525893012.png" alt="1568525893012" style="zoom:150%;" />
- TreeMap 默认排好顺序

<img src="/home/spade/Documents/markdown/java_pics/1568524676277.png" alt="1568524676277" style="zoom:150%;" />

## 写时复制CopyOnWrite

写的时候赋值一份，然后更改引用。读时直接读。

**读效率高，写效率低**

- CopyOnWriteArrayList()写最慢 500+.用于写的很少，但是读的特别多的情况
- ArrayList 没有锁，在写的时候可能出现覆盖。
- Vector 没问题。读写均衡  100

![1568525650321](/home/spade/Documents/markdown/java_pics/1568525650321.png)



## Queue

### ConcurrentLinkedQueue

```java
Queue<String> strs=new ConcurrentLinkedQueue<>();//无限队列
strs.offer("a"+i); //相当于add，但是add失败会抛出异常，offer返回布尔类型可以判断。
strs.poll(); //从头上娶一个并且删除
strs.peek(); //从头拿不删

```

Queue高并发有两种

ConcurrentLinkedQueue

### BlockingQueue 阻塞式容器

- put如果满了就会等待
- take如果空了就会等待。

可以写出阻塞的生产者消费者模式。

![1568526879243](/home/spade/Documents/markdown/java_pics/1568526879243.png)



有界容器

put //满了就会等待，程序阻塞，一直等待。

add //满了抛异常

offer //返回boolean类型，自己判断. 可以设置阻塞时间

<img src="/home/spade/Documents/markdown/java_pics/1568526939762.png" alt="1568526939762" style="zoom:150%;" />

### DelayQueue

每个元素记录我还有多少时间可以从队列拿走。是排好顺序的。

在加内容时需要实现接口Delay接口

可以执行定时任务。



### TransferQueue

生产者生产出后如果有消费者，就直接给消费者不放入队列了。用在更高的并发。没有消费者就会阻塞。

用add和put等都不会阻塞。用transfer会一直等待处理。transfer不放队列就等待消费



### SynchronusQueue

容量为0.不能调用add。只能调用put，put在这里是阻塞等待消费者来消费。其实put里面用的是transfer



# 线程池

## Interface

### Interface：Executer 执行器

接口里定义了一个执行 public void execute(Runnalbe command)方法

顶级接口

### interface：ExecutorService  

所有的线程池都实现了这个接口。

submit可以执行有返回值，也可以执行无返回值的

execute 执行runable方法。没有返回值的。

继承Executer接口



### Interface：Callable & Runnable

Runnable里面有run方法， Callable接口有call方法

区别：run不返回值。不能抛出异常。 call有返回值，能抛出异常

### class： Executors

工具类, 操作Executor ,ExecutorService等



## 线程池类型

### FixedThreadPool 定量的线程池

<img src="/home/spade/Documents/markdown/java_pics/1568609026099.png" alt="1568609026099" style="zoom:150%;" />

一共启动6个线程，但是线程池的容量是5。第6个来的时候，在线程池维护的队列中等待，如果有线程空闲下来，线程不用停止重启，直接完成第6个的任务，而不是启动第6个线程。效率较高，线程重用（不用新人，一直用前5个）

**一个线程维护两个队列**

- 等待队列
- 完成队列

service.shutdown() 正常关闭，等待所有的进程都完成工作。

shutNow（）直接关

service.isTerminated 是否执行完任务

isShutdown() 是否正在关闭，或者是否给关闭命令

### Future类型

FutureTask

<img src="/home/spade/Documents/markdown/java_pics/1568610891328.png" alt="1568610891328" style="zoom:150%;" />



### 应用 并行计算：





### newCatchThreadPool

如果有空闲，就交给空闲的线程，如果没有，就启动新的。一直到系统能支撑的数量。线程默认超过60s会自动结束。可以自己制定。



<img src="/home/spade/Documents/markdown/java_pics/1568614722247.png" alt="1568614722247" style="zoom:150%;" />

### newSingleThreadExecutor

只有一个线程



### newSchedulePoolThreadPool

<img src="/home/spade/Documents/markdown/java_pics/1568614965699.png" alt="1568614965699" style="zoom:150%;" />

每隔500毫秒执行。线程可以复用。定时执行



### newWorkStealingPool
每个线程维护一个任务列表，当自己的任务完成之后，会去别的线程的任务列表里偷任务。

<img src="/home/spade/Documents/markdown/java_pics/1568615367711.png" alt="1568615367711" style="zoom:150%;" />

是封装了ForkJoinPool

启动的线程是Daemon线程（精灵线程，守护线程）。运行在后台，具体自己查



### newForkJoinPool

<img src="/home/spade/Documents/markdown/java_pics/1568615599826.png" alt="1568615599826" style="zoom:150%;" />



<img src="/home/spade/Documents/markdown/java_pics/1568616267596.png" alt="1568616267596" style="zoom:150%;" />



先把大任务切成小任务，小任务还可以接着分（fork）。最后汇总（Join)。

RecursiveAction没有返回值，不能汇总

注释的RecursiveTask有返回值，可以join回来

其实是递归。只是有线程池维护 

常用大规模的数据计算

![1568616470668](/home/spade/Documents/markdown/java_pics/1568616470668.png)

## 使用parallelStream

计算产生的随机数是否是质数.例子



<img src="/home/spade/Documents/markdown/java_pics/1568617032930.png" alt="1568617032930" style="zoom:150%;" />