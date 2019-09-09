[TOC]

> 本文是拜读unix高级环境编程(第三版)的个人理解以及摘录，以markdown形式记录，方便日后的查阅。如果错误，请指正。

# 文件I/O

## 概述

这里描述的函数为*不带缓冲的I/O* （unbuffered I/O) ，与标准I/O相对。

*不带缓冲*是指每个read和write都调用一个系统调用，这些I/O函数不是ISO C的组成部分，但是是POSIX.1等的组成部分。

## 文件描述符

- 非负整数，打开或者创建文件时，`内核`向`进程`返回一个文件描述符。

- 文件描述符的关联：

  - 0	进程标准输入  STDIN_FILENO

  - 1    进程标准输入  STDOUT_FILENO

  - 2    标准错误  STDERR_FILENO

    以上在头文件unistd.h中定义

- 文件描述符变化范围为`0～OPEN_MAX-1`，早期是19个，现在一般增加到63个。

> Linux文件描述的变化范围几乎是无限的，受系统配置内存总变量、整型字长和管理员配置的软件限制等约束。

## open和openat 打开文件

```c
int open(const char *path , int oflag, ...);

int openat(int fd, const char *path, int oflag, ...);


```

功能： 创建或打开文件

- ... IOS C 用这种方法表示剩下的参数数量是可变的。

- path 打开或创建的文件名。

- oflag 用下列一个或者多个变量进行或运算，构成oflag参数。

  - O_RDONLY 只读  定义为0

  - O_WRONLY 只写  1

  - O_RDWR 读写  2

  - O_EXEC 只执行

  - O_EXCL 如果同时制定了O_CREAT，而且文件存在，就会出错。通常用来测试文件是否存在，如果不存在就创建。这两个操作是原子操作。
  
  - O_CREAT 如果文件不存在就创建。使用此选项，需要走致命第三个参数mode。
  
    等等等，详见函数库。

open和openat返回的文件描述是**最小的、未用的**描述符的数值。比如一个程序可以关闭标准输出（文件描述符一般为1），然后再打开文件，这个文件一定会在文件描述符为1上打开。

open和openat的区别：

- path是绝对路径，fd参数会被忽略，他们两个等价
- path是相对路径，fd指出了相对路径在文件系统中的起始地址。fd通常通过相对路径所在的目录来获取。
- path是相对路径，fd参数为特殊值`AT_FDCWD`。这时路径名在当前的工作路径区。

openat函数是POSIX.1中增加的。可以让线程使用相对路径名打开文件。在多线程中，所有线程共享当前工作目录，所以很难让不同线程同时工作在不同的目录中。同时，openat可以避免time-of-check-to-time-of-use 错误。

![1566893155487](/home/spade/Documents/markdown/linux_programing_pics/1566893155487.png)

## creat 创建文件(可用open代替)

```c
#include <fcnt1.h>
int create(const char *path,mode_t mode);
//若成功，返回文件描述，失败，返回-1
```

这个函数存在因为早期的open不能够打开一个尚未存在的函数，没有创建的功能。所以有create。现在的open提供了O_CREAT和O_TRUNC，不需要create了。

```c
open(path, O_RDWR | O_CREAT | O_TRUNC , mode );
```

- mode是文件访问权限。

## close 关闭文件

```c
#include <unistd.h>
int close (int fd);
//成功返回0，失败返回-1
```

## lseek 设置文件偏移量

### 概念

每个文件打开后都有一个`当前文件偏移量`。通常是一个非负整数，从文件开始处计算字节数，通常文件的读写都是从当前文件偏移量开始，默认情况下，除非指定的是O_APPEND选项，否则打开时偏移量都是0.

### 功能

lseek就是用来设置文件偏移量的函数。



```c
#include <unistd.h>
off_t lseek(int fd, off_t offset, int whence);
//成功:返回文件偏移量；失败返回-1
```

- whence
  - SEEK_SET 偏移量从文件开始处计算，便宜offset字节。
  - SEEK_CUR 从当前位置算，offset可以正或者负
  - SEEK_END 从最后开始算，文件长度+offset ，offset可正可负

确定当前偏移量，把offset设为0，whence为SEEK_CUR

```c
lseek(fd, 0, SEEK_CUR);
```

**如果文件是管道或者套接字、FIFO,lseek会返回1，errno设置为ESPIPE**

### 文件空洞：

创建文件，将偏移量设置大于文件长度，然后继续写文件，大于的部分就会设置为0。但是不用分配磁盘块。但是显示的大小是总体的大小。



## read 读取制定字节数

### 原型

```c
#include<unistd.h>
//iso c
ssize_t read(int fd, void *buf, size_t nbytes);
//posix.1
int read(int fd, char *buf, unsigned nbytes);
//返回读到的字节数,-1出错。0已经到文件尾
```

- buf 是缓存地址
- nbytes是读取的字节数。如果设置为100，实际上还剩30到文件末尾，就返回30

## write 写入文件

### 原型

```c
#include <unistd.h>
ssize_t write(int fd, const void *buf, size_t nbytes);
//成功返回写入的字节数，出错返回-1

```

对于普通文件，读写是从偏移量开始，如果打开文件制定了O_APPEND，在每次写操作的时候都将文件偏移量设置到结尾，并且文件偏移量相应增加。

## I/O效率

对于一个复制的程序，read进缓冲区，然后再write到另外一个文件。比如从标准输入中读，写向标准输出。

