---
title:  PhantomJS
date: 2016-08-03 20:46:52
tags:
-  PhantomJS
-  快递查询
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
#### 2.phantomjs 代理服务器脚本 pagegrab.js
```html
"use strict";
var port, server, service,
    system = require('system');
function urlArg(url, arg) {
    if (url.indexOf("?") > 0) {
        var arrParams = url.split("?");

        var arrURLParams = arrParams[1].split("&");

        var arrParamNames = new Array(arrURLParams.length);
        var arrParamValues = new Array(arrURLParams.length);

        var i = 0;
        for (i = 0; i < arrURLParams.length; i++) {
            var sParam = arrURLParams[i].split("=");
            if (arg == sParam[0]) {
                return sParam[1];
            }
        }
        return '';
    } else {
        return '';
    }
}
function isURL( str) {
    return /^(?:(?:(?:https?):)?\/\/)(?:\S+(?::\S*)?@)?(?:(?!(?:10|127)(?:\.\d{1,3}){3})(?!(?:169\.254|192\.168)(?:\.\d{1,3}){2})(?!172\.(?:1[6-9]|2\d|3[0-1])(?:\.\d{1,3}){2})(?:[1-9]\d?|1\d\d|2[01]\d|22[0-3])(?:\.(?:1?\d{1,2}|2[0-4]\d|25[0-5])){2}(?:\.(?:[1-9]\d?|1\d\d|2[0-4]\d|25[0-4]))|(?:(?:[a-z\u00a1-\uffff0-9]-*)*[a-z\u00a1-\uffff0-9]+)(?:\.(?:[a-z\u00a1-\uffff0-9]-*)*[a-z\u00a1-\uffff0-9]+)*(?:\.(?:[a-z\u00a1-\uffff]{2,})).?)(?::\d{2,5})?(?:[/?#]\S*)?$/i.test( str );
}
function urldecode (str) {
  return decodeURIComponent((str + '')
    .replace(/%(?![\da-f]{2})/gi, function () {
      return '%25'
    })
    .replace(/\+/g, '%20'))
}
if (system.args.length !== 2) {
    console.log('使用方法:phantomjs pagegrab.js <端口号>');
    phantom.exit(1);
} else {
    port = parseInt(system.args[1]);
    server = require('webserver').create();

    service = server.listen(port, function (request, response) {
        response.statusCode = 200;
        response.headers = {
            'Cache': 'no-cache',
            'Content-Type': 'text/html'
        };
        var webPage = require('webpage');
        var url = urlArg(request.url, 'url');
        if (url == "") {
            response.write("未传递参数url（要抓取的页面地址）");
            response.close();
            return
        }

        //@todo url解析bug，2次才行
        url = urldecode(url);
        url = urldecode(url);
        if(!isURL(url)){
            response.write("不是有效的url地址,应以http或https开头");
            response.close();
            return
        }

        var page = webPage.create();
        page.settings.Referer = url;
        page.settings.userAgent = 'Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36';
        page.open(url, function (status) {
            response.write(page.content);
            response.close();
        });
    });

    if (service) {
        console.log('程序运行在端口 ' + port);
    } else {
        console.log('错误: 无法监听端口 ' + port);
        phantom.exit();
    }
}

```
运行代理
```js
/usr/local/bin/phantomjs /home/phantomjs/pagegrab.js 8522
```
添加supervisor服务
```shell
[program:pagegrab]
command=/usr/local/bin/phantomjs /home/phantomjs/pagegrab.js 8522

```
#### 3.通过代理请求百度页面，解析快递信息
```shell
$url = 'https://www.baidu.com/s?wd=快递单号&rsv_spt=1&rsv_iqid=0xf102131b00025a56&issp=1&f=8&rsv_bp=1&rsv_idx=2&ie=utf-8&rqlang=cn&tn=baiduhome_pg&rsv_enter=0&rsv_t=e62f5I3NWHOVrH3YAcmKgoewPs6q6MCulUM%2BNbI%2BPq3u9Q%2BZ6R5DqZip9ZudwqSXUbB%2F&oq=227857757844&rsv_pq=ffc1e7c800002ba4';
$queryUrl =  'http://192.168.0.27:8522/?url='.urlencode($url);

#curl获取queryUrl的内容，然后解析
```
