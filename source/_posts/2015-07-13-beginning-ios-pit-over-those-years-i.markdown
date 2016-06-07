---
layout: post
title: "初学IOS那些年踩过的坑（十）"
date: 2015-07-13 16:01:53 +0800
comments: true
categories: 
---
##页面间pop和push导致的问题
问题描述：问题大概是这样，一个运动中页面RunningViewController，一个首页IndexViewController，一个结果页RunResultViewController
运动：IndexViewController--Push-->RunningViewController
结束：RunningViewController --Pop-->IndexViewController--push-->

结果IndexViewController的ViewWillAppear会弹出个UIAlertView，其逻辑会造成影响，在IOS8情况下UIAlertView不会弹出（应该是弹出后立马跳到了RunResultViewController），而IOS7情况下到RunResultViewController才弹出该UIAlertView。

该问题很简单，该逻辑其实放错位置了，应该放在ViewDidLoad中，但是从UIAlertView造成的错误逻辑去排查到该问题比较麻烦。

##TableView的load时机
问题描述：在使用FXForm做表单时候，在viewDidLoad中进行了初始化，但是初始化内容无效（同样也是排查比较麻烦，开始没怀疑这个问题）。IOS8是在ViewWillAppear之后（除非你调用了tableView.tableFooterView = … 之类方法，调用后）再进行loadView的操作，而IOS7则是在[super viewDidLoad]之后。最安全的做法就是在ViewWillAppear之后重新调用一次 [tableView reloadData];
