# 进程关系

[TOC]

## 终端登录

### 终端登录原理

启动过程：

1. 内核创建init进程(ID=1)
2. init读取/etc/ttys，对于每个允许登录的终端设备，调用一次fork
3. 子进程exec getty程序

![1569232896868](pics/9_Thread_relationship/1569232896868.png)

现代操作系统支持多个身份验证登录，基本都支持PAM(Pluggable Authentication Modules，可出入的身份验证模块)的方案。允许管理人员配置使用何种身份验证方法来访问那些基于PAM库编写的服务。

服务程序验证用户是否具有适当的权限执行某个服务，两种方案：

- 将身份验证机制编写到应用程序中
- 使用PAM库

如果用户正确登录，login完成的工作：

- 设置工作目录为用户起始目录
- 调用chown更改总段的所有权，使登录用户为所有者
- 对终端设备的访问权限改为“用户读和写”
- 调用setgid和initgroups设置进程的组ID
- 初始化环境
- 更改登录用户ID并调用该用户的shell。
- 等等略，如打印日期消息等

现在登录shell读取启动文件是.profile(Bourne shell和Korn shell)。GUN Bourne-again shell是.bash_profile .bash_login或.profile。