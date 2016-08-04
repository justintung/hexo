---
title:  PhantomJS
date: 2016-08-03 20:46:52
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
站点页面index.html源码
```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
    <script type="text/javascript" src="jquery-1.9.0.js"></script>

</head>
<body>
    <div id="content"></div>
<script type="text/javascript">
    $(document).ready(function(){
		$("#content").html("PhantomJS JS赋值的内容");
	})

</script>
</body>
</html>
```
负责抓取index.html的PhantomJS源码
```js
var webPage = require('webpage');
var page = webPage.create();

page.open('http://test.itmotu.com/index.html', function (status) {
  var content = page.content;
  console.log(content);
  phantom.exit();
});
```
保存为test.js,然后执行
```shell
/usr/local/phantomjs-2.1.1-linux-x86_64/bin/phantomjs test.js > result.html
```
result.html内容
```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd"><html><head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
    <script type="text/javascript" src="jquery-1.9.0.js"></script>

</head>
<body style="zoom: 1;">
    <div id="content">PhantomJS JS赋值的内容</div>
<script type="text/javascript">
    $(document).ready(function(){
		$("#content").html("PhantomJS JS赋值的内容");
	})

</script>

</body></html>
```

对比index.html和result.html得知，result.html为index.html页面执行js后的结果
