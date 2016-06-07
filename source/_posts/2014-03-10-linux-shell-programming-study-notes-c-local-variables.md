---
layout: post
title: 'Linux Shell编程学习笔记（三）本地变量 '
date: 2014-03-10 14:43
comments: true
categories: [Linux, bash, Shell]
---
## 本地变量
### 显示本地变量
set命令可以显示当前环境所有变量，包括本地变量和环境变量。
```bash
[David@localhost ~]$ aaa=bbb
[David@localhost ~]$ env | grep aaa
[David@localhost ~]$ set | grep aaa
aaa=bbb
[David@localhost ~]$ export aaa
[David@localhost ~]$ env | grep aaa
aaa=bbb
[David@localhost ~]$ set | grep aaa
aaa=bbb
```
### 定义本地变量
变量名=value 会解析，用于定义连续的比较简单的字符串数字
变量名=’value’  所见即所得，单引号里面的是什么就会显示什么，不会进行解析
变量名=”value” 会解析，一般字符串定义推荐使用
```bash
[David@localhost ~]$ a=192.168.0.248
[David@localhost ~]$ b='192.168.0.248'
[David@localhost ~]$ c="192.168.0.248"
[David@localhost ~]$ echo "a=$a"
a=192.168.0.248
[David@localhost ~]$ echo "b=$b"
b=192.168.0.248
[David@localhost ~]$ echo "c=$c"
c=192.168.0.248
[David@localhost ~]$ a=192.168.1.2-$a
[David@localhost ~]$ echo $a
192.168.1.2-192.168.0.248
[David@localhost ~]$ b='192.168.1.2-$b'
[David@localhost ~]$ echo $b
192.168.1.2-$b
[David@localhost ~]$ c="192.168.1.2-$c"
[David@localhost ~]$ echo $c
192.168.1.2-192.168.0.248
```

### 通过反引号用本地变量引用linux命令
用反引号将命令括起来，可以赋给本地变量。显示的时候就会执行该变量。但是记得不能用单引号。推荐使用$(date)方式处理命令，因为反引号不容易发现且会给当成单引号。
```
[David@localhost ~]$ userCommand=`whoami`
[David@localhost ~]$ echo $userCommand 
David
[David@localhost ~]$ echo "`date`"
2014年 03月 10日 星期一 22:07:38 CST
[David@localhost ~]$ echo `date`
2014年 03月 10日 星期一 22:07:44 CST
[David@localhost ~]$ echo '`date`'
`date`
```
### 规范
数字或简单字符串不用引号；
没特殊情况一般用双引号；
原样输出用双引号；
常量尽量用大写，多单词用下划线连接，特别是全局变量；
引用最好是${VAR_NAME}；
避免使用无意义的名字，比如COUNT；
在函数里定义变量，记得用local来声明，例如：local i；
使用$(date)方式处理命令，因为反引号不容易发现且会给当成单引号；

