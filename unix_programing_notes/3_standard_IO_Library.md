[TOC]

## 流和FILE对象

标准IO库由ISO C标准说明，后来Single UNIX Specification进行了补充。

I/O函数是围绕文件描述符进行的，打开一个文件，返回描述符，该文件描述符用于之后的IO操作。

对于标准IO库，操作是围绕`流(stream)`进行的。使用IO库打开或者创建文件，就使一个`流`和文件相关联。

### 关于字节定向

`标准IO文件流`可用于`单字节`或者`多字节`的字符集。`流的定向(stram's orientation)`决定了读写字符是单字节还是多字节的。最初创建的流是没有定向的。如果为定向的流上使用一个多字节IO函数(wchar.h)，那么该流就被设置为宽定向的(多字节)。第一次使用的单字节函数，那么就会被设置为单字节定向的。

## fwide设置流的定向

```c
//stdio.h
//wchar.h
int fwide(FILE *fp, int mode);
//宽定向，返回正值，字节定向，返回负值；为定向返回0
```

- mode参数为负，指定为字节定向。
- mode 为0，不设置流的定向，但是返回标识该流定向的值。

**注意** fwide并不改变已经定向的流。fwide也无出错返回。唯一可依靠的是调用fwide前清除errno，然后从fwide返回时去检查errno的值。确定fwide是否出错。



打开一个流，标准IO库fopen返回一个FILE对象的指针，包含了管理该流的所有信息。

文件指针就是FILE * f 中的f

## 标准输入、标准输出和标准错误

和文件描述中STDIN_FILENO、STDOUT_FILENO和STDERR_FILENO所引用相同

通过预定义的文件指针stdin、stdout和stderr加以引用。在stdio.h中定义。

## 缓冲

标准库提供缓冲目的是尽量减少read和write的调用此书。之前讨论过缓冲大小和调用次数以及性能的关系。

标准IO库提供的3中缓冲类型：

- 全缓冲，填满IO缓冲区才进行实际IO操作。对于驻留在磁盘上的文件通常实施全缓冲
  - 冲洗（flush）：
- 行缓冲，遇到换行符，标准IO库执行IO操作。涉及终端（标准输入和标准输出）通常使用行缓冲
- 不带缓冲 标准错误流stderr通常不带缓冲，可以使错误信息尽快显示出来。

**惯例**

- 当且仅当标准输入和标准输出并不指向交互设备时，他们才是全缓冲的。
- 标准错误绝不会是全缓冲的

很多系统的默认情况

- 标准错误不带缓冲的
- 若是指向终端的设备的流，则是行缓冲；否则是全缓冲

### 更改缓冲类型setbuf /setvbuf

```c
//stdio.h
void setbuf(FILE *restrict fp, char *restrict buf);
void setvbuf(FILE *restrict fp, char *restrict buf, int mode , size_t size);
//成功0，出错非0

```

setbuf 可以打开或者关闭缓冲机制。buf参数必须指向长度为BUFSIZE的缓冲区，BUFSIZE在<stdio.h>中通常会定义。关闭缓冲将buf设置NULL

setvbuf精确说明所需要的缓冲类型。mode参数

- _IOFBF 全缓冲 
- _IOLBF 行缓冲
- _IONBF 不带缓冲  此时忽略buf和size参数

当该流是带缓冲，制定的buf是NULL，则标准库自动飞去适当的缓冲区。

![1567919157406](/home/spade/Documents/markdown/linux_programing_pics/5/1567919157406.png)



缓冲区的一部分用于存放自己的管理操作信息，所以存放在缓冲区中的实际字节数少于size。一般由操作系统选择缓冲区长度，自动分配缓冲区，这种情况下IO库会自动释放缓冲区。

## fflush  冲洗一个流

任何时候都可以强制冲洗一个流

```c
//stdio.h
int fflush(FILE *fp);
//成功0，出错EOF
```

所有未写的数据传送内核，如果fp是NULL，所有输出流都被冲洗。

## 打开流fopen 、freopen、fdopen

```c
//stido.h
FILE *fopen(const char *restrict pathname, const char *restrict type);
FILE *freopen(const char *restrict pathname, const char *restrict type, FIlE *restrict fp);
FILE *fdopen(int fd, const char *type); //只有POSIX.1有
//成功返回文件指针，失败返回NULL
```

区别：

- fopen打开pathname路径的文件
- freopen 在制定流上打开一个文件，如果该流已经打开，则需要关闭流。如果该流已经定向，则使用freopen清除该定向。多用于将指定文件打开文一个预定义的流，如标准输入，标准输入，标准错误。
- fdopen取一个已有的文件描述符（open dup等），并使一个标准IO流与之结合。常用于常见管道和网络通信通道函数返回的描述符，因为这些不能用标准IO函数fopen打开。

