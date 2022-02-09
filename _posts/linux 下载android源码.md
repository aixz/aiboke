---
layout: post
title: android
date: 2021-11-08
Author: aixz
categories:
tags: [笔记, Android]
comments: true 
---

### CentOS 7 python 为2.7  安装python3.5

先安装依赖 

```
sudo yum install zlib-devel bzip2 bzip2-devel readline-devel sqlite sqlite-devel openssl-devel xz xz-devel libffi-devel
```



下载安装包

```text
wget https://www.python.org/ftp/python/3.5.2/Python-3.5.2.tgz
```



解压后修改Module文件夹下Setup 以及Setup.list 文件,即将注释取消：

```
# Socket module helper for socket(2)
_socket socketmodule.c

# Socket module helper for SSL support; you must comment out the other
# socket line above, and possibly edit the SSL variable:
SSL=/usr/local/ssl
_ssl _ssl.c \
	-DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
	-L$(SSL)/lib -lssl -lcrypto
```

后执行:

```text
# ./configure --prefix=/usr/local/python3 --enable-optimizations
```

若执行报错：
```
configure: error: no acceptable C compiler found in $PATH
```

```
//执行
# yum install make gcc gcc-c++ 
```

再次执行

```text
# ./configure --prefix=/usr/local/python3 --enable-optimizations
```

```text
# make 
```

```text
# make install 
```

```text
# mv /usr/bin/python /usr/bin/python.bak
```

```text
# ln -s /usr/local/bin/python3 /usr/bin/python
```