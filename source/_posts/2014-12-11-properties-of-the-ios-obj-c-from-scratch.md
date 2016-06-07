---
layout: post
title: 'IOS从头开始之Obj-C的属性'
date: 2014-12-11 14:48
comments: true
categories: [属性, obj-c]
---
#学习开始
开始学习obj-c了，最近工作的事情也挺忙的，只能抓住一点点的休息时间来看书。Mac机器还没弄起来，就按看书的理解瞎写点文章，看看之后敲代码后会不会觉得之前理解的偏差太大了。
今天就先随便写一点对属性的理解吧。
#属性
##什么是属性
对象是一个类的实例，而封装在一个对象中的数据在Java中常称为数据、状态、特性、成员变量或者属性，而在Obj-c中属性有另外一番意义。
在Obj-c中属性主要用于声明类的数据成员的存取方法，有了他们就不必使用很多标准代码（如Java中就需要定义getter & setter）。
##如何声明
@property，在头文件中的数据前加这个标志，就可以让编译好器自动编写一个与数据成员同名的方法声明来省去读写方法的声明。
例如：
```objc
@property int count;
```
等同于
```objc
@property int count;
- (void)setCount:(int)inCount;
- (int)count; 
```
##参数
如下面代码，property的完整语法是：@property (参数1,参数2,...) 类型 属性1名,属性2名;
```objc
@property (nonatomic, weak) IBOutlet UIBarButtonItem \*clearButtonItem;
```
###nonatomic
指定生成的存取器函数是非原子性的，即线程安全的。默认为原子性。
书中是建议如果不需要多线程，不要使用原子性。我也这么认为，毕竟在取属性前都加一个锁的开销也是挺大的。
###readwrite
既生成setter和getter方法，默认为readwrite
###readonly
即只生成getter方法
###assign
既生成赋值函数只是简单的变量赋值，就当做java中的set方法好了，毕竟java中没有内存管理。看旧的书说默认为assign，但是看新的书说已经改成默认strong了。这点暂不确定，有疑问。
###retain
既生成的赋值函数在赋值到变量前，会对参数进行release旧值，再retain新值。这个开始我不是和理解，但结合后面学习的引用计数器就理解了，retain方法会给实例的引用计数器+1，而release会-1，那么set操作就需要把旧值的引用先去了，再将新值的引用+1.
但是，这个属性在ARC下编译就可以去掉了，ARC会在编译的时候在适当的地方加retain和release。
###copy
生成的存取函数会复制传入值到数据成员，初始计数为1.
对于NSString、NSArray等非可变类，书中是推荐用copy来声明的。
###strong
该属性值对应 __strong 关键字，即该属性所声明的变量将成为对象的持有者。

###weak
该属性对应 __weak 关键字，与 __weak 定义的变量一致，该属性所声明的变量将没有对象的所有权，并且当对象被破弃之后，对象将被自动赋值nil。

并且，delegate 和 Outlet 应该用 weak 属性来声明。同时，如上一回介绍的 iOS 5 之前的版本是没有 __weak 关键字的，所以 weak 属性是不能使用的。这种情况我们使用 unsafe_unretained。

#如何实现
关在同文件中声明可不行，还得在实现文件中对属性的处理方式进行声明。
#synthesize和dynamic
在实现文件中用前者就不用为实现文件中的属性写任何代码，而用后者编译器就会希望你为属性创建一对合适的getter & setter。
但是，在Obj-c2.0后，已经不需要使用@synthesize关键字了，书中是这么建议的**在以后的编程中都使用属性和自动合成的实例变量，在头文件中声明公用的属性，在实现文件的类扩展部分声明私有属性。**
```objc MyClass.h
@interface MyClass : NSObject
  @property (nonatomic,readwrite,week) id delegate;
  @property (nonatomic,readonly,strong) NSString \*readonlyString;
@end
```
```objc MyClass.m
@implementation MyClass ()
  @property (nonatomic,readwrite,strong)  NSString \*privateString;
  @property (nonatomic,readwrite,strong) NSString \*readonlyString;
@end
```
注意：**readonlyString在实现时候重定义使用了readwrite，那么他就有一个私用的setter方法。**
##隐藏的实例变量
如上代码会自动创建几个实例变量：\_readonlyString ,\_delegate,\privateString。最右在init、dealloc和覆盖的存取器中才应该使用这些实例变量。