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

