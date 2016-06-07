---
layout: post
title: '初学IOS那些年踩过的坑（六）'
date: 2015-04-20 08:54
comments: true
categories: 
---
0、实现类似Eclipse的cmd+shift+r打开指定函数
一直以为没这功能，今天算发现了。在Xcode中可以用control+6打开所有函数的导航，然后可以进行输入检索，也可以按control+a显示所有。

1、用IOS自带NSJSONSerialization 去把Foundation对象转成JSON串在NSNumber转换精度上存在问题
```objc
NSMutableArray *pointArray = [[NSMutableArray alloc] init];
for (RunningPoint *point in pointEnityArray) {
[pointArray addObject:@{
@"latitude" : point.latitude,
@"longitude" : point.longitude,
@"chinaLatitude" : point.chinaLatitude,
@"chinaLongitude" : point.chinaLongitude,
@"isCorrect" : point.isCorrect,
@"distance" : point.distance,
@"altitude" : point.altitude,
@"speed" : point.speed,
@"mark" : point.mark,
@"pause_mark" : point.pauseMark,
@"britishMark" : point.britishMark,
@"running_time" : point.runningTime,
@"step_count" : point.stepCount,
@"collect_time" : [point.collectTime toString],
@"step_count" : point.stepCount,
@"running_time": point.runningTime,
}];
}
running.pointsData = [NSKeyedArchiver archivedDataWithRootObject:pointArray];
```
改为
```objc
for (RunningPoint *point in pointEnityArray) {
[pointArray addObject:@{
@"latitude" : [NSString stringWithFormat:@"%f",point.latitude.doubleValue],
@"longitude" : [NSString stringWithFormat:@"%f",point.longitude.doubleValue],
@"chinaLatitude" : [NSString stringWithFormat:@"%f",point.chinaLatitude.doubleValue],
@"chinaLongitude" : [NSString stringWithFormat:@"%f",point.chinaLongitude.doubleValue],
@"isCorrect" : point.isCorrect,
@"distance" : point.distance,
@"altitude" : point.altitude,
@"speed" : point.speed,
@"mark" : point.mark,
@"pause_mark" : point.pauseMark,
@"britishMark" : point.britishMark,
@"running_time" : point.runningTime,
@"step_count" : point.stepCount,
@"collect_time" : [point.collectTime toString],
@"step_count" : point.stepCount,
@"running_time": point.runningTime,
}];
}
```

2、table header and footer的高度问题
问题描述：这几天在重构一个页面，那个页面有一个tableview，因为重构把它从漂浮出来改成了加在底部。
结果在底部的tableview最地上出现了一个空行，因为没遇到过这种问题，也不知道那个界面是什么引起的。
问题排查：经过有经验的同事提示，知道了这类问题的排查方式。就是用模拟器运行，然后如下图打开。
![](http://chuantu.biz/t2/3/1429520293x-954497745.jpg)
然后在界面上找到不知来历的界面，就会看到提示这是啥。
![](http://chuantu.biz/t2/3/1429520387x1822611348.jpg)
原来是下面这个tableView的HeaderFooterView啊。那么怎么去掉他呢
要排查两个方面，一是代码中
```objc
- (CGFloat)tableView:(UITableView *)tableView heightForHeaderInSection:(NSInteger)section {

return 50.0f;
}
```
而是nib文件中设置，如下图。这样这个问题应该就解决了。
![](http://chuantu.biz/t2/3/1429520410x1822611348.jpg)

3、用了autolayout的界面中，要怎么实现位移动画呢？

```objc
//x,y,z动画要偏移的量
CATransform3D transform = CATransform3DMakeTranslation(x, y, z);
CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"transform"];
//view为要进行动画的view
animation.fromValue = [view.layer.presentationLayer valueForKey:animation.keyPath];
animation.toValue = [NSValue valueWithCATransform3D:transform];
//opacityAnimation.timingFunction 是用来控制动画线性发展。
//其中[CAMediaTimingFunction functionWithControlPoints:1.0 :0.0 :1.0 :0.1] 是一个贝塞尔曲线的控制方法。
//这也可以令动画做到先慢后快或先快后慢的结果。
//效果参考http://netcetera.org/camtf-playground.html
animation.timingFunction = [CAMediaTimingFunction functionWithControlPoints:0 :size :0 :size*2];
animation.duration = 0.6;
view.layer.transform = transform;
[view.layer addAnimation:animation forKey:animation.keyPath];
```

除此之外，用这种方式还可以做移动中大小变化
```objc
float bigInfoViewScale = scale*0.6 + 0.4;
float bigInfoViewY = -GET(200) + GET(200) * scale;
CATransform3D scaleTransform = CATransform3DMakeScale(bigInfoViewScale, bigInfoViewScale, 1);
CATransform3D transform = CATransform3DTranslate(scaleTransform, 0, bigInfoViewY, 0);

CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"transform"];
animation.fromValue = [self.bigInfoView.layer.presentationLayer valueForKey:animation.keyPath];
animation.toValue = [NSValue valueWithCATransform3D:transform];
animation.timingFunction = [CAMediaTimingFunction functionWithControlPoints:0.3 :0.5 :0.9 :1.275];
animation.duration = duration;

self.bigInfoView.layer.transform = transform;
[self.bigInfoView.layer addAnimation:animation forKey:animation.keyPath];
```
4、ViewController中，当一个界面只有一个scrollView时候，scrollView会自动偏移一个状态栏的高度
在ViewController的ViewDidLoad中加入下面代码可解决该问题。
```objc
self.automaticallyAdjustsScrollViewInsets = NO;
```
5、IOS中做类似GIF的动画
其实只要为每一帧切一张图，然后用如下代码就Ok了。
```objc
NSArray *images = @[[UIImage imageNamed:@"step1"], [UIImage imageNamed:@"step2"], [UIImage imageNamed:@"step3"]];
UIImage *animateImage = [UIImage animatedImageWithImages:images duration:1.2];
UIImageView *imageView = [[UIImageView alloc] initWithImage:animateImage];
// 默认是0 0代表无限循环
imageView.animationRepeatCount = 0;
[imageView startAnimating];
```

6、主界面使用scrollView后，nib文件有警告
has ambigous scrollable content height
原因：scrollView需要确定的contentSize，而subViews并没有足够的约束可以确定contentSize
解决：在scrollView中套一层view，设置如下图
![](http://chuantu.biz/t2/3/1429520432x1822611348.jpg)
















