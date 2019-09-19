[TOC]

# 文件和目录

## stat、fstat、fstatat和lstat 文件信息获取

### 原型

```c
#include <sys/stat.h>
int stat(const char *restrict pathname, struct stat *restrict buf);
int fstat(int fd, struct stat *buf);
int lstat(const char *restrict pathname, struct stat *restrict buf);
int fstatat(int fd, const char *restrict pathname, struct stat * restrict buf, int flag);
//成功返回0 失败返回-1
```

**stat** 给出pathname， stat函数就会返回和该文件相关的`信息结构`

**fstat** 获得描述符fd上打开的文件的相关信息

**lstat** 功能和stat相似，但是当前文件是一个符号链接的时候，lstat返回的是符号链接的信息，而不是指向的文件信息。

**fstatat** 相对于当前打开目录( fd参数指向这个目录) 的路径返回文件的统计信息。 

- flag参数控制着是否跟随一个符号链接。

  - AT_SYLINK_NOFOLLOW **fstatat**不会跟随符号链接，返回符号链接的本身信息。否则默认情况是返回符号链接所指向的文件信息。
  - AT_FDCWD，并且pathname是一个相对路径，fstatat会计算当前目录的pathname路径，付过pathname是绝对路径，该标志就会忽略。这两种情况，fstatat的作用和stat或lstat一样。

- buf 参数是一个指针，指向一个必须提供的结构，函数来填充buf指向的结构。就是接收函数的返回结果。

  stat具体内容：

  ```c
  struct stat{
  ```

  ![1567146996825](linux_programing_pics/1566893155487.png)

  根据实际定义可能结构稍有不同。

## 文件类型

- 普通文件(regular file)： 常见的文件，这种数据是文本还是二进制和内核无关，对于普通文件的解释是由处理该文件的应用程序进行。

- 目录文件(directory file)：这种文件包括了其他`文件名`和指向这些文件的`有关信息的指针` 。对于文件目录，有`读权限`的任何进程都可以读，但是只有`内核` 可以直接写目录文件。进程必须使用本章介绍的函数才能够更改目录。

- 块特殊文件(block special file ): 对设备（磁盘等）带缓冲访问，每次访问用固定长度为单位进行。

  > FreeBSD不再支持块特殊文件，对设备的访问需要通过字符特殊文件进行。

- 字符特殊文件(character special file ) 提供对设备不带缓冲的访问，每次访问的长度可以改变。系统中的所有设备要么是字符特殊文件，要么是块特殊文件。

- FIFO 用于进程间通信，也成为`命名管道( named pipe )`

- 套接字 (socket) 进程间的网络通讯，套接字也可以在一台宿主机上进程间的非网络通讯。

- 符号链接(symbolic link) 指向另一个文件，类似win的快捷方式。

文件类型信息在stat结构中的st_mode 中。

![1567147837382](linux_programing_pics/1567147837382.png)

图片来自unix环境高级编程。

```c
//使用方法
//判断一个文件是否是常规文件
char *pathname="/etc/passwd";
struct stat buf;
if (lstat(pathname, &buf)<0){
    printf("lstat error");
}
if (S_ISREG(buf.st_mode)) //是该类型返回1
    printf("regular file");
```



## 用户ID和用户组

进程相关联的ID：

- 实际用户ID

- 实际组ID

  实际是谁，取自登录时口令文件中的登录项。一般不变，但是root用户可以改变。

- 有效用户ID

- 有效用户组ID

- 附属组ID

  用于文件访问权限检查，决定了我们的文件访问权限。

- 保存的设置用户ID

- 保存的设置组ID

  有exec函数保存，在执行下一个程序时包含了有效用户ID和有效用户组的副本。作用见setuid

在执行程序文件时，通常有效用户ID等于实际用户，有效用户组等于实际用户组。

但是可以在`文件模式字(st_mode)`中设置一个特殊标志，含义是执行这个文件时，将`进程的有效ID`设置为`所有者的用户ID(st_uid)`。在`文件模式字(st_mode)`中也可以设置另一位，将执行此文件进程的`有效用户组ID` 设置为`文件所有者的ID`

在文件模式字中这两位分别成为`设置用户ID(set-user-ID)位`和` 设置组(set-group-ID)位`。

比如，文件的所有者是超级用户，并且设置了该文件的 `设置用户ID位` ,那么执行这个文件的进程就拥有了超级用户的权限。



## 文件权限

所有的文件类型都有访问权限(access permission)。每个文件有9个访问权限。

| st_mode  |   含义   |
| :------: | :------: |
| S_IRUSER |  用户读  |
| S_IWUSR  |  用户写  |
| S_IXUSR  | 用户执行 |
| S_IRGRP  |   组读   |
| S_IWGRP  |   组写   |
| S_IXGRP  |  组执行  |
| S_IROTH  |  其他读  |
| S_IWOTH  |  其他写  |
| S_IXOTH  | 其他执行 |

