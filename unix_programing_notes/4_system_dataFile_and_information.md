[TOC]

# 系统数据文件和信息

## 口令文件

![1568281492956](/home/spade/Documents/markdown/linux_programing_pics/6/1568281492956.png)

例如，/etc/passwd文件

```
root:x:0:0:root:/root:/bin/bash
```

- root 通常ID都是0
- 加密口令占用一位。早期版本该字段存放口令，现在口令转移到别的文件。
- 如果口令字段为空，该用户无口令。
- shell字段是可执行程序，指定用户登录的shell。如果为空使用系统默认

- nobody是任何人都能登陆，但是不提供任何特权，这能访问人人都可读写的文件。

## 获取口令文件 getwpuid getpwnam

```c
//pwd.h
struct passwd * getpwuid(uid_t uid);
struct passwd * getpwnam(const char *name);
//成功返回指针，失败放回NULL


struct passwd *getpwent(void); //搜索整个口令文件
//成功返回指针，出错或到文件尾返回NULL
void setpwent(void); //自我保护措施，确保在调用在文件头
void endpwent(void); //getpwnam和getpwuid完成后要调用endpwent关闭文件

//搜索整个文件demo
struct passwd *p;
setpwent(); //确保定位开始处
while((p=getpwent())!=NULL){
    if(strcmp(name, ptr->pw_name) == 0)  //name自己替换
        break; //找到对应的
}
endpwent(); // 关闭文件
```

eg:

```c
#include <pwd.h>
int main(int argc, char *argv[]) {
	struct passwd * passwd;
	passwd=getpwnam(argv[1]);
	printf("name: %s \n", passwd->pw_name);
	printf("passwd: %s",passwd->pw_passwd);
}
```

## 阴影口令

加密口令是单向加密算法处理的用户口令副本。

![1568461255684](/home/spade/Documents/markdown/linux_programing_pics/6/1568461255684.png)

阴影口令文件不是一般用户可以读取的。但是passwd可以用户自由读取

```c
//shadow.h
struct spwd *getspnam(const char *name);
struct spwd *getspent(void);
//失败返回NULL

void setspent(void); 
void endspent(void); //关闭文件
```

## 组文件

头文件： grp.h

struct group :  

![1568462239762](/home/spade/Documents/markdown/linux_programing_pics/6/1568462239762.png)

```c
//grp.h
struct group *getgrgid(gid_t gid);
struct group *getgrnam(const char *name);
//成功返回指针，失败返回NULL
```

如果搜索整个文件组：

```c
//grp.h
struct group *getgrent(void);
//成功返回指针，出错或到文件尾端返回NULL
void setgrent(void);
void endgrent(void);
```

