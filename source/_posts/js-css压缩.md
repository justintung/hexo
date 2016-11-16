---
title: js、css压缩
date: 2016-06-25 23:49:26
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
#执行./uploads-baiduyun.sh &
#要求：必须安装inotify
#yum install -y inotify-tools
minExt=("js" "css")
excludeExt=("yuicompressor" "html" "htm")
#监听目录
uploadsDir="/home/wwwroot/js.knowstep.com/branches"
bos="18183-js"
/usr/bin/inotifywait --exclude '(\.svn|\.tmp)' -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w%f %e' --event modify,attrib,moved_to,create,delete $uploadsDir |  while read  date time file event
do
    case $event in
          CREATE|MOVED_TO)
                    #排除临时文件
                    ext=${file##*.}
                    ext=$(echo $ext | tr '[A-Z]' '[a-z]')
                    if [[ "${excludeExt[@]}" =~ $ext ]];then
                        continue
                    fi
                    if [[ "$file" =~ ".yuicompressor." ]];then
                        continue
                    fi
                    if [[ "${minExt[@]}" =~ $ext ]];then
                        #压缩
                        #uglifyjs "${file}" -o "${file}.yuicompressor"
                        java -jar /usr/bin/yuicompressor-2.4.8.jar --type "${ext}" "${file}" > "${file}.yuicompressor.${ext}"

                        #上传到bos
                        dest=${file/#$uploadsDir\//}
                        bce bos cp "${file}.yuicompressor.${ext}" "bos:/${bos}/${dest}" > /dev/null 2>&1
                        rm -rf "${file}.yuicompressor.${ext}"
                        #echo "crete,move${event} >>> ${file} >>> ${dest}"
                    else
                        dest=${file/#$uploadsDir\//}
                        #上传到bos
                        bce bos cp "${file}" "bos:/${bos}/${dest}" > /dev/null 2>&1
                        #echo "crete,move${event} >>> ${file} >>> ${dest}"
                    fi
            ;;
		  DELETE)
                    #排除临时文件
                    ext=${file##*.}
                    ext=$(echo $ext | tr '[A-Z]' '[a-z]')
                    if [[ "${excludeExt[@]}" =~ $ext ]];then
                        continue
                    fi
                    if [[ "$file" =~ ".yuicompressor." ]];then
                        continue
                    fi
                    #从bos删除
                    dest=${file/#$uploadsDir\//}
                    bce bos rm --yes "bos:/${bos}/${dest}"
                    #echo "rm ${event} >>> ${file} >>> ${dest}"
            ;;	
      esac
done

```