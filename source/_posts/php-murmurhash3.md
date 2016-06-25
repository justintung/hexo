---
title: php-murmurhash3
date: 2016-05-28 02:40:42
tags:
- PHP扩展
- hash计算
categories:
- PHP
---
1.简介
MurmurHash算法：高运算性能，低碰撞率的hash算法
murmurhash是 Austin Appleby于2008年创立的一种非加密hash算法，适用于基于hash进行查找的场景。murmurhash最新版本是MurMurHash3，支持32位、64位及128位值的产生。
murmurhash标准使用c++实现，但是也有其他主流语言的支持版本，包括：perl、c#、ruby、python、java等。murmurhash在多个开源项目中得到应用，包括libstdc、libmemcached、nginx、hadoop等。

2.安装
```shell
git clone https://github.com/j42/php-murmurhash3.git
cd php-murmurhash3/
/www/itmotu/php-5.5.33/bin/phpize
./configure --with-php-config=/www/itmotu/php-5.5.33/bin/php-config
make && make install
```
`vi /www/itmotu/php/lib/php.ini`添加`extension=php-murmurhash3`
```shell
service php-fpm restart
php -m
```

3.扩展中的函数
```c
string murmurhash3 ( string $key , long $seed )
```
demo:
```php
<?php
echo murmurhash3("justin",1);
echo "<br/>";
echo murmurhash3("276887367@qq.com",1);
?>
```