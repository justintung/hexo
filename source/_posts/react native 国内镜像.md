---
title: react native 国内镜像
date: 2017-09-10 08:59:52
tags:
- react native
- 安装
categories:
- 架构/软件
---
1.npm 源设置
```shell
npm config set registry https://registry.npm.taobao.org
npm config set disturl https://npm.taobao.org/dist
```
或直接在/home/用户名/.npmrc中添加两行
```shell
registry=https://registry.npm.taobao.org
disturl=https://npm.taobao.org/dist
```
2.gradlew下载的gradle配置
编辑 react native项目根目录/android/wrapper/gradle-wrapper.properties文件
修改如下，改为本地包：
```shell
#distributionUrl=https\://services.gradle.org/distributions/gradle-2.14.1-all.zip
distributionUrl=file:///data/coding/env/android/gradle-2.14.1-all.zip
```
3.jcenter下载地址修改
编辑 react native项目根目录/build.gradle文件,
修改如下：
```shell
#jcenter()
jcenter(){url 'http://jcenter.bintray.com/'}
```