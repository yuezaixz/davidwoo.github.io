---
layout: post
title: 'Python版FizzBuzzWhizz(一)--第一晚'
date: 2014-05-02 13:35
comments: true
categories: [Python, 设计, fizzbuzzwhizz]
---
#1 概述
[该项目github地址点这](https://github.com/yuezaixz/fizzBuzzWhizz)
ThoughtWorks这个被评为世界上最难面试的IT公司，最近搞了个“抛弃简历，用代码说话”的活动，其实就是出了一道题，让程序员们编码实现后发给他们。题目是英国小学生练习除法的一个游戏，比如100名学生玩这个游戏。游戏的规则是：

*	你首先说出三个不同的特殊数，要求必须是个位数，比如3、5、7。
*	让所有学生拍成一队，然后按顺序报数。
*	学生报数时，如果所报数字是第一个特殊数（3）的倍数，那么不能说该数字，而要说Fizz；如果所报数字是第二个特殊数（5）的倍数，那么要说Buzz；如果所报数字是第三个特殊数（7）的倍数，那么要说Whizz。
*	学生报数时，如果所报数字同时是两个特殊数的倍数情况下，也要特殊处理，比如第一个特殊数和第二个特殊数的倍数，那么不能说该数字，而是要说FizzBuzz, 以此类推。如果同时是三个特殊数的倍数，那么要说FizzBuzzWhizz。
*	学生报数时，如果所报数字包含了第一个特殊数，那么也不能说该数字，而是要说相应的单词，比如本例中第一个特殊数是3，那么要报13的同学应该说Fizz。如果数字中包含了第一个特殊数，那么忽略规则3和规则4，比如要报35的同学只报Fizz，不报BuzzWhizz。

现在，我们需要你完成一个程序来模拟这个游戏，它首先接受3个特殊数，然后输出100名学生应该报数的数或单词。比如，
输入 3,5,7 输出（片段）
1 2 Fizz 4 Buzz Fizz Whizz 8 Fizz Buzz 11 Fizz Fizz Whizz FizzBuzz 16 17 Fizz 19 Buzz … 一直到100
那3个单词我是完全不能理解，所以我稍微把题目改了下，改成了Ha Hei Ah。
#2 实现
这个题目我自己用Python、Java、node.js都实现了一次，Java主要是用责任链设计模式，Python则通过一行代码实现逻辑功能，node.js类似python。本文主要讲的是完成python的思路。
#3 分析
##3.1 可能的情况
首先分析，1-100之间有几种情况：

1) 数字中含有3。
2) 3的倍数，而不是5、7的倍数。
2) 5的倍数，而不是3、7的倍数。
2) 7的倍数，而不是3、5的倍数。
2) 既是3的倍数，又是5的倍数，但不是7的倍数。
2) 既是3的倍数，又是7的倍数，但不是5的倍数。
2) 既是5的倍数，又是7的倍数，但不是3的倍数。
2) 既是3的倍数，又是5的倍数，又是7的倍数。
3) 既不是3的倍数，又不是5的倍数，又不是7的倍数。
##3.2 条件间的关系
数字代表优先级，数字相同表示同级别，且数字相同的情况不会同时出现。
##3.3 组合条件
因此，我们可以把数字不同的条件用or连接，序号相同的用and连接。
比如这样：条件1 or (条件2  and 条件3 and 条件4……) or 条件10
##3.4 再思考
有没更好的方式呢？再想了下，条件2虽然情况很多种，输出无非是3个单词的集合，且顺序固定。那么我们把条件改一下。
条件1 or ( (输出Ha)  and (输出Hei)  and (输出Ah)) or 条件10
另外，想到python的一个特殊的语法 str[x:y:z]，大概就是用来截取字符串的，比如’abcdefg’[1:6:2]的输出会是bd。
那怎么用呢，比如’Ha’[0*2::]这样就是输出’Ha’[x*2::]就是输出空字符（x>=1）。那么如果n能被3整除，那么n%3就为0。很好，Perfect！
#4 实现思路
##4.1 逻辑实现
既然有了思路，那么先在python的shell里实现下
```python
for x in range(1,100):
print ("Ha" if str(x).find(str(3)) >=0 else '' ) or "Ha"[x%3*2::]+"Hei"[x%5*3::] +"Ah"[x%7*2::] or x
```
看下面的输出信息可以发现，很好，已经可以得到想要的结果了。（这中间当然还有一些Fix，一部到位那么是神人啊）

