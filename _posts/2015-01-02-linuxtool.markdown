---
layout:     post
title:      "实用Linux工具命令备忘"
subtitle:   " \"工欲善其事，必先利其器.\""
date:       2015-01-02 19:00:00
author:     "WangChongjie"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 实用工具库
---
开发和生产环境中，我们需要对物理机和平台服务进行治理、监控、运维。用合适的工具做正确的事情，Use The Right Tool Do 
Right Things。本文，整理汇总常用的工具命令。

## 查看远端服务器端口的类型和状态

```xml
  命令行：
  nmap 186.186.186.186 -p 8080
  
  结果示例：
  PORT     STATE SERVICE
  8080/tcp open  unknown
```

## JAVA Dump内存和线程

```xml
  命令行：
  jmap -heap:format=b app进程号
  jstack -l tomcat进程号 > out
  jstat  -gccause pid 86400 > /tmp/pid.jstat
```

## Redis数据pipeline方式导入

```xml
  Redis数据pipeline导入：
  awk '{print "*4\r\n$4\r\nHSET\r\n$9\r\nbessitekv\r\n$"length($1)"\r\n"$1"\r\n$"length($2)"\r\n"$2"\r"}' sitekv.out | ./redis-cli -h 1.1.1.1 -p 8888 --pipe 

  Redis数据导出：
  echo "hgetall sitekv"|./redis-cli -h 1.1.1.1 -p 8888 -a passwd >> sitekv.dat
  
  Redis数据导出数据处理（先删掉最后一行）：
  awk '{if((NR%2)==1) print $0}' sitekv.dat > sitekv.dat.1
  awk '{if((NR%2)==0) print $0}' sitekv.dat > sitekv.dat.2
  awk 'FILENAME=="sitekv.dat.1"{MAP[NR]=$0;line=NR} FILENAME=="sitekv.dat.2"{print MAP[NR-line]"\t"$0 }' sitekv.dat.1 sitekv.dat.2>sitekv.out

  MySQL数据导入：
  mysql -h1.1.1.1 -P6666 -uxxx -pxxx -e "load data local infile sitekv.out into table uv_kv.newdsp_site_kv"
```

## MongoDB数据导入导出

```xml
  mongoimport --ignoreBlanks -h127.0.0.1 --port 8017 -ddb_wangcj -ccollec_wangcj -f"id,key" --type "tsv" --file data.dat
  mongoexport -h127.0.0.1 --port 8017 -ddb_wangcj -ccollec_wangcj -f"id,key" --csv > mongo.out
```

## 控制台Console乱码处理

```xml
  ctrl-N导致的则：
  cat
  ^O
  ^D
```

## 删除其它文件

```xml
  rm -f `ls | grep -iv -E 20151208`
```

## 文件编码格式转换

```xml
  iconv -f gbk -t utf8 in_file > out_file
  iconv -f utf8 -t gbk in_file > out_file
```

## 带行号grep

```xml
  grep  –ni
```

## 文件时间戳批量替换

```xml
  find /home/work/my-web -type f -name "*.html" -exec touch -m -t "201501182200" {} \;
  find /home/work/my-web -type f -name "*.js" -exec touch -m -t "201501182200" {} \;
  find /home/work/my-web -type f -name "*.css" -exec touch -m -t "201501182200" {} \;
  find /home/work/my-web -type f -name "*.gif" -exec touch -m -t "201501182200" {} \;
  find /home/work/my-web -type f -name "*.png" -exec touch -m -t "201501182200" {} \;
  find /home/work/my-web -type f -name "*.jpg" -exec touch -m -t "201501182200" {} \;
  find /home/work/my-web -type f -name "*.swf" -exec touch -m -t "201501182200" {} \;
```

## 查看本机端口运行情况

```xml
  lsof -i :8080
```

## 查看域名的DNS相关信息

```xml
  dig wangchongjie.com
  dig -x 192.30.252.154
```

## 测算程序或命令的执行时间

```xml
  time ps aux
  
  ps aux命令的执行返回结果：
  real	0m0.031s
  user	0m0.007s
  sys	0m0.016s
```

## 删除当前目录下的所有.git目录

```xml
  find .  -name ".git" -exec rm -rf {} \;
```

## 查找硬盘占用最大的几个文件

```xml
  du -a|sort -nr|head;
```

## HTTP资源抓取

```xml
  curl -o index.html wangchongjie.com
```

## 查找特定目录下的文件cp至指定目录

```xml
  find /home/work -name *.gz -exec cp {} /tmp \;
```

## 查找进程占用内存情况

```xml
  ps -aux|awk '{print $4"\t"$11}'|grep -v MEM|sort -r
```

## 查找CPU等使用情况

```xml
  vmstat 3
```

## 查找Linux开机运行时间

```xml
  uptime
  16:18:27 up 407 days, 23:32, 11 users,  load average: 0.36, 0.12, 0.10
```

## 显示当前月份

```xml
  cal
      November 2016
  Su Mo Tu We Th Fr Sa
         1  2  3  4  5
   6  7  8  9 10 11 12
  13 14 15 16 17 18 19
  20 21 22 23 24 25 26
  27 28 29 30
```

## 显示当前日期

```xml
  date
  Wed Nov 30 16:27:23 CST 2016
```


## 列出打开的文件

```xml
  lsof
  可以查看文件被哪些进程使用
```

## 显示机器名内核等信息

```xml
  uname -a
  Linux hd01-xxx-111.cj.com 2.8.39_1-18-0-0_virtio #1 SMP Thu 
  May 14 15:30:56 CST 2015 x86_64 x86_64 x86_64 GNU/Linux
```

## 开发机通过go命令跳转

```xml
    #!/bin/sh
    
    tc177="tc-177.wangchongjie.com"
    yf155="yf-155.wangchongjie.com"
    cp104="cp-104.wangchongjie.com"
    
    if [ $# -gt 0 ] ; then
    
        case "$1" in
            "tc177" ) host="${tc177}";;
            "yf155 " ) host="${yf155 }";;
            "cp104" ) host="${cp104 }";;
        esac
    
        if [ ! -z "${host}" ] ; then
            echo "ssh work@${host}"
            ssh "work@${host}”
        else
            echo "$1 do not match any machine"
        fi
    fi
```

## 查看用户分库信息

```xml
    #!/bin/env python
    import string
    import sys
    
    #useridcode = ((userid >>> tableShardingLength) & (shardingNum-1));
    
    tableShardingLength=6
    shardingNum=8
    
    def help():
        print """Usage: dbu <userid>
        e.g. dbu 18
    """
    
    if __name__ == "__main__":
        if len(sys.argv) != 2:
            help()
            sys.exit(1)
        else:
            userid = string.atoi(sys.argv[1])
            useridcode = ((userid >> tableShardingLength) & (shardingNum-1));
            print useridcode
```

此外，还有一些强大的工具如sed、awk等，不展开介绍。