chmod命令就是修改这些参数的。

- u 代表usr
- g 代表组
- o 代表其他

**关于文件权限的规则**

- 打开一个文件时，必须要对所`经过的目录`以及`该文件`都具有执行权限。对一个没有执行权限的目录，将看不到里面的任何内容用。
- 读权限决定是否可以打开该文件进行读取。
- 写权限决定是否可以写入一个文件
- 在open函数中指定一个文件O_TRUNC标志，要具有写权限
- 在目录中创建文件，必须具有`写权限`和`执行权限`
- 删除一个文件，必须对目录有`写`和`执行`权限，对文件有`读写权限`
- 如果用exec系列中的任何函数执行文件，必须具有`执行权限`，该文件还应该是`普通文件`



进程处理文件时，内核会对文件访问权限进行测试，也就是对比进程里面的ID和文件的属性ID，具体方法见《unix高级环境变成 第三版》80页。



## 新文件和目录的权限

新文件的用户ID是进程的有效用户ID，用户组ID(1)可以使进程有效组ID，(2)也可能是所在目录的组ID

>FreeBSD和Mac Os 是用（2），Linux 可以使用mount命令修改，默认情况下是先尝试(2)，如果目录组ID没设置，就用(1)

## access和faccessat 按照实际用户ID和实际ID组进行访问权限测试

```c
#include <unsitd.h>
int access(const char *pathname , int mode );
int facessat(int fd, const char *pathname , int mode, int flag);
//成功返回0，失败返回-1
```

mode选项：

- R_OK 测试读取权限

- W_OK 测试写权限

- X_OK 测试执行权限

- F_OK 测试是否存在

  使用时按位或

flag参数如果设置为AT_EACCESS, 访问检查用的是进程的有效用户ID
和有效用户组ID, 而不是实际用户ID和实际用户组ID

## umask为进程设置文件模式创建屏蔽字

```c
#include <sys/stat.h>
mode_t umask(mode_t cmask);
```

cmask是文件9个权限中的若干个按位`或`构成的。

**在文件模式创建屏蔽字中为1的为，在文件mode中相应的位就一定会被关闭（屏蔽）**

使用方法

```c
#define RWRWRW (S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP | S_IROTH | S_IWOTH)

umask(0);//不屏蔽
if (create("foo",RWRWRW)<0)//创建的文件权限是-rw-rw-rw-
    printf("create failed"); 
umask(S_IRGRP | S_IWGRP | S_IROTH | S_IWOTH);
if (create("bar",RWRWRW)<0) //创建的文件权限-rw-------
    printf("create failed"); 
eixt(0);
    
```

![1567238234062](/home/spade/Documents/markdown/linux_programing_pics/1566893155487.png)

shell初始化时会这只一次umask，然后不在改变，一般为0022，屏蔽了其他写和其他执行。为了确保权限正确，需要考虑umask的值，避免需要的权限被屏蔽掉的情况。

### umask命令

在终端可使用umask查看当前环境的屏蔽字

```bash
spade@spade-PC:~$ umask -S #查看权限
u=rwx,g=rx,o=rx
spade@spade-PC:~$ umask #查看屏蔽的权限
0022 #2是其他写，20是组写
spade@spade-PC:~$ umask 0 #设为不屏蔽
spade@spade-PC:~$ umask
0000
x
```

## chmod/fchmod/fchmodat 修改文件权限

### 原型

```c
#include <sys/stat.h>
int chmod(const char *pathname, mode_t mode);
int fchmod(int fd, mode_t mode);
int fchmodat(int fd, const char *pathname, mode_t mode, int flag);
//成功返回0，失败返回-1
```

fchmodat 与 chmod在以下两种情况是相同的

- pathname是绝对路径

- fd参数取值为AT_FDCWD而参数pathname是相对路径

  flag设置AT_SYMLINK_NOFOLLOW，fchmodat不会跟随符号链接

![1567239649516](/home/spade/Documents/markdown/linux_programing_pics/1567239649516.png)

mode取值按位或

chmod在下列条件会清除两个权限位：

- 对于Solaris等系统，没有超级用户权限设置`粘着位`，那么mode中的`粘着位`就会自动关闭。防止恶意用户设置粘着位

  > Linux3.2和mac oS X 对这个没限制，因为对普通文件没意义

创建新文件组ID可能不是调用进程所属的组（详见新文件和目录的所有权）。新文件的组ID可能是父目录的组ID，如果这种情况，又没有超级用户权限，那么设置组ID位会被自动屏蔽，防止用户创建一个设置组ID，而该文件是非该用户组所属的组拥有的。



