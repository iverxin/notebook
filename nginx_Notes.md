[TOC]

# 1. Nginx 编译

```bash
#!/usr/bin/bash
. auto/configure --sbin-path=/usr/local/nginx/nginx --conf-path=/usr/local/nginx/nginx.conf --pid-path=/usr/local/nginx/nginx.pid  #--with-pcre=../pcre-8.43 --with-zlib=../zlib-1.2.11
```

然后make  --file Makefile

不用make isntall 安装。

# 2. Nginx 启动分析

经过编译后，`codeVS` 才能顺利找到相应的头文件。

**main**函数位于src/core/nginx.c文件中。



# 数据结构

## 地址池

## 3.2 创建地址池



pool 是一个链表，current指向了下次申请地址时去哪个结点开始。

current指向的结点最大failed值是5，包括该结点在内后边最多有6个。failed值是5.4.3.2.1.0的结点。

failed值不能可能





## cycle

1. 开始先创建一个init_cycle，在log初始化之后，开始将log的地址挂载到cycle上。并且让`ngx_cycle` 指向`init_cycle`。
2. 创建一个大小为1024的pool挂载到`init_cycle`上。
3. 对init_cycle的参数进行一些处理。
4. 创建新的cycle，并把old_cycle指向init_cycle
5. 处理或创建新的cycle挂载

