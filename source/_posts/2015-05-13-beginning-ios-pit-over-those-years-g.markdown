---
layout: post
title: "初学IOS那些年踩过的坑（八）"
date: 2015-05-13 11:25:59 +0800
comments: true
categories: 
---

##毛玻璃效果实现
不管是用FXBlurView还是用GPUImage实现，都会有卡顿的问题，所以在追求流畅度的时候可以考虑用一下代码替换。

```objc
UIVisualEffectView *visualEffectView = [[UIVisualEffectView alloc] initWithEffect:[UIBlurEffect effectWithStyle:UIBlurEffectStyleLight]];
visualEffectView.frame = CGRectMake(0, _mapView.frame.size.height - self.dataBed.frame.size.height, self.dataBed.frame.size.width, self.dataBed.frame.size.height);
visualEffectView.alpha = 0.92;
[_mapView addSubview:visualEffectView];

//注释掉的为旧代码
// if (nil == self.blurView) {
// self.blurView = [[FXBlurView alloc]initWithFrame:CGRectMake(0, _mapView.frame.size.height - self.dataBed.frame.size.height, self.dataBed.frame.size.width, self.dataBed.frame.size.height)];
// [self.blurView setDynamic:YES];
// self.blurView.blurEnabled = YES;
// self.blurView.blurRadius = 5.0;
// self.blurView.tintColor = [UIColor clearColor];
// [_mapView addSubview:self.blurView];
// UIView *grayLine = [[UIView alloc]initWithFrame:CGRectMake(0, 0, self.blurView.frame.size.width, 1)];
// grayLine.backgroundColor = RUNMOVE_DARK_GRAY ;
// [self.blurView addSubview:grayLine];
//
// }

```

##NSTimer结束一定要释放
因为NSTImer的创建会让你的ViewController计数加1，那么你的VC就不会释放，所以记得创建就一定要结束。

```objc
//创建
heartrateTime = [NSTimer scheduledTimerWithTimeInterval:HEARTRATE_TIME_INTERVAL target:self selector:@selector(insertHeartrate) userInfo:nil repeats:YES];

//释放
if (heartrateTime) {
[heartrateTime invalidate];
}
heartrateTime = nil;

```

##CocoaPods的问题

CocoaPods的问题颇多，竟然会遇到Apple Mach-O Linker Error相关的问题，多看看错误日志进行相应的解决，大多数都是找不到目录，找不到头文件的原因，也有找不到控件.o文件的时候（原因为模拟器是i386架构，所以相应Pods的Building Setting中设置）。
在一切办法都无法解决时候可以试试把Pods文件（除了podfile）都删了，重新进行pod install。

##pod install的问题

pod install后，编译出一堆问题，再删再装还是有问题，无法解决。可能是因为CocoaPods的控件缓存造成。

```
rm -rf ~/Library/Caches/CocoaPods
pod install
```

就ok了

##用宏提速NSCoding
http://stablekernel.com/blog/speeding-up-nscoding-with-macros/

