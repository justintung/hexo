title: shell特殊变量
date: 2016-04-26 22:14:36
tags:
- shell
---
1.`$`变量含义($0,$1,$2,$#,$@)
#####以./xx.sh start 112 113为例

$#------执行脚本时传递的参数个数（$#=3）
$@------传递的参数（$@=start 112 113）
$0-------脚本文件本身($0=./xx.sh)
$1-------传递给脚本的第一个参数($1=start)
$2-------传递给脚本的第一个参数($2=112)
$3-------传递给脚本的第一个参数($3=113)
$$-------当前进程的pid
$()与``-------命令替换(command subsititution)
$(())-------数学运算（arithmetical operation）

2.`${}`用法
##### 假设我们定义了一个变量为： 
file=/dir1/dir2/dir3/my.file.txt 

#####我们可以用 ${ } 分别替换获得不同的值： 
${file#*/}：拿掉第一条 / 及其左边的字符串：dir1/dir2/dir3/my.file.txt 
${file##*/}：拿掉最后一条 / 及其左边的字符串：my.file.txt 

${file#*.}：拿掉第一个 . 及其左边的字符串：file.txt 
${file##*.}：拿掉最后一个 . 及其左边的字符串：txt 

${file%/*}：拿掉最后条 / 及其右边的字符串：/dir1/dir2/dir3 
${file%%/*}：拿掉第一条 / 及其右边的字符串：(空值) 

${file%.*}：拿掉最后一个 . 及其右边的字符串：/dir1/dir2/dir3/my.file 
${file%%.*}：拿掉第一个 . 及其右边的字符串：/dir1/dir2/dir3/my 

###记忆的方法为：

**\# 是去掉左边(在鉴盘上 # 在 $ 之左边)**
**% 是去掉右边(在鉴盘上 % 在 $ 之右边)**
**单一符号是最小匹配﹔两个符号是最大匹配。 **


${file:0:5}：提取最左边的 5 个字节：/dir1 
${file:5:5}：提取第 5 个字节右边的连续 5 个字节：/dir2 


我们也可以对变量值里的字符串作替换： 
${file/dir/path}：将第一个 dir 提换为 path：/path1/dir2/dir3/my.file.txt 
${file//dir/path}：将全部 dir 提换为 path：/path1/path2/path3/my.file.txt 

利用 `${ }` 还可针对不同的变量状态赋值(没设定、空值、非空值)：
${file-my.file.txt} ：假如 $file 为空值，则使用 my.file.txt 作默认值。(保留没设定及非空值) 
${file:-my.file.txt} ：假如 $file 没有设定或为空值，则使用 my.file.txt 作默认值。 (保留非空值) 

${file+my.file.txt} ：不管 $file 为何值，均使用 my.file.txt 作默认值。 (不保留任何值) 
${file:+my.file.txt} ：除非 $file 为空值，否则使用 my.file.txt 作默认值。 (保留空值) 

${file=my.file.txt} ：若 $file 没设定，则使用 my.file.txt 作默认值，同时将 $file 定义为非空值。 (保留空值及非空值) 
${file:=my.file.txt} ：若 $file 没设定或为空值，则使用 my.file.txt 作默认值，同时将 $file 定义为非空值。 (保留非空值) 

${file?my.file.txt} ：若 $file 没设定，则将 my.file.txt 输出至 STDERR。 (保留空值及非空值)) 
${file:?my.file.txt} ：若 $file 没设定或为空值，则将 my.file.txt 输出至 STDERR。 (保留非空值) 

还有，${`#var`} 可计算出变量值的长度： 
${`#file`} 可得到 27 ，因为 /dir1/dir2/dir3/my.file.txt 刚好是 27 个字节
