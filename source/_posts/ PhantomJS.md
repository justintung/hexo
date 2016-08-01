---
title:  PhantomJS
date: 2016-06-08 20:46:52
tags:
-  PhantomJS
-  Webkit
categories:
- 安装
---
#### 1.安装
下载地址: [http://phantomjs.org/download.html](http://phantomjs.org/download.html)

```shell
yum -y install gcc gcc-c++ make flex bison gperf ruby openssl-devel freetype-devel fontconfig-devel libicu-devel sqlite-devel libpng-devel libjpeg-devel

wget https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-2.1.1-linux-x86_64.tar.bz2

tar xvf phantomjs-2.1.1-linux-x86_64.tar.bz2 -C /usr/local/

```
#### 2.demo
```js
var webPage = require('webpage');
var page = webPage.create();

page.open('http://www.baidu.com', function (status) {
  var content = page.content;
  console.log('Content: ' + content);
  phantom.exit();
});
```
保存为test.js,然后执行
```shell
/usr/local/phantomjs-2.1.1-linux-x86_64/bin/phantomjs test.js
```