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



# 3. 地址池

## 3.2 创建地址池



pool 是一个链表，current指向了下次申请地址时去哪个结点开始。

current指向的结点最大failed值是5，包括该结点在内后边最多有6个。failed值是5.4.3.2.1.0的结点。

failed值不能可能

