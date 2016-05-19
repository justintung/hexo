###1.[hexo](https://hexo.io/)+[maupassant](https://github.com/tufu9441/maupassant-hexo)+[yilia](https://github.com/litten/hexo-theme-yilia)
集成了maupassant主题和yilia主题，默认为maupassant主题
###2.安装
a.hexo安装
```shell
npm install hexo-cli -g
```
b.集成hexo下载
```shell
git clone https://github.com/justintung/hexo.git
```
c.运行代码
```shell
npm install hexo-renderer-jade --save
npm install hexo-renderer-sass --save
cd hexo
npm install
hexo server
```
d.访问 [http://127.0.0.1:4000](http://127.0.0.1:4000)
###3.国内npm代理设置
```shell
cd ~
vi .npmrc
```
.npmrc内容如下：
```shell
registry = https://registry.npm.taobao.org
```
