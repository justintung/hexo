---
title: erlang dialyzer 使用
date: 2019-04-10 23:17:32
tags:
- erlang
- dialyzer
categories:
- Linux
---
1.环境变量设置
```
cd I:\www\erlang\test
//powershell
$Env:HOME="I:\www\erlang\test"
//$Env:HOME

//cmd
set HOME=I:\www\erlang\test
//echo %HOME%

linux
HOME=/coding/codes/erlang/test
```
2.生成 .dialyzer_plt缓存文件
```
dialyzer --build_plt --apps erts kernel stdlib mnesia
```
3.编译.erl文件
```
erlc +debug_info demo.erl
```
4.执行dialyzer分析文件
```
dialyzer -c demo.beam
```