```
1
2
Ha
4
Hei
Ha
Ah
8
Ha
Hei
11
Ha
Ha
Ah
HaHei
16
17
Ha
19
Hei
HaAh
22
Ha
Ha
Hei
26
Ha
Ah
29
Ha
Ha
Ha
……(省略)
```

##4.2 性能和显示的优化
这时候效果是达到了，但是没有优化,现在优化，改用xrange在数字量比较大的时候性能可以提升，判断含有3的代码也优化同后面形式一样，并加上原数字进行提示。
```python
for x in xrange(1,101):
        print x,"Ha"[0 if str(x).find(str(num_a))>-1 else 2:] or "Ha"[x%num_a*2::]+"Hei"[x%num_b*3::] +"Ah"[x%num_c*2::] or x;
```
这样改进下性能好了一些，而且提示信息更友好了。
##4.3 单元测试
写模块前，用先写单元测试的方法可以更好的设计松耦合的函数方法。
```python testHaHeiAh.py
#!/usr/bin/env python  
# -*- coding:utf-8 -*-
  
import unittest
import haHeiAh as testClass
    
class testHaHeiAh(unittest.TestCase):
    #初始化工作
    def setUp(self):
        pass
    #推出清理工作
    def tearDown(self):
        pass
    def testHhaHandle(self):
        #测试被第一个数（3）整除
        self.assertEqual(testClass.hhaHandle(3,3,5,7),’Ha’,'fail')
        #测试不被任何数整除，且不含有第一个数（3）
        self.assertEqual(testClass.hhaHandle(4,3,5,7),4,'fail')
        #测试被第二个数（5）整除
        self.assertEqual(testClass.hhaHandle(5,3,5,7),’Hei’,'fail')
        #测试被第三个数（7）整除
        self.assertEqual(testClass.hhaHandle(7,3,5,7),’Ah’,'fail')
        #测试同时被两个数整除
        self.assertEqual(testClass.hhaHandle(15,3,5,7),’Ha’+’Hei’,'fail')
        #测试同时被三个数整除
        self.assertEqual(testClass.hhaHandle(105,3,5,7),’Ha’+’Hei’+’Ah’,'fail')
        #以下测试除3 5 7外别的数字组合
        self.assertEqual(testClass.hhaHandle(16,4,6,9),’Ha’,'fail')
        self.assertEqual(testClass.hhaHandle(18,3,6,9),’Ha’+’Hei’+’Ah’,'fail')
        self.assertEqual(testClass.hhaHandle(24,3,6,8),’Ha’+’Hei’+’Ah’,'fail')
        self.assertEqual(testClass.hhaHandle(24,1,2,3),’Ha’+’Hei’+’Ah’,'fail')

if __name__ == '__main__':
    unittest.main()
```
现在运行单元测试结果当然是失败，因为这个方法都没有。
##4.4 整合成模块
上面的代码都还只是面向过程级别的，没有可复用性，那么我们就重构一下。
模块：
```python haHeiAh.py
def haHeiAh(num_a,num_b,num_c):
    print 'the div num is ',num_a,num_b,num_c;
    return [str(num) + ":" + str(hhaHandle(num,num_a,num_b,num_c)) for num in xrange(1,101)];

def hhaHandle(num,num_a,num_b,num_c):
    return "Ha"[0 if str(num).find(str(num_a))>-1 else 2:] or "Ha"[num%num_a*2::]+"Hei"[num%num_b*3::] +"Ah"[num%num_c*2::] or num;
```
##4.5 执行单元测试
写好模块接可以执行单元测试了，执行的过程会结合了很多Fix。

```

[David@localhost haheiahx]$ python testHaHeiAh.py 
----------------------------------------------------------------------
Ran 1 test in 0.000s
OK

```

