---
layout: post
title: 'Linux Shell编程学习笔记（五）-进程变量'
date: 2014-03-13 00:02
comments: true
categories: [Linux, bash, Shell, 进程变量]
---
# 进程变量
## $$ 获得当前shell的PID
```bash $$查看当前shell
[David@localhost study]$ ps
  PID TTY          TIME CMD
 4410 pts/7    00:00:00 ps
27865 pts/7    00:00:00 bash
[David@localhost study]$ echo $$
27865
```
## 例子
运行个脚本，把PID写到一个文件中去。然后不让改脚本结束，打开另一个窗口查看PID是否正确。
```bash
[David@localhost study]$ cat m.sh
echo "$$" >m.pid
while true
do
    sleep 1
done
[David@localhost study]$ sh m.sh
```
再打开一个窗口输入命令查看是否正确
```bash
[David@localhost study]$ cat m.pid
6022
[David@localhost study]$ ps -ef|grep m.sh
David     6022  4430  0 23:07 pts/9    00:00:00 sh m.sh
David     6128 29747  0 23:08 pts/8    00:00:00 grep m.sh
```
## $? 获取执行上一个指令的返回值（0为成功，非0为失败）
```bash $?查看上一指令的结果
[David@localhost study]$ cat testPrevResult.sh 
#/bin/sh

case "$1" in
    success)
        exit 0
        ;;
    *)
        exit 1
        ;;
esac
[David@localhost study]$ sh testPrevResult.sh success
[David@localhost study]$ echo $?
0
[David@localhost study]$ echo $?
0
[David@localhost study]$ sh testPrevResult.sh erro
[David@localhost study]$ echo $?
1
```

```
[David@localhost study]$ hahha
bash: hahha: command not found
[David@localhost study]$ echo $?
127
[David@localhost study]$ pwd
/home/David/shellwork/study
[David@localhost study]$ echo $?
0
```
## 常见返回码
0 成功
2 权限拒绝
1-125 表示运行失败，脚本命令、系统命令错误或参数错误
126 找到该命令但是无法执行
127 未找到要运行的命令
128 命令被系统强制结束

## 例子
压缩当前文件夹，如果成功则提示。
```bash
[David@localhost study]$ cat zipCurr.sh 
#/bin/bash
cd ..
tar zcf studyShell.tar.gz ./study
if [ $? -eq 0 ];then
    echo "tar success"
else
    echo "tar erro"
fi
[David@localhost study]$ sh zipCurr.sh 
tar success
[David@localhost study]$ cd ..
[David@localhost shellwork]$ ls
study  studyShell.tar.gz
```
## #!上一个进程的PID
不常用，不细说。
## #_上一个进程的最后一个参数
也不常用，举个栗子带过。
```
[David@localhost study]$ sh m.sh 
已杀死
[David@localhost study]$ echo $_
m.sh
```
