---
title: nginx-ssi
date: 2016-05-21 10:36:14
tags:
- nginx
- ssi
categories:
- Nginx
---
####1.相关指令
ssi
语法: ssi [ on | off ]
默认值: ssi off
作用域: http, server, location 在location作用域中将启用SSI文件处理.

ssi_last_modified
语法:	ssi_last_modified on | off
默认值:ssi_last_modified off
作用域:http, server, location
允许在SSI处理相应缓存的同时，从原始相应中保存“Last-Modified”头

ssi_min_file_chunk
语法:	ssi_min_file_chunk size
默认值:ssi_min_file_chunk 1k
作用域:http, server, location
Sets the minimum size for parts of a response stored on disk, starting from which it makes sense to send them using sendfile.
在sendfile开启时，设置存储到硬盘的响应的最小size

ssi_silent_errors
语法: ssi_silent_errors [on|off]
默认值: ssi_silent_errors off
作用域: http, server, location
在处理SSI文件出错时不输出错误提示:"[an error occurred while processing the directive] "

ssi_types
语法: ssi_types mime-type [mime-type ...]
默认值: ssi_types text/html
作用域: http, server, location
使SSI能够处理除"text/html"之外的mime-types

ssi_value_length
语法: ssi_value_length length
默认值: ssi_value_length 256
作用域: http, server, location
定义SSI允许使用的参数长度