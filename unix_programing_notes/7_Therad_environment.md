[TOC]



# 进程环境

## 进程的终止方式

1. main返回
2. 调用exit
3. 调用\_exit 或 _ Exit
4. 最后一个线程从启动例程返回
5. 最后一个线程调用p\_thrad_exit 
6. 异常终止
   - 调用abort
   - 接到信号
   - 最后一个线程对取消请求做出响应

### exit系列退出函数

```c
//stdlib.h iso c声明
void exit(int status);
void _Exit(int status);

//unistd.h  posix.1声明
void _exit(int status);
```

- 区别：
  
- exit会执行标准IO库的清理关闭操作，对所有流fclose，缓冲区会冲洗，写到文件上。
  
- 参数status：`终止状态`。大多数系统shell会提供检查终止状态的方法（例如，echo $? 是输出刚才执行的程序的终止状态）。当然也可以不定义终止状态。

  对于main，return(0);和exit(0); 是等价的。$? 是0



### 进程登记函数atexit

atexit登记的函数，将由exit自动调用，所以这些函数成为`终止处理程序(exit handler)`。

```c
//stdlib.h
int atexit(void (*func)(void));
//成功0，失败负
```

atexit的参数是一个函数地址。这个函数无参数，也无返回参数。exit调用的顺序与登记的顺序**相反**,登记多次可以调用多次。return也会调用这些函数。

自己写内存模型时可以设计内存清理函数，让exit时调用



<img src="/home/spade/Documents/markdown/linux_programing_pics/7/1568626450349.png" alt="1568626450349" style="zoom:150%;" />

- 内核运行程序的唯一方法是调用exec函数

- 进程自愿终止的方法是显示或者隐式调用_exit或\_Exit。
- 进程可以使用信号非自愿终止



## 环境表

`环境表`是一个字符指针数组

```c
extern char **environ;
```

例如，某个环境有5个字符串，那么environ是一个数组指针，environ[0-4]是这5个字符串的值，其实和char **argv一个道理。

![1568626985133](/home/spade/Documents/markdown/linux_programing_pics/7/1568626985133.png)

## 存储空间布局

c编译之后的主要组成部分

- 正文段  机器指令部分。通常只读，防止恶意修改

- 初始化数据段   编写时已经赋值的变量。

- 未初始化数据段  也叫bbs段（block started by symbol） 执行之前内核将这里的数据初始化为0或者空指针。通常是函数外的没有赋值的声明。

- 栈   

  - 自动变量
  - 每次函数调用保存的信息
  - 函数调用返回地址和环境信息
  - 当前调用运行的函数里面的变量或者临时分配的存储空间

- 堆  通常是童泰存储分配在堆里。 堆位于未初始化数据段好栈之间。

  ![1568627454119](/home/spade/.config/Typora/typora-user-images/1568627454119.png)

(典型安排，不一定都这样)

### size命令

size命令可以查看各个段的内存安排

```bash
spade@spade-PC:~$ size a.out 
   text	   data	    bss	    dec	    hex	filename
   1944	    616	      8	   2568	    a08	a.out

```

a