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

https://xu3352.github.io/python/2018/05/15/python-3-install

先安装依赖 

```
sudo yum install zlib-devel bzip2 bzip2-devel readline-devel sqlite sqlite-devel openssl-devel xz xz-devel libffi-devel
```



下载安装包

```shell
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

```
make: *** [Modules/_ssl.o] 错误 1
```

```shell
手动安装 openssl-1.0.2e:
$ cd /tmp
$ wget http://www.openssl.org/source/openssl-1.0.2e.tar.gz
$ tar xzvf openssl-1.0.2e.tar.gz
$ cd openssl-1.0.2e
$ ./config --prefix=/usr/local/openssl --openssldir=/usr/local/openssl
$ make && make install
```

注意:不适用参数编译时, 默认安装目录为: `/usr/local/ssl`, 这里我们安装到了 `/usr/local/openssl`, 后面也需要对应的修改
修改```./setup.py```: (默认的openssl路径不改也可以)

  ```
        # Detect SSL support for the socket module (via _ssl)
        search_for_ssl_incs_in = [
                              '/usr/local/openssl/include', # 修改为新目录
                              '/usr/local/openssl/include/openssl',  # 新增
                              '/usr/contrib/ssl/include/'
                             ]
  ```
修改 `./Modules/Setup.dist`:

- ```
  # Socket module helper for socket(2)
  _socket socketmodule.c
  
  # Socket module helper for SSL support; you must comment out the other
  # socket line above, and possibly edit the SSL variable:
  SSL=/usr/local/openssl   # 这里改为我们指定的目录
  _ssl _ssl.c \
      -DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
      -L$(SSL)/lib -lssl -lcrypto
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