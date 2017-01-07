---
title: js、css压缩
date: 2016-11-11 23:49:26
tags:
- yuicompressor
- uglifyjs
categories:
- 软件
---
1.介绍
yuicompressor 能压缩js，也能压缩css，地址: [http://yui.github.io/yuicompressor/](http://yui.github.io/yuicompressor/)
uglifyjs 只能压缩js,地址: [https://github.com/mishoo/UglifyJS/](https://github.com/mishoo/UglifyJS/)
2.使用demo
```shell
#!/bin/bash 
#程序实现uploads目录的监听，并将文件上传到bos
#执行./uploads-bos.sh &
#要求：必须安装inotify
#yum install -y inotify-tools
minExt=("js" "css")
excludeExt=("bak")
#监听目录
uploadsDir="/home/wwwroot/www.365coding.com/branches"
bos="365coding-js"
/usr/bin/inotifywait --exclude '(\.svn|\.tmp)' -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w%f %e' --event modify,attrib,moved_to,create,delete $uploadsDir |  while read  date time file event
do
    case $event in
          CREATE|MOVED_TO)
                    #排除临时文件
                    ext=${file##*.}
                    ext=$(echo $ext | tr '[A-Z]' '[a-z]')
                    path=${file%/*}

                    if [[ "${excludeExt[@]}" =~ $ext ]];then
                        continue
                    fi
                    if [[ "${minExt[@]}" =~ $ext ]];then
                        mkdir -p "/tmp/svn/${path}"
                        #压缩
                        if [ "$ext" = "css" ];then
                            /usr/bin/css-compress "${file}" "/tmp/svn/${file}"
                        else
                            uglifyjs "${file}" -o "/tmp/svn/${file}"
                            #java -jar /usr/bin/yuicompressor-2.4.8.jar --type "${ext}" "${file}" > "/tmp/svn/${file}"
                        fi
                        #上传到bos
                        dest=${file/#$uploadsDir\//}
                        bce bos cp "/tmp/svn/${file}" "bos:/${bos}/${dest}" & 
                        usleep 1000
                    else
                        dest=${file/#$uploadsDir\//}
                        #上传到bos
                        bce bos cp "${file}" "bos:/${bos}/${dest}" & 
                        usleep 1000
                    fi
            ;;
		  DELETE)
                    #排除临时文件
                    ext=${file##*.}
                    ext=$(echo $ext | tr '[A-Z]' '[a-z]')
                    if [[ "${excludeExt[@]}" =~ $ext ]];then
                        continue
                    fi
                    #从bos删除
                    dest=${file/#$uploadsDir\//}
                    bce bos rm --yes "bos:/${bos}/${dest}" &
                    usleep 1000
            ;;	
      esac
done


```
3.inotify如果监听的东西过得，需要修改一些系统参数
编辑文件 /etc/sysctl.conf ，追加或修改，值按需要修改
```shell
fs.inotify.max_user_watches=99999999
fs.inotify.max_queued_events=99999999
fs.inotify.max_user_instances=2048
```