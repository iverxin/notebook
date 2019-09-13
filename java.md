[TOC]

# java高并发

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