**type**

![1567919955280](/home/spade/Documents/markdown/linux_programing_pics/5/1567919955280.png)

`b`使得标准IO库可以区分文本文件和二进制文件，由于
UNIX内核不对这两种文件进行区分，所以制定b作为type一部分其实没有作用。

`+`的限制：

- 如果中间没有fflush/fseek/fsetpos/rewind，输出的后面不能跟随输入
- 如果中间没有fseek/fsetpos/rewind，或者一个输入操作没有达到文件尾端，则输入操作之后不能直接跟随输出。

除非流引用终端设备，否则系统默认打开时是`全缓冲的`。引用终端设备，是行缓冲。



## fclose关闭文件流

```c
//<stido.h>
int fclose(FILE *fp)
    //0 、 EOF
```

## 读写流

- 每次一个字符的IO	如果带缓冲，标准IO函数处理所有缓冲
- 每次一行的IO    使用fgets和fputs，以换行符终止。调用fgets时，要说明能处理的最大行长。
- 直接IO，fread和fwrite。

### getc/fgetc/getchar输入函数

```c
//stdio.h
int getc(FILE *fp);
int fgetc(FILE *fp);
int getchar(void);
//成功返回下一个字符，达到文件尾或者出错返回EOF
```

**getchar等同getc(stdin)**

getc和fget区别是fgetc是函数，getc是宏定义，：

- getc的参数不应当是具有副作用的表达式。
- fgetc是一个函数，有地址，可以作为参数传给另一个函数
- fgetc时间可能要长，函数的调用时间通常长于宏调用

### EOF情况判断ferror feof clearerr

```c
//stdio.h
int ferror(FILE *fp);
int feof(FILE *fp);
//条件为真返回非0，否则返回0
void clearerr(FILE *fp)
```



### 压回字符到流中

```c
//stdio.h
int ungetc(int c,FILE *fp);  //写到流缓冲区中。
//成功返回c，出错返回EOF
```

独处字符的顺序与压入相反。类似栈。ISO C支持任何次数的回送，但是每次只能会送一个字符。

进行某种切词操作常用到。需要看一下字节流中下一个字符。然后再会送回去。



### 输出函数 putc/fputc/putchar

```c
//stdio.h
int putc(int c, FILE *fp);
int fputc(int c, FILE *fp);
int putchar(int c);
//成功返回c，出错返回EOF
```

putchar等同于putc(c, stdout); putc可被实现为宏，fputc不能实现为宏

## 按行IO

### 输入fgets

```c
//<stdio.h>
char *fgets(char *restrict buf, int n, FILE *restrict fp);
char *gets(char *buf); //不建议
//成功返回buf，末尾或出错返回NULL
```

gets从标准输入读，fgets从制定流读

fgets必须制定**缓冲区长度**n , 此函数一直读取到下一个换行符为止，但是不超过n-1个字符。缓冲区以null结尾。如果该行包括最后换行符在内超过n-1，则fgets返回不完整的行，缓冲区仍以null结尾。不会溢出。下次还会继续读该行。

gets不推荐使用，因为使用gets不能指定缓冲区的长度，可能造成缓冲区溢出，写到缓冲区之后的存储空间。网络蠕虫就是利用这个缺陷。

### 输出fputs

```c
//stdio.h
int fputs(const char *restrict str, FILE *restrict fp);
int puts(const char *str);
成功放回非负，失败返回EOF
```

fputs 把以null结尾的字符串写到制定的流中。终止符null不写出。

puts会将一个换行符写到标准输出中

也不建议使用puts，一面记住它在最后是否添加了一个换行符。

## 标准IO库效率讨论

总体上使用标准IO库是比使用read和write要慢。但是对于复杂的应用程序，这些并不是影响程序效率的关键因素。具体讨论略。

## 二进制IO

如果进行二进制IO，我们更希望能够一次读写完一个完整的结构。getc和putc读写结构，需要使用循环。fgets和fputs遇到null字节停止，但是结构中可能包含null。

```c
//stdio.h
size_t fread(void *restrict ptr, size_t size, size_t nobj, FILE *restrict fp);
size_t fwrite(const void *restrict ptr, size_t size, size_t nobj, FILE *restrict fp);
//返回读或者写的对象数
```

- ptr 开始地址
- size 每个单位的大小
- 写的数量
- 文件指针

常见用法：

1. 读写一个二进制数组。

   ```c
   float data[10];
   if(fwrite(&data[2], sizeof(float), 4, fp)!=4)
       err_sys("fwrite error")
   ```

