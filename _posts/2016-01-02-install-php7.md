---
layout: post
category: "php"
title:  "安裝PHP7"
tags: [php,web]
summary: "php7的安裝記錄"
---
PHP7的性能號稱是5.x的3倍以上，是一個值得嘗試的大版本更新，但常用的擴展目前還沒有更新到pecl/pear中，故本文記錄一下安裝的步驟（以debian系列示例）。

## **Install dependencies**
``` shell
sudo apt-get install -y libmcrypt-dev libcurl4-openssl-dev
```

## **configure**
```shell
./configure \
    --prefix=/usr/local/php7 \
    --with-config-file-path=/usr/local/php7/etc \
    --with-iconv-dir=/usr/local \
    --with-mysqli \
    --with-iconv-dir=/usr/local \
    --with-freetype-dir \
    --with-jpeg-dir \
    --with-png-dir \
    --with-zlib \
    --with-libxml-dir=/usr \
    --enable-xml \
    --disable-rpath \
    --enable-bcmath \
    --enable-shmop \
    --enable-sysvsem \
    --enable-inline-optimization \
    --with-curl \
    --enable-mbregex \
    --enable-fpm \
    --enable-mbstring \
    --with-mcrypt \
    --with-gd \
    --enable-gd-native-ttf \
    --with-openssl \
    --with-mhash \
    --enable-pcntl \
    --enable-sockets \
    --with-xmlrpc \
    --enable-zip \
    --enable-soap
```

If any error happens, check error message to see missing what, then use 'sudo apt-get install -y [pkg]-dev' to install the missing one.

## **安装扩展**
```shell
sudo apt-get update
sudo apt-get install build-essential libmemcached-dev
```

- 安装memcached擴展

```
git clone https://github.com/php-memcached-dev/php-memcached 
cd php-memcached
git checkout -b php7 origin/php7
/usr/local/php7/bin/phpize
./configure --with-php-config=/usr/local/php7/bin/php-config --disable-memcached-sasl
make
sudo make install
```

- 安裝Redis擴展

```
git clone https://github.com/phpredis/phpredis.git
cd phpredis
git checkout -b php7 origin/php7
/usr/local/php7/bin/phpize
./configure --with-php-config=/usr/local/php7/bin/php-config
make
sudo make install
```





