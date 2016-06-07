---
layout: post
title: 'Python版FizzBuzzWhizz(二)--重构'
date: 2014-05-02 14:21
comments: true
categories: [Python, 设计, fizzbuzzwhizz]
---
#6 重构
##6.1 缺少常量
很多信息没有通过常量来读取，导致如果变化修改的成本很高。比如Ha如果我要改成Wa，那么我要修改项目中所有引用到的地方，包括单元测试。
那么引入常量类。

```python const.py
# -*- coding:utf-8 -*-
class ConstError(Exception): pass
class _const(object):
    def __setattr__(self, k, v): 
        if k in self.__dict__:
            raise ConstError
        else:
            self.__dict__[k] = v 
const = _const()
#三个转换单词
const.WORD1 = 'Fizz'
const.WORD2 = 'Buzz'
const.WORD3 = 'Whizz'
#模块名
const.MODULE_NAME = 'haHeiAh'
#方法名
const.METHOD_NAME = 'haHeiAh'
#报数人数
const.NUM_PERSON = 100
#报数间隔，如果不希望间隔请设置个大数字
const.INDENT = 15
```

然后就是把需要用到的地方做个替换。
##6.2 不友好
交互性很差，不懂程序内部基本上无法调用方法。
增加交互性输入数字，增加输入信息校验，提供退出方法，提供分屏展现功能，提示信息有不同颜色字体区分。
 ![](http://i1.tietuku.com/b82f024cd064476b.jpg)
再看看代码中的实现：
```python test.py
#!/usr/bin/python
# -*- coding: utf-8 -*- 
from const import const
from sys import exit
def testHaHeiAh(p_a=3,p_b=5,p_c=7):
    moduleName = const.MODULE_NAME
    methodName = const.METHOD_NAME
    module = __import__(moduleName)
    method = getattr(module,methodName)
    print "三个特殊数字为%d、%d和%d" %(p_a,p_b,p_c)
    method(p_a,p_b,p_c,const.NUM_PERSON)
    

def getInputInt(title):
    for i in range(0,3):
        attr = raw_input("请输入%s：" %title).strip()

        if len(attr) == 0:
            print "您输入的%s为空，请重新输入" %title
            continue
        elif len(attr) != 1:
            print "您输入的%s不是个位整数，请重新输入" %title
            continue
        else:
            try:
                return int(attr)
            except ValueError:
                print "您输入的%s不是数字，请重新输入" %title
    else:
        print "连续3次输入错误，程序退出！"
        exit()
if __name__ == '__main__':
    testHaHeiAh(getInputInt('第一个数字'),getInputInt('第二个数字') ,getInputInt('第三个数字') )
```
```python haHeiAh.py
# -*- coding:utf-8 -*-
from const import const
def haHeiAh(num_a,num_b,num_c,num_person):
    print "%d个学生开始报数："%num_person
    for num in xrange(1,num_person+1):
        if num != 1 and (num-1) %const.INDENT == 0 :
            command = raw_input("继续查看下%d个请按ENTER（回车），退出请输入q按回车:" %const.INDENT )
            if command == 'q':
                break
        print "第%s个学生报数%s！" %(str(num) , str(hhaHandle(num,num_a,num_b,num_c)))

def hhaHandle(num,num_a,num_b,num_c):
    return const.WORD1[0 if str(num).find(str(num_a)) >= 0 else len(const.WORD1):] or const.WORD1[num%num_a*len(const.WORD1)::]+const.WORD2[num%num_b*len(const.WORD2)::] +const.WORD3[num%num_c*len(const.WORD3)::] or num;
```
#7 再次运行单元测试
重构完记得再次运行单元测试。单元测试除了设计驱动外，还有对重构进行保护的作用。

```

[David@localhost haheiahx]$ python testHaHeiAh.py 
----------------------------------------------------------------------
Ran 1 test in 0.000s
OK

```
#8 文档
最后是补充文档，包括项目介绍、运行环境说明、部署脚本介绍等，具体可以连接到github上查看。这里就不贴出来了。
#9 结束
有空的话把Java的思路整理下也写成博文，node.js的就算了。思路和python的差不多。