2. 读写一个结构。

   ```c
   struct{
       short count;
       long total;
       char name[NAMESIZE];
   }item; //item是一个实例
   if (fwrite(&item, sizeof(item)，1，fp) !=1)
       err_sys("fwrite error");
   ```

   如果出错或者到达文件尾端，返回的字数少于nobj。在这种情况下，使用ferror和feof来判断是出错了呢还是到头了呢。

## 定位流

三种方法定位IO流。

- `ftell`和`fseek`，假定文件的位置可以存放在`长整型`
- `ftello`和`fseeko，文件偏移量可以不必是长整形，使用off_t数据类型`
- `fgetpos`和`fsetpos`，ISO C引入，使用抽象数据类型fpos_t记录文件位置。可以根据定义为一个足够大的数来记录文件的位置

非UNIX系统应用程序应该使用fgetpos和fsetpos

```c
//stdio.h
long fell(FILE *fp);
//成功返回当前文件位置，出错返回-1
int fseek(FILE *fp, long offset, int whence);
//成功返回0，出错返回-1
void rewind(FILE *fp);

//除了偏移量的类型是off_t，其他和上面相同
off_t ftello(FILE *fp);
int fseeko(FILE *fp, off_t offset, int whence);
//0 -1

```

- offset 指定偏移量
- whence 解释偏移量计算方法，与lseek相同

  - SEEK_SET 偏移量从文件开始处计算，便宜offset字节。
  - SEEK_CUR 从当前位置算，offset可以正或者负
  - SEEK_END 从最后开始算，文件长度+offset ，offset可正可负

对于二进制文件，`文件位置`从`文件起始`开始计算，以`字节`为单位。

```c
//stdio.h
int fgetpos(FILE *restrict fp, fpos_t *restrict pos);
int fsetpos(FILE *fp, const fpos_t);
//成功0 出错非0
```

fgetpos 将文件位置指示器存入有pos指向的对象中。

## 格式化IO

### 输出 printf系列

#### 原型

```c
//stdio.h
int printf(const char * restrict format, ...);
int fprintf(FILE *restrict fp, const char *restrict format, ...); //可以制定文件
int dprintf(int fd, const char *restrict format, ...);
//成功返回输出字符数，出错返回负值
int sprintf(char *restrict buf, const char *restrict format, ...);
//成功返回存入数组的字符数，编码出错返回负
int snprintf(char *restrict buf, size_t n, const char *restrict format, ...);
//若缓冲区足够大，返回存入数组的字符数，编码出错返回负值
```

sprintf将格式化的字符送入数组buf中。sprintf在数组的尾段自动加一个null字节，但是该字节不包含在返回值内。**有可能造成缓冲区溢出，需要注意使用**

snprintf是为了解决缓冲区溢出的，缓冲区是一个显式的参数，草果缓冲区尾端的字符会被丢弃。

#### 格式控制

```
%[flags][fldwidth][precision][lenmodifier]convtype
```

- flags

  ![1568193117290](/home/spade/Documents/markdown/linux_programing_pics/5/1568193117290.png)

- fldwidth 是最小字段宽度
- precision 精度    `.4` 保留4为小数
- lenmodifier 说明参数长度

![1568193205860](/home/spade/Documents/markdown/linux_programing_pics/5/1568193205860.png)

- convtype 必写的，如何解释参数、格式

![1568193261256](/home/spade/Documents/markdown/linux_programing_pics/5/1568193261256.png)

### 格式化输入scanf族

#### 原型

```c
//stdio.h
int scanf(const char *restrict format, ...);
int fscanf(FILE *restrict fp, const char *restrict format, ...);
int sscanf(const char *restrict buf, const char *restrict format, ...);
//返回赋值的输入项数，输出出错或者任意转换前到达文件尾端，返回EOF

```

format说明了如何转换参数，对变量进行赋值。

#### format

```c
%[*][fldwidth][m][lenmodifier]convtype
```

- *用于抑制转换。
- fldwidth 最大宽度，最大字符数量
- lemodifier 转换结果赋值的参数大小
- convtype同上

## 临时文件 tmpnam、tmpfile

```c
//stdio.h
char *tmpnam(char *ptr);

FILE *tmpfile(void);
//成功返回文件指针，失败NULL
```

tmpnam创建一个唯一的路径名。得到这个之后手动创建文件。ptr初始化为长度至少L_tmpnam个字符的数组。（stdio.h有定义）

tmpfile 创建一个临时的二进制文件(wb+)，在关闭文件或者程序结束自动删除。

## 内存流

将内存区当做文件流的方式处理。通俗理解可以把一个demo[100]的内存区按照文件流的方式处理、

```c
//stdio.h
FILE *fmemopen(void *restrict buf, size_t size, const char *restrict type);
//成功返回指针，失败返回NULL
```

具体要求见书。P171

