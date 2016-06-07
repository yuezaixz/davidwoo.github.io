---
layout: post
title: 'Linux Shell编程学习笔记（二）全局变量'
date: 2014-03-10 12:46
comments: true
categories: [Linux, bash, Shell]
---
# 变量
## 全局环境变量
### 文件
/etc/profile 这个文件中预设了系统的环境变量。
修改后记得 . /etc/profile(中间有个空格)
### 启动执行路径
/etc/profile.d 将脚本放到该目录下，登陆用户的时候就会自动执行脚本。
### 规范
记得，传统上，所有全局环境变量要大写，注意，大小写是有区别的。
### 定义
export myname=’David’
export MYNAME='David_Wu'
```
[David@localhost study]$ export MYNAME='David_Wu'
[David@localhost study]$ echo $MYNAME
David_Wu
[David@localhost study]$ echo $myname
David
```
### 常用的全局变量
HOME目录
UID登陆用户ID
USER登陆用户
OSTYPE操作系统类似
HISTSIZE历史操作大小
HISTFILESIZE历史文件行数
TMOUT 超时退出时间
IFS系统分隔符
### 取消全局变量
unset MYNAME
```bash 控制台
[David@localhost ~]$ export MYNAME=David
[David@localhost ~]$ echo $MYNAME
David
[David@localhost ~]$ unset MYNAME
[David@localhost ~]$ echo $MYNAME
```

### 用户环境变量和系统环境变量
ll ~ -al命令可以看到家目录下隐藏的用户环境变量
etc/profile就是系统环境变量