## chown、fchown、fchownat、lchown 修改文件用户ID和组ID

```c
#include <unistd.h>
int chown(const char *pathname, uid_t owner, gid_t group);
int fchown(int fd, uid_t owner, gid_t group);
int fchownat(int fd, const char * pathname, uid_t owner, gid_t group, int flag); //fd 取值AT_FDCWD时，pathname可以使用相对路径
int lchown(const char *pathname, uid_t owner, gid_t group);//只用来修改link文件的属性
//成功返回0 失败-1
```

## 文件长度

stat结构体中st_size。

**单位**:字节

由于单位的关系，只针对普通文件、目录文件和连接符号有意义。

注意：

- 普通文件的床都可以是0
- 目录长度通常是一个数(16或者512的整数倍)
- 符号链接长度是文件名中实际字节数



st_blksize和st_blocks

- st_blksize是对文件I/O合适的块的长度
- st_blocks 是分配的512字节块数量



## 文件截断truncate/ftruncate

### 功能：

在文件尾端去掉一些数据缩短文件，将现有文件截断为length，如果文件长度小于length，文件长度增加，创建了一个空洞。

### 原型

```c
#include <unistd.h>
int truncate(const char *pathname , off_t length);
int ftruncate(int fd, off_t length);
//成功返回0，失败返回-1
```



## 文件系统

把磁盘分成若干个区，每个区都可以是不同的文件系统。

i结点是固定长度的`记录项`，包含文件的大部分信息。

![1567399266621](linux_programing_pics/1567399266621.png)



- 每个i结点有一个链接计数，指向该节点的目录项数。当链接计数为0时可以删除释放文件块。这也是删除目录项函数是unlink不是delete，链接计数在st_nlink成员中，这种链接是硬链接。可以将重要文件设置多个硬链接，防止误删。
- 另一种是软连接，符号链接，包含了该符号链接所指向的文件名字。并不直接指向i结点，而是数据区存放另一个文件的路径。
- i结点包含了文件相关的所有信息。stat中大多数信息来自i结点。
- 一个目录项不能指向另一个文件系统的i结点，这就是为什么ln的硬链接不能跨越文件系统。（软连接可以)

![1567399295325](linux_programing_pics/1567399295325.png)

## link/linkat 链接文件

```c
#include<unistd.h>
int link(const char * existingpath, const char *newpath);
int linkat(int efd, const char *existingpath, int nfd, const char *newpath, int flag);
//成功0，错误-1
```

这两个函数功能都是创建一个新的目录项newpath，引用现有文件existingpath，如果newpath已经存在，就返回时错误。**只创建newpath的最后一项，其他的部分应该已经存在。**

linkat 函数中`efd`是existing fd，nfd是new fd。其他的使用方式和之前的带有at结尾的函数类似。详见openat

## ulink、unlinkat ,remove删除链接

```c
//unistd.h
int unlink(const char *pathname);
int unlinkat(int fd, const char *pathname, int flag);
//stdio.h
int remove(const char *pathname)
//成功0，错误-1
```

解除对文件的链接，需要有该文件的写和执行权限。

- flag参数使用AT_REMOVEDIR时，unlinkat可以类似rmdir一样删除目录。

unlink经常用来保证即使程序在崩溃时，临时文件也随之销毁。用open或者creat创建一个文件，然后立刻使用unlink。文件处于打开状态，内容不会被立刻删除（目录项会被删除），等进程结束后，该文件内容才被删除。

remove对于文件和unlink相同，对于目录，和rmdir相同。

## rename/renameat重命名

```c
//stdio.h
int rename(const char *oldname, const char *newname);
int renameat(int oldfd, const char *oldname, int newfd, const char *newname);
//0 /-1
```

可能出现的一些特殊情况：

- oldname是文件或者符号链接，newname已经存在
  - newname存在，且不在一个目录，先将该目录项删除然后将oldname重命名为newname。对于涉及的目录需要有写权限。因为对这个目录进行修改
  - newname存在，它不能是引用一个目录，（不能是一个目录。）
- oldname 是一个目录
  - newname存在，必须引用一个目录（是一个目录，且是空目录）,先将其删除，然后将oldname重命名为newname
- oldname或newname引用符号链接，处理的是链接的本身，不是指向的文件。
- 不能对. 和.. 重命名
- oldname和newname是同一个文件，函数不做更改直接返回。

## 符号链接(软连接)symlink/symlinkat

### 区别

符号链接为了解决硬链接的一些限制

- 硬链接直接指向文件`i结点`

- 只有超级用户才能创建指向目录的硬链接（防止环）

符号链接以及指向的对象没有文件系统的限制。

使用函数时要注意这些函数是否处理符号链接。

### 原型