##4.6 可运行测试入口来重构
可运行测试方法：
```python test.py
#!/usr/bin/python
# -*- coding: utf-8 -*-

def testHaHeiAh(p_a=3,p_b=5,p_c=7):
    moduleName = "haHeiAh";
    methodName = "haHeiAh";

    module = __import__(moduleName);
    method = getattr(module,methodName);
    print method(p_a,p_b,p_c);
import sys
if __name__ == '__main__':
    params = [int(x) for x in sys.argv[1::] if x.__len__() == 1];
    testHaHeiAh() if params.__len__() < 3 else testHaHeiAh(params[0],params[1],params[2]);
        
```
这下代码好多了。而且3个特殊数字也可以通过用户自己来输入了。具体看下面操作
```

[David@localhost haheiah]$ python test.py 
the div num is  3 5 7
['1:1', '2:2', '3:Ha', '4:4', '5:Hei', '6:Ha', '7:Ah', '8:8', '9:Ha', '10:Hei', '11:11', '12:Ha', '13:Ha', '14:Ah', '15:HaHei', '16:16', '17:17', '18:Ha', '19:19', '20:Hei', '21:HaAh', '22:22', '23:Ha', '24:Ha', '25:Hei', '26:26', '27:Ha', '28:Ah', '29:29', '30:Ha', '31:Ha', '32:Ha', '33:Ha', '34:Ha', '35:Ha', '36:Ha', '37:Ha', '38:Ha', '39:Ha', '40:Hei', '41:41', '42:HaAh', '43:Ha', '44:44', '45:HaHei', '46:46', '47:47', '48:Ha', '49:Ah', '50:Hei', '51:Ha', '52:52', '53:Ha', '54:Ha', '55:Hei', '56:Ah', '57:Ha', '58:58', '59:59', '60:HaHei', '61:61', '62:62', '63:Ha', '64:64', '65:Hei', '66:Ha', '67:67', '68:68', '69:Ha', '70:HeiAh', '71:71', '72:Ha', '73:Ha', '74:74', '75:HaHei', '76:76', '77:Ah', '78:Ha', '79:79', '80:Hei', '81:Ha', '82:82', '83:Ha', '84:HaAh', '85:Hei', '86:86', '87:Ha', '88:88', '89:89', '90:HaHei', '91:Ah', '92:92', '93:Ha', '94:94', '95:Hei', '96:Ha', '97:97', '98:Ah', '99:Ha', '100:Hei']
[David@localhost haheiah]$ python test.py 3 6 9
the div num is  3 6 9
['1:1', '2:2', '3:Ha', '4:4', '5:5', '6:HaHei', '7:7', '8:8', '9:HaAh', '10:10', '11:11', '12:HaHei', '13:Ha', '14:14', '15:Ha', '16:16', '17:17', '18:HaHeiAh', '19:19', '20:20', '21:Ha', '22:22', '23:Ha', '24:HaHei', '25:25', '26:26', '27:HaAh', '28:28', '29:29', '30:Ha', '31:Ha', '32:Ha', '33:Ha', '34:Ha', '35:Ha', '36:Ha', '37:Ha', '38:Ha', '39:Ha', '40:40', '41:41', '42:HaHei', '43:Ha', '44:44', '45:HaAh', '46:46', '47:47', '48:HaHei', '49:49', '50:50', '51:Ha', '52:52', '53:Ha', '54:HaHeiAh', '55:55', '56:56', '57:Ha', '58:58', '59:59', '60:HaHei', '61:61', '62:62', '63:Ha', '64:64', '65:65', '66:HaHei', '67:67', '68:68', '69:Ha', '70:70', '71:71', '72:HaHeiAh', '73:Ha', '74:74', '75:Ha', '76:76', '77:77', '78:HaHei', '79:79', '80:80', '81:HaAh', '82:82', '83:Ha', '84:HaHei', '85:85', '86:86', '87:Ha', '88:88', '89:89', '90:HaHeiAh', '91:91', '92:92', '93:Ha', '94:94', '95:95', '96:HaHei', '97:97', '98:98', '99:HaAh', '100:100']
```

#5 再思考
以上部分是在周二下班后完成的，做到那里也就洗洗睡了。隔天拿起程序不禁又看了下，发现了很多不足，在下一章节我将介绍下还有哪些改进的地方。