- 缓冲区太小会导致循环读取的次数变多
- 缓冲区增大到一定数值，继续增大也不会明显减少时间。磁盘块长度大小最合适。

## 文件共享

在打开一个文件的时候，操作系统一共有三个数据结构来表示这个文件。

- 在进程中，进程表有一个记录项，记录该进程打开的文件。
  - 文件描述标志
  - 指向文件表项的指针。（**指向内核中的文件表项**）
- 在内核中，为所有打开的文件维护一个文件表，每个文件表项包括：
  - 文件状态标志（读、写、填写等）
  - 当前文件偏移量
  - 指向该文件v结点表项的指针。（linux使用i结点，原理相似）
- 每个打开文件或者设备都有一个v结点，v-node。v结点包含：
  - 文件类型
  - 对文件进行各种操作的函数指针。
  - i结点（大多数unix包括），索引结点。包括文件的基本信息，长度等。

> v结点的目的是对计算机上的多文件系统类型进行支持。Sun把这种文件系统成为细腻文件系统(Virtual File System), 与文件系统无关的i结点的部分成为v结点。Linux中没有v结点，而是采用了与文件系统相关的`i结点`和与文件系统无关的`i结点`

图示，来自unix高级编程第三版。

![1567053330013](/home/spade/Documents/markdown/linux_programing_pics/1567053330013.png)

### attention

可能有多个文件描述符项fd（进程中）指向同一个文件表项。如fork后，父进程和子进程打开的每一个文件描述符共享一个文件表项（内核中）。

## 原子操作

### 定义：

atomic operation: 多步组成的一个操作，如果该操作原子执行，要么执行完所有步骤，要么一步也不执行，不可以只执行步骤中的一部分。

### 案例：

检查文件是否存在和创建文件操作是一个原子操作执行的。如果不是原子操作，检查不存在之后，这时其他进程创建了该文件，当前进程会认为文件不存在从而重新创建，导致中间写的内容丢失。

### pread 和 pwrite 原子读写

```c
#include <unistd.h>
ssize_t pread(int fd, void *buf, size_t nbytes, off_t offset);
//返回值：读取的字节数 出错返回-1
ssize_t pwrite(int fd , void *buf, size_t nbytes, off_t offset);
//成功，返回写入的字节数；失败返回-1

```

相当于先调用lseek后再read，且

- 调用pread时，无法中断他的定位和读操作。

- 不更新当前文件的偏移量

  

## dup和dup2 复制现有文件描述

### 描述

```c
#include <unistd.h>
int dup(int fd);
int dup2(int fd, int fd2);
//成功，返回新的面见描述，失败，返回-1
```

dup返回可用的最小的文件描述，dup2返回指定的文件描述fd2，dup2是原子操作

- 如果fd2已经打开，就将其关闭，
- 如果fd和fd2相等，就返回fd2不关闭

![1567055331468](/home/spade/Documents/markdown/linux_programing_pics/1567055331468.png)

## sync 、fsync、fdatasync 缓冲区和文件系统同步

传统unix没有设置高速缓存或者页高速缓存。磁盘I/O通过缓冲区进行。内核先将我们写入的数据复制到缓冲区，放入队列，然后再写入磁盘。叫做`延迟写`。sync函数是为了保证文件系统和缓冲区一致性设计的。

```c
#include<unistd.h>
int fsync(int fd);
int fdatasync(int fd);
//成功返回0，失败返回-1

void sync(void);
```

- sync 将所有修改的块缓冲区排入写队列，然后返回。
- fsync 只对指定fd文件有作用，等待磁盘操作结束返回
- fdatasync 类似fsync，不过他只影响文件的数据部分。fsync还会同步更新文件属性

## fcntl 改变已经打开的文件属性

### 原型

```c
#include <fcntl.h>
int fcntl(int fd, int cmd, ... /*int arg*/);
```

### 功能

- 复制一个已有的描述符(cmd=F_DUPFD 或者F_DUPFD_CLOEXEC)
- 获取/设置文件描述符标志(cmd=F_GETFD或F_SETFD)
- 获取/设置文件状态标志(cmd=F_GETFL或F_SETFL)
- 获取/设置异步IO所有权(cmd=F_GETOWN或F_SETOWN)
- 获取/设置记录锁(cmd=F_GETLK、F_SETLK、F_SETLKW)

## ioctl I/O操作的杂物箱

### 原型

```c
#include <unistd.h>  //system v

#include <sys/ioctl.h> //bsd and linux

int ioctl(int fd, int request , ...);
```

原型对应于POSIX.1

通常，还要求另外的设备装用文件，例如终端IO的ioctl命令都需要头文件<termios.h>

具体后叙

## /dev/fd

这个是一个目录，较新的系统都有。里面的文件是0/1/2等文件。如果n是打开的，那么打开/dev/fd/n 等效于复制描述符n

```c
fd=open("/dev/fd/0",mode);
//等效
fd=dup(0);
```

0和fd共享一个文件表项。若之前打开的0为只读，复制的fd也是只读。

> Linux 上的/dev/fd例外，它把文件描述符映射指向了底层物理文件的符号链接，打开/dev/fd/0时，打开的是与标准输入相关联的文件，因此返回的新文件描述符的模式和/dev/fd文件描述模式不相关。

也可以使用creat来调用/dev/fd，和open里面制定O_CREAT相同。

> Linux 上必须小心。Linux实现使用指向实际文件的符号链接。在/dev/fd上使用creat会导致底层文件被截断。