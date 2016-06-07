---
layout: post
title: 'Linux不是window，关机须谨慎'
date: 2014-02-15 10:47
comments: true
categories: [Linux]
---
最近饱受感冒和咳嗽的折磨，结果今天又被我的centos折磨了一番。估计是昨天异常关机导致扇区受损，还好没伤害到根目录，还能恢复，不过得长记性了，千万别像操作window那样随意。今天也就把linux关机前要注意的写一写。

#1 为什么要正确的关机
Linux系统不像window系统那样是单用户、假多任务的情况。在linux中，由于每个程序都是在后台执行的，因此在你看不到的背后，其实可能有很多人同时在你的主机上工作。
还有个问题就是，若不能正常关机，后台的程序来不及将数据写回到文件中，有些服务的文件就会有问题，就可能造成文件系统的损坏。
#2 关机要注意什么
##2.1 观察系统的使用状态
首先可以用who命令查查谁在线，可以发出who命令。
```
[David@localhost ~]$ who
David    tty1         2014-01-06 21:28 (:0)
David    pts/1        2014-01-06 21:46 (:0.0)
David    pts/12       2014-02-15 14:53 (:0.0)
```
然后要看看网络的联机状态
```
[David@localhost ~]$ netstat -a
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address               Foreign Address             State      
tcp        0      0 *:sunrpc                    *:*                         LISTEN      
tcp        0      0 *:28017                     *:*                         LISTEN      
//省略1万字
```
最后在看看后台的进程状态ps –aux。
##2.2 将数据写入磁盘
Linux系统为了加快数据的读取速度，默认情况下某些数据将不会直接写入磁盘，而是先暂存在内存中。但是万一异常关机，这些数据将不能写入磁盘机，造成数据更新不正常，因此，关机前最好执行几次sync命令来写入磁盘。当然shutdown/reboot/halt命令都会调用sync，但是最好也还是要执行几次。
##2.3 使用正确的关机命令
关机（Shutdown）、重启（reboot\halt\poweroff）。
###2.3.1 Shutdown
这是最常用的关机命令，该命令会通知系统内的各个进程，并且通知系统中运行级别内的一些服务来关闭。该命令只能有root用户来操作。
```
[David@localhost ~]$ shutdown -k now 'abcd'
shutdown: Need to be root
```
可以用man shutdown命令，这个命令当然不是用来“找男人”的，而是用来查看帮助文档的。下面稍微对比较有用的信息翻译了下。
命令的格式为：shutdown [OPTION]...  TIME [MESSAGE]
-r     将系统服务停了后就重新启动.
-h     将系统服务停了后就立即关机.
-H     将系统服务停了后就立即挂起.
-P     将系统服务停了后就立即切断电源.
-c     取消关机，该命令不需要指定时间，第一个参数就是通知信息。
-k     只是发送关机提示信息，并不真的关机.
例子：
Shutdown –h now 立即关机
Shutdown –h 20:25 不解释干嘛的
Shutdown –h +10 系统字啊管10分钟后自动关机
Shutdown –r now “我要重启了” 立即重启，并发送通知。
###2.3.2 Reboot
该命令基本和Shutdown –r now效果一样。最好这样执行sync;sync;sync;reboot;
#3 扇区错乱问题修复
该问题就是因为异常关机导致，如果根目录没有损坏，那么再提示press root pass word or ctrl+D，这时输入root的口令登入系统进行单人单机的维护，输入fsck /dev/hda7(/dev/hda7为坏的磁盘块，按实际输入)。最后再询问clear?的时候输入Y就好了。修复完毕后reboot。
如果根目录坏了，那就麻烦了。
 

