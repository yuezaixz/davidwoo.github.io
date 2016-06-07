---
layout: post
title: "初学IOS那些年踩过的坑（九）"
date: 2015-06-29 11:25:10 +0800
comments: true
categories: 
---

##如何让小图片按钮的点击区域变大
之前经常做的事情就是，用个imageView来放图片，然后用一个更大透明UIButton盖住他，这样有个问题就是，一旦按钮需要有点击高亮效果，就会很麻烦的（各种点击和释放的回调，然后去修改imageView的图片）。

最近发现了个方法，可以用setContentEdgeInsets来设置Button内容的区域。注意，top、bottom等，都是与其距离，比如bottom，就是与底部距离。开始我以为是从(left,top)到(right,bottom)这两个点的范围做图片的位置，然后导致无法得到想要的效果。

```objc
//[self.runBtn setContentEdgeInsets:UIEdgeInsetsMake(<#CGFloat top#>, <#CGFloat left#>, <#CGFloat bottom#>, <#CGFloat right#>)];
[self.runBtn setContentEdgeInsets:UIEdgeInsetsMake(8, 18, 8, 18)];
```

##view没有设置backgroundColor，setNeedsDisplay后触发drawRect，会导致重复绘制

```objc
-(void)drawRect:(CGRect)rect{
//….

}

-(void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event{
//….
[self setNeedsDisplay];
}
```
这时候drawRect中的内容会重复绘制，只要设置backgroundColor就不会重复绘制了。
```objc
lineChartView.backgroundColor = [UIColor whiteColor];
```

##ore-Data升级后导致数据升级失败
用的是MagicRecord，AppDelegate代码中有初始化
```objc
[MagicalRecord setupCoreDataStackWithAutoMigratingSqliteStoreNamed:SQLITE_NAME];
[MagicalRecord shouldAutoCreateDefaultPersistentStoreCoordinator];
[MagicalRecord setShouldDeleteStoreOnModelMismatch:YES];
[MagicalRecord setupAutoMigratingCoreDataStack];//这里报错
```
报错内容为Can't find or automatically infer mapping model for migration。

在SOF上找到了下文，具体原因不明，可能是添加版本时候有错误操作，这时候只要把版本先退回到上一个版本，然后删除新的版本，提交代码（不先提交SVN可能会出问题哦）。然后重新添加一次再提交就OK了。
[参考](http://stackoverflow.com/questions/7708392/how-to-delete-an-old-unused-data-model-version-in-xcode-4)
