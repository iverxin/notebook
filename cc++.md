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

- 简化struct声明

- 将某个复杂声明进行部分替换，简化复杂的声明。

  ```c
  //声明两个max_Func_p和min_Func_p两个函数指针
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



# 异常

## 不用异常对程序出错的处理办法

### 调用abort()

如果出现异常情况，调用abort()，然后中程序，并且告诉操作系统或者他的父进程，处理失败。abort()是否刷新文件缓冲区(存储读写到文件中的数据的内存区)，取决于库的具体实现。也可以使用exit()，刷新文件缓冲区，但是不显示消息。

```c#
#include <iostream>
#include <cstdlib>
using namespace std;
double div(double a, double b);
int main(argc,char * argv[])
{
    double x, y;
    while (cin >> x >> y){
        cout << Answer is << div(x,y)<<endl;
        return 0;
    }
}
double div(double a, double b)
{
    if (b==0){
        cout << "b can't be zero!";
        abort();
    }
    return a/b;
}
```

Note: 调用abort() 直接终止程序，而不返回到main()。为了避免终止，函数在调用时要依靠程序员目测检查，这是很容易出现问题的。

### 返回错误码

```c
#include <iostream>
#include <cstdlib>
using namespace std;
bool div(double a, double b，double *ans);
int main(argc,char * argv[])
{
    double x, y, ans;
    while (cin >> x >> y){
        if(div(x,y,&ans)){
            cout<<answer is <<ans<<endl;
        }
        else
            cout << "b can't be zero!"
        
        return 0;
    }
}
bool div(double a, double b， double *ans)
{
    if (b==0){
        return false;
    }
   	*ans = a/b;
    return true;
}
```

这里引入了第三个参数指针，用来存放结果。（c++中也可以使用引用）。

通过检查返回值，来确定函数内部是否发生了异常。且不中断主程序。

也可以设置一个全局变量，如果出现问题将这个全局变量设置为特定值，然后检查该变量。例如：c库标准输入输出流中getchar到达文件末尾和出错返回都是EOF。具体是哪种情况需要使用函数来访问状态变量确定。

### 使用异常处理：

```c
#include <iostream>
#include <cstdlib>
double div(double a, double b);
int main(argc,char * argv[])
{
    double x, y, ans;
    while (cin >> x >> y){
		try{
            ans=div(x,y);
            cout<< Answer is << ans<< endl;
        }
        catch (char * e){
            cout << e << endl;
        }
    }
}
double div(double a, double b)
{
    if (b==0){
        throw "b can't be zero!!!";
    }
   	return = a/b;
}
```



## 异常机制

出错的情况使用throw抛出异常。throw语句实质是跳转。

使用try catch处理捕获异常

- 在try外面，无法处理异常
- throw将控制权返回给调用程序。

### 异常组成



### 抛出异常的类型

抛出的异常可以是任意类型，字符串、int等，但是通常抛出是异常类。异常类可以自己构建，也可以使用库中的。

#### 字符串

#### 类

```c++
class bad_div{
    private:
    	double v1;
    	double v2;
    public:
    	bad_div(int a=0, int b=0):v1(a), v2(b){}
};

inline void bad_div::mesg(){
    std::out<< v1 << "divided by" << v2 <<": v2 can't be Zero!";
}


//fun2()代码端:
if(b==0){
    throw bad_div(a, b);
}

    
//fun1
try{
    fun2();
}
catch (bad_div & bg){//注意这里
    bg.mesg();
}
```





## 异常原理和特性

### 栈退解

return:

1. C++将信息存放在栈中，处理函数调用。
2. 将调用函数的指令地址存放到栈中，调用返回后，根据栈中地址确定返回地址
3. 函数的参数也存放到栈中。返回时释放变量，如果是类，析构函数调用。

### 栈退解和return区别

### throw抛出和接收注意



## 标准类库exception

### exception

### exception派生类



## 异常迷失

异常引发之后，有两种情况会导致问题：

- 意外异常

  简单来说，如果带异常规范的函数中引发的，必须与规范列表中的某种异常匹配。如果异常不在规范列表中，成为意外异常。默认情况下会导致程序终止。

- 未捕获异常

  没有try   catch块，函数抛出异常，默认情况下，导致程序异常终止。



## 异常的注意事项

缺点：

- 增加代码量
- 降低程序运行速度
- 异常规范不适用于模板，因为模板函数引发的异常可能随着特定的具体化而异
- 异常和动态内存并非能够协同工作，可能造成`内存泄漏`

### 造成内存泄露的原因

#### 正常情况：

```c++
void test1(int n)
{
    string msg("I love c++");
    ...; //此处省略一万行代码
    if(somewrong)
        throw exception(); //抛出exception异常类
    ...; //此处省略一万行代码
    return;
}
```

上述情况，string内部采用动态内存非配，当函数结束的时候，string的析构函数会释放msg申请的内存。即使`throw`语句会提前终止函数，但是由于栈退解原理，析构函数仍然能够正确调用，内存会被正确管理。

#### 内存泄露情况：

```c++
void test2(int)
{
    double * msg = new double[n];  //动态申请内存 c常用malloc
    ... ; //此处省略一万行代码
    if (somewrong)
        throw exception(); //抛出异常
    ... ; //此处省略一万行
    delete [] msg;
    return;
}
```

`throw`过早的终止函数，意味着程序无法执行delete [] msg 。栈退解时，只是删除了栈中的指针变量 msg，而指向的内存并没有被释放掉，并且再也无法找到。这时发生了内存泄露。

#### 解决方案：

这种泄露的解决办法之一，就是在引发异常的函数中，捕获异常，处理之后再重新引发异常。

```c++
void test3(int)
{
    double * msg = new double[n];
    ... ; //此处省略一万行代码
    
    //在这里使用try catch
    try{
        if(somewrong)
            throw exception();
    }
    catch(exception & ex)  //在函数内捕获这个异常
    {
        delete [] msg ; //清除内存
        throw; //再次抛出
    }
}
```