```c
//unistd.h
int symlink(const char*actualpath, const char *sympath);
int symlinkat(const char *actualpath, int fd , const char *sympath);
//0/1
```

要求actualpath已经存在。

由于open会打开链接指向的文件，所以设计了打开链接本身的方法，并读取链接中的名字。

```c
//unistd.h
ssize_t readlink(const char *restrict pathname, char *restrict buf, size_t bufsize);
ssize_t readlinkat(int fd, const char* restrict pathname, char *restrict buf, size_t bufsize);

//成功返回读取字节数，出错返回-1
```



## 文件的时间

每个文件维护3哥时间字段

| 字段    | 说明                    | 例子        | ls选项 |
| ------- | ----------------------- | ----------- | ------ |
| st_atim | 最后访问时间            | read        | -u     |
| st_mtim | 最后修改时间            | write       | 默认   |
| st_ctim | i结点状态的最后修改时间 | chmod/chown | -c     |

![1567507469834](/home/spade/Documents/markdown/linux_programing_pics/1567507469834.png)



### 修改的函数futimens/utimensat/utimes



```c
//sys/stat.h
int futimens(int fd, const struct timespec times[2]);
int utimensat(int fd, const char *path, const struct timespec times[2], int flag);
//0、/1
```

修改文件的访问时间。futimens和utimensat可以制定纳秒级别的时间戳。

- times 如果是空指针，访问时间和修改时间都设置为当前时间
- times指向两个timespec结构数组，
  - 任意一个数组元素的tv_nsec字段的值是UTIME_NOW，相应的时间戳就设置为当前时间，忽略tv_sec字段。
  - 任意一个数组元素的tv_nsec字段值是UTIME_OMIT，相应的时间戳保持不限，忽略tv_sec字段-
  - tv_nsec字段不是上边两个，相应时间戳设置为tv_sec和tv_nsec

![1567509266741](/home/spade/Documents/markdown/linux_programing_pics/1567509266741.png)

## mkdir、mkdirat、rmdir

```c
//sys/stat.h
int mkdir(const char *pathname, mode_t mode);
int mkdirat(int fd, const char *pathname , mode_t mode);
//0.-1

int rmdir(const char *pathname) //可以删除空目录
```

## mkdir、mkdirat、rmdir

```c
// sys/stat.h
int mkdir(const char *pathname, mode_t mode);
int mkdirat(int fd, const char *pathname, mode_t mode);
//0,-1
```

```c
//unistd.h
int rmdir(const char *pathname);
//0,-1
```

- 如果目录的链接计数为0，并且没有其他进程打开此目录，就释放此目录占用的空间。

- 如果链接计数是0，有别的进程打开此目录，在次函数返回前删除最后一个链接及. 和 .. 项。这个目录不能创建新文件，最后一个进程关闭之后再释放目录占用的空间。（这些进城不能在目录下执行其他操作，就相当于没啥用了。）

## 读目录

任何具有访问权限的用户都可以读目录。但是为了防止文件系统混乱，只有**内核才能写目录**

目录的`写`和`执行`权限决定能否创建新的文件和目录。

```c
// dirent.h
DIR *opendir(const char *pathname);
DIR *fdopendir(int fd); //把文件描述符处理为DIR结构

// 成功返回指针，失败返回NULL
struct dirent *readdir(DIR *dp);
//成功返回指针，失败返回NULL
void rewinddir(DIR *dp);
int closedir(DIR *dp);
//0，-1
long telldir(DIR *dp);
//与dp关联的目录中的当前位置
void seekdir(DIR *dp, long loc);

```

DIR结构是一个内部结构，7个函数用这个内部结构保存当前正在被读取的目录的相关信息。

`opendir`和`fdopendir`返回的DIR给下面的5个函数调用

**目录中的各项目录顺序与实现有关，通常不按照字母顺序**



## chdir、fchdir、getcwd

```c
//unistd.h
//制定新的工作目录
int chdir(const char *pathname);
int fchdir(int fd);
//0,-1

char *getcwd(char *buf, size_t size);

```



## 设备特殊文件

- 文件系统所在的存储设备由主、次设备号表示，谁被好数据类型是dev_t。次设备号标识子设备。如在同一个磁盘驱动器上，主设备号相同，次设备号不同。

- major和minor是两个宏用来访问主次设备号。
- 与文件名相关联的st_dev值是文件系统设备号。major和minor访问。
- st_rdev包含设备号。只有字符特殊文件和块特殊文件才有此值。用major和minor访问

st_dev和st_rdev 都在stat(pathname,&buf)的buf中。(struct stat buf;)



## 文件权限

![1567830376735](C:\Users\Spade\Desktop\notebook\linux_programing_pics\1567830376735.png)

![1567830398267](C:\Users\Spade\Desktop\notebook\linux_programing_pics\1567830398267.png)

