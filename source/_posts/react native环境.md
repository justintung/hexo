---
title: react native环境
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

2.yarn安装
```shell
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

sudo apt-get update && sudo apt-get install yarn
```
3.react相关安装

```shell
npm install -g react-native  react react-native-cli
npm install -g create-react-native-app

npm install -g create-react-app
```

4.创建react-native项目
```shell
#test-rn为项目名称
create-react-native-app test-rn
```
或使用yarn
```shell
#test-rn为项目名称
yarn create react-native-app test-rn
```

5.运行（进入对应项目）
```shell
npm start
```
或使用yarn
```shell
yarn start
```
6.create-react-native-app创建的项目生成apk文件
```shell
yarn run eject
react-native run-android
```