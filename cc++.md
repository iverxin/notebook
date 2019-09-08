[TOC]

## 函数指针

``` c
#include <stdio.h>
#include <stdlib.h>

void func(){
    printf("here is the function!\n");
}

int main()
{
    printf("Hello world!\n");
    //声明funp 是一个函数指针，形参为void
	void (*funp)(void); //注意括号不能省略，否则被认为返回值是void* 的函数
    //把函数赋值给函数指针。func看起来像一个指针，但是如果用(*func)() 就会报错
    funp=func;
    //函数指针在使用的时候要用(*funp),否则会认为是*(funp*())
    (*funp)();
    return 0;
}

```

输出：

```bash
Hello world!
here is the function!
```

## typedef 的四个用法

- 定义类型别名

  - 

  - 

   

- 简化struct声明

- 将某个复杂声明进行部分替换，简化复杂的声明。

  ```c
  //声明两个max_Func_p和min_Func_p两个哈数指针
  int (*max_Func_p)(int,int);
  int (*min_Func_p)(int,int);
  //简化
  typedef (*Func_p)(int,int);
  func_p max_Func_p;
  func_p min_Func_p;
  ```

  ### typedef陷阱

  **第二、两大陷阱**

  **陷阱一：**

  记住，typedef是定义了一种类型的新别名，不同于宏，它不是简单的字符串替换。比如：
   先定义：
   typedef char* PSTR;
   然后：
   int mystrcmp(const PSTR, const PSTR);

  const PSTR实际上相当于const char*吗？不是的，它实际上相当于char* const。
   原因在于const给予了整个指针本身以常量性，也就是形成了常量指针char* const。
   简单来说，记住当const和typedef一起出现时，typedef不会是简单的字符串替换就行。

  **陷阱二：**

  typedef在语法上是一个存储类的关键字（如auto、extern、mutable、static、register等一样），虽然它并不真正影响对象的存储特性，如：
   typedef static int INT2; //不可行
   编译将失败，会提示“指定了一个以上的存储类”。

  > 参考材料:http://blog.sina.com.cn/s/blog_4826f7970100074k.html  作者：赤龙



## struct 里面放函数

```c
#include <stdio.h>
#include <stdlib.h>

void func(){
    printf("here is the function!\n");
}

typedef void (*funType)(void);

typedef struct A A_t;

struct A {
  funType funp;
};

int main()
{
    printf("Hello world!\n");
    A_t test; //test相当于类，里面的函数是公有函数
    test.funp=func;
    test.funp();
    //(*test.funp)()也可以
    return 0;
}

```



## void * p

p是一个无类型的指针，任何类型的指针都可以赋值给p。(当a赋值给其他类型时，需要先进行类型转换,gcc测试会正常运行，即使把char类型给int类型指针也没报错，只是警告。)

在c++中任何指针可以给void，但是不同类型之间必须显示转化指针。否则报错。



## int * func( int,int )

这类函数返回的是int型的指针



## 指针类型

所有的指针所占空间都一样,32位系统是4bytes，64位系统是8bytes。

那为什么声明指针类型呢？

指针类型绝对了按照这个指针，一次需要取多少bytes的数据。例如，int型数据所占4字节，那么按照int型指针一次取的数据便是4字节。short数据所占为2字节，如果用short型指针去取int值，那么只能取出来2哥字节。





## int main(int argc , char **argv)

- argc 传入参数的数量

- **argv 字符串数组。

  - argv[0]是文件地址, 字符串

    如何理解**argv

    argv 是指针数组，argv[2]指向一个char型的数组，也就是字符串

    argv是指针数组开始地址，申请一块内存，每个数组里存放一个指针，指向不同的字符串。

    *argv 是数组里的第一个，是一个地址。

    ** argv 是数组里第一个地址指向的内存，是string

  
  
  ## volatile 关键字
  
  volatile是易变的。意味这改变量时刻都有可能发生改变，每次读取变量的时候需要从内存中读取出来。
  
  告诉编译器不要对这个变量进行优化。编译器在优化时肯能将该变量放在寄存器中，这样在修改内存的值时寄存器可能没有及时更新。
  
  > 如果一个寄存器或者变量表示一个端口或者多个线程的共享数据，就容易出错，所以volatile可以保证对特殊地址的稳定访问。



## strlen( (const char *) s)

计算字符串s的长度，'ABCD' 返回4 。

如果想要把字符串存储到数组，则申请的地址大小应该是

s=alloc(strlen('ABCD')+1) //因为s[4]应该存放 \0



## restrict 

告诉编译器，对象已经被指针所引用，不能通过该指针以外的其他直接或者简介的方式修改对象的内容

可以帮助编译器更好的优化代码，生成更有效的汇编代码。

修饰形参时，可以要求传入的指针有效。



