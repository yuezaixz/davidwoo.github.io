---
layout: post
title: 'Linux Shell编程学习笔记（四）位置变量'
date: 2014-03-12 13:18
comments: true
categories: [Linux, bash, Shell, 位置变量]
---
# 位置变量
$0 shell脚本的文件名，会包括路径
$n 第几个参数
$* 获取当前shell的所有参数，将所有的命令行参数视为一个字符串。
$# shell脚本参数的总个数
$@ 获取当前shell的所有参数，并保留在内嵌在参数中的任何空白，常用于传参
```bash

/bin/sh
echo '$0:'$0
echo '$1:'$1
echo '${10:}'${10}
echo '$#:'$#                                                                    
echo '$*:'$*
echo '$@:'$@

```

控制台输出
[David@localhost study]$ ./echoDolar.sh 'seq 11'
0:./echoDolar.sh
$1:1
${10:}10
$#:11
$*:1 2 3 4 5 6 7 8 9 10 11
$@:1 2 3 4 5 6 7 8 9 10 11


# 例子-$0、$1
做一个小例子，接受参数，参数为start则执行start函数，stop则执行stop函数。如果都不是的话，就提示错误，并提示如何输入正确的命令。例子中用到了$0和$1。


```bash
start() {
    echo "start"
}
stop(){
    echo "stop"
}

erro(){
    echo "Argument is erro ,please input what like '$0 start|stop'"
}

if [ $# -ne 1 ];then
    echo "Argument's number is Erro,please input only one argument."
else
    case "$1" in
        start)
            start
            ;;
        stop)
            stop
            ;;
        *)
            erro
            ;;

    esac
fi

```

控制台：
[David@localhost study]$ ./operByName.sh  start
start
[David@localhost study]$ ./operByName.sh  stop
stop
[David@localhost study]$ ./operByName.sh  erro
Argument is erro ,please input what like './operByName.sh start|stop'


# 例子-$*、$@
用个例子，可以看出，加了引号后$*变成一个字符串，而$@仍然是多个。
控制台：
[David@localhost study]$ set -- "I am" David Wu.
[David@localhost study]$ echo $#
3
[David@localhost study]$ echo $*
I am David Wu.
[David@localhost study]$ echo $@
I am David Wu.
[David@localhost study]$ for i in $*;do echo $i;done
I
am
David
Wu.
[David@localhost study]$ for i in $@;do echo $i;done
I
am
David
Wu.
[David@localhost study]$ for i in "$@";do echo $i;done
I am
David
Wu.
[David@localhost study]$ for i in "$*";do echo $i;done
I am David Wu.


