---
layout: post
title: '初学IOS那些年踩过的坑(四)'
date: 2015-02-28 02:47
comments: true
categories: 
---
#集成友盟后报错
错误日志如下：
```
15-02-12 09:26:47.206 Runmove[2532:13249] NSScanner: nil string argument
2015-02-12 09:26:47.206 Runmove[2532:13249] Umeng: Read binary image info failed!
```
分析：其实不是友盟问题，是自己程序中有了个错误，只是打开了友盟的调试开关，所以报了友盟错误。
```
//    [MobClick setLogEnabled:YES]; 
```
解决：注释该行代码后可以正常报告出程序中的异常

#tableview下方空白处的seprator line去除
tableview的黑魔法
```objc
self.tableView.tableFooterView = [UIView new];
```
#navigationbar的左滑返回
```objc

    self.navigationItem.leftBarButtonItem = [[UIBarButtonItem alloc]
                                         initWithImage:img
                                         style:UIBarButtonItemStylePlain
                                         target:self
                                         action:@selector(onBack:)];
self.navigationController.interactivePopGestureRecognizer.delegate = (id)self;
```
#autolayout的nib和手写并用的实践
使用nib做界面，但是layoutAttribute有些又需要根据适配来适配，nib+手写混写好烦恼啊，感觉睡不着觉了
其实错了，混写其实很简单的，可以像拉button一样的拉你的约束.nslayoutattribute也是可以拉线的，这样一切都太平了

#画面不流畅
画面不流畅可能是因为一些透明视图影响了fps，可以去掉这些透明视图
方法：Product->Profile->Core Animation->右下第二个选项卡中勾上Color Blended Layers
然后看到红色的就都是透明的视图，不需要透明效果的可以去掉透明
