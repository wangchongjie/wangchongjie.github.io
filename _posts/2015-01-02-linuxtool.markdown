---
layout:     post
title:      "常用Linux工具命令备忘"
subtitle:   " \"工欲善其事，必先利其器.\""
date:       2015-01-02 19:00:00
author:     "WangChongjie"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 设计模式
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

## 配置开发机间通过go命令跳转

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

此外，还有一些强大的工具如sed、awk等，不展开介绍。