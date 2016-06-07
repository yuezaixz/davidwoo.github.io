---
layout: post
title: '初学IOS那些年踩过的坑(一)'
date: 2015-01-30 02:36
comments: true
categories: 
---
#Indicator的隐藏显示
##结论
ActivityIndicator的隐藏用stopAnimating，不要用hidden = YES。
##坑描述：
![](http://chuantu.biz/t/63/1422585664x1822611446.jpg)
代码的停止方式使用这种方式去停止
```objc
self.heartIndicator.hidden = YES;
```
带来的问题是push到别的页面pop回来后，又开始旋转了。
![](http://chuantu.biz/t/63/1422585698x-954497586.jpg)
创建了个Indicator子类，用KVO可以发现页面PUSH走和POP回来都会change hidden的值
##坑解释
所以可以看出，IOS对于显示隐藏都会对View属性进行重构，而Indicator的Hidden值在重构时候是根据animate的值来判断的，如果值为YES，那hidden的属性则为NO。
所以，千万别用hidden属性来隐藏Indicator（其他地方都操作View都如此），要用[self.heartIndicator stopAnimating];
