---
layout: post
title: '初学IOS那些年踩过的坑(三)'
date: 2015-02-05 04:55
comments: true
categories: 
---
#我是主题
今天这章就是一些小问题的汇总，木有主题
#我是内容
##无符号整形的负溢出负溢出
要小心无符号整形的负溢出负溢出，比如下面代码，当photos数组的数量为1时，第三行就负溢出了。要使用 [array count]的时候要注意。
```
        NSString *leftPhotoName = [_photos count] > indexPath.row ?[_photos objectAtIndex:indexPath.row]:nil;
        NSString *middlePhotoName = [_photos count]-1) > indexPath.row ?[_photos objectAtIndex:(indexPath.row+1)]:nil;
        NSString *rightPhotoName = ([_photos count]-2) > indexPath.row ?[_photos objectAtIndex:(indexPath.row+2)]:nil;
```
##NIB与手写代码不一致的一些坑
###UIView的交互开关
用Nib创建的View默认是不进行交互的，所以要勾上User Interaction Enabled。
否则将不能接受如下代码的手势的事件
```objc
- (void)initPhotoImageViewAction:(UIImageView \*) imageView
{
    UITapGestureRecognizer \*tapRecognizer = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(scrollerTapped:)];
    
    [imageView addGestureRecognizer:tapRecognizer];
}

- (void)scrollerTapped:(UITapGestureRecognizer*)gesture
{
    NSLog(@"%@",gesture.view);
    NSLog(@"11");
}
```
###UIActivityIndicator的动画
nib中创建的会自动旋转，而手写代码要记得[indicator startAnimation]，否则不显示。

##UIViewController的绘制
问题是这样的，UIViewController的self.view只有一个subview，而subview为UIImageView，开始在initWithNibName中初始化UIImageView的image，UIImageView的初始化不成功，第一次进入时候图片无法展现。
之后迁移到viewDidLoad中，进行初始化，就没问题了，所以要注意，UIVIewCOntroller的生命周期，initWithNibName并未完成初始化。

##UIImageVIew的autolayout和Aspect Fit
autolayout和Aspect Fit不冲突，因此，UIImageVIew需要增加完全的约束，而Aspect Fit会根据计算的大小进行Aspect Fit。

##用@class避免循环引用
A引用B，B引用A则为造成引用循环，这时候可以用@class来解决。此外，还可以提高效率，比如引用链为A->B,B->C,C->D如果都用#import，则中间有一个修改都会造成全部的编译，效率较低

##UIVIewController对于布局的生命周期

[[UIVIewController alloc] init]时候，不会UpdateContrainers，不会进行AutoLayout，不会触发viewDidAppear等方法，所以在[[UIVIewController alloc] init]后就对视图进行和布局有关的操作都是无效的，在viewDidLoad中、[[UIVIewController alloc] init]后只能进行数据的初始化操作。

```
    [runPhotoViewController index:gesture.view.tag];
    [self presentViewController:runPhotoViewController animated:YES completion:^{}];
``` 
要改为
```
    [self presentViewController:runPhotoViewController animated:YES completion:^{
        [runPhotoViewController index:gesture.view.tag];
    }];
```


```
- (void)viewDidLoad {
    //去掉
    //[self setScrollViewContentSize];
}
-(void)viewDidAppear:(BOOL)animated
{
    if(!isInitScale)//防止多次调用
    {
        [self setScrollViewContentSize];
        isInitScale = YES;
    }
}

```
##真机调试证书问题
coding signing indntity问题，需要更新证书
![](http://chuantu.biz/t/65/1423112120x1822611171.png)
![](http://chuantu.biz/t/65/1423112614x1822611171.png)
![](http://chuantu.biz/t/65/1423112647x-954497605.png)

##building时候报错
报错说XXX文件找不到，考虑可能是更新代码后遗留库影响，product中的clean执行一下
![](http://chuantu.biz/t/65/1423112680x-954497605.png)











