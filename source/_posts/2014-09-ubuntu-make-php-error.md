---
title: ubuntu源码编译安装php常见错误解决办法
date: 2014-09-01 23:26:29
tags:
  - ubuntu
  - linux
  - php
category: PHP
---

``` bash

./configure –prefix=/usr/local/php –with-config-file-path=/etc \
–with-mysql=/usr/local/mysql –with-mysqli=/usr/local/mysql/bin/mysql_config \
–with-iconv-dir=/usr/local –with-freetype-dir –with-jpeg-dir
–with-png-dir –with-zlib –with-libxml-dir=/usr –enable-xml \
–disable-rpath –enable-safe-mode –enable-bcmath –enable-shmop \
–enable-sysvsem –enable-inline-optimization –with-curl \
–with-curlwrappers –enable-mbregex –enable-fpm –enable-mbstring \
–with-mcrypt –with-gd –enable-gd-native-ttf –with-openssl \
–with-mhash –enable-pcntl –enable-sockets –with-xmlrpc \
–enable-zip  –enable-soap

```

出现得错误如下：
错误一：
configure: error: xml2-config not found. Please check your libxml2 installation.
而我已经安装过了libxml2,但是还是有这个提示：
解决办法：
```bash
sudo apt-get install libxml2-dev
```

错误二：
configure: error: Please reinstall the BZip2 distribution
而我也已经安装了bzip2，网上找到得解决方案都是需要安装bzip2-dev，可是11.10里面没有这个库。
解决办法：在网上找到bzip2-1.0.5.tar.gz，解压，直接make ,sudo make install.(我使用的该源来自于http://ishare.iask.sina.com.cn/f/9769001.html)

错误三：
configure: error: Please reinstall the libcurl distribution -easy.h should be in /include/curl/
解决办法：
```bash
sudo apt-get install libcurl4-gnutls-dev
```

错误四：
configure: error: jpeglib.h not found.
解决办法：
```bash
sudo apt-get install libjpeg-dev
```

错误五：
configure: error: png.h not found.
解决办法：
```bash
sudo apt-get install libpng-dev
```

错误六：
configure: error: libXpm.(a|so) not found.
解决办法：
```bash
sudo apt-get install libxpm-dev
```

错误七：
configure: error: freetype.h not found.
解决办法：
```bash
sudo apt-get install libfreetype6-dev
```

错误八：
configure: error: Your t1lib distribution is not installed correctly. Please reinstall it.
解决办法：
```bash
sudo apt-get install libt1-dev
```

错误九：
configure: error: mcrypt.h not found. Please reinstall libmcrypt.
解决办法：
```bash
sudo apt-get install libmcrypt-dev
```

错误十：
configure: error: Cannot find MySQL header files under yes.
Note that the MySQL client library is not bundled anymore!
解决办法：
```sh
sudo apt-get install libmysql++-dev
```

错误十一：
configure: error: xslt-config not found. Please reinstall the libxslt >= 1.1.0 distribution
解决办法：
```bash
sudo apt-get install libxslt1-dev
```

可见PHP源码安装之前需要先安装这些依赖，详细可见http://forum.ubuntu.org.cn/viewtopic.php?f=88&t=231159
如上错误都解决之后，再次./config….没有错误之后，
```
make
sudo make install
```
