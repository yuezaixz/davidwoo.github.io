---
layout: post
title: 'IOS从头开始之Block语法'
date: 2014-12-13 02:41
comments: true
categories: [iOS, cocoa, block, obj-c, 代码块, 闭包]
---
#Block
我认为block（代码块）语法算是给Obj-c增加了很多优秀的特性，它类似于函数式编程中的闭包，通过他你可以像对待对象一样的对待函数，即函数成了面向对象中的头等公民，可以在方法和函数中定义及传递，这个是在Java中绝对无法想象的（在Java8之前必须在要传递的函数上套一层类，且无法在函数中定义）。
总之，通过这个特性，你可以创建比之前更灵活、可复用性更高的代码。
#特性
对javascript比较熟悉的人都知道，回调函数除了事件循环异步IO的好处外，还有一个就是可以通过传递回调函数实现很多优秀的特性，比如以下代码中通过传入回调函数实现了打印和过滤。

```javascript
//遍历并打印
["one","two","three"].forEach(function(element ,index) {
    console.log(index+":"+element);
});
//过滤数组中含有wu的
console.log(["david-wu","ying-tang","lily-wu"].filter(function(element) {
    return element.indexOf("wu") >= 0;
}));

```

那么，通过block就可以实现代码段的复用，而且这种复用很灵活，不像普通函数那样，在Java中一个函数中不确定的逻辑经常是使用钩子方法，在函数中把不确定的逻辑抽象在钩子方法中，通过之类实现来达到多态的效果，但是这样的限制导致的应用场景非常有限，如果可以直接将函数作为参数传递，既可以充分的将函数抽象后高度复用。
#声明与定义
如下代码，void (^firstBlock)(NSSring \*x)声明了变量，void为返回类型。^则声明这是一个block，把他当做定义指针时候的\*就好了。^为block的变量名，变量名和^一起用刮号刮起来。刮号后又是一个刮号，那个刮号中为参数，多个参数用“,”间隔。如果无参数则直接带一对刮号即可。注意，参数名字可以省略，但是书中建议最好不要这样，我也这么认为，因为这样很可能会造成迷惑，特别对于我这样的新手。
之后就是等号，等号后即为代码块的定义，也是以^开头，之后跟着刮号，刮号中为参数，参数如果为空可以省略该刮号及其内部内容，刮号之后就是花刮号包裹的代码块，可以在一行上定义，也可以跨多行，总之就和定义正常函数时候一样。

```objc

void (^firstBlock)(NSSring \*x) = ^(NSString \*x){
	NSLog(@"%@",x);
};

```

其他定义

```objc

//不带参数且一行内定义
void (^aVoidBlock)() = ^{NSLog(@"Hello Workld!")}
//直接内联的定义，传入其他函数
doIt(^(NSString \*x){NSLog(@"%@",x);})

```
#使用
首先，要了解如何声明一个使用代码块的函数。定义时注意不同于其他参数，他需要返回值，需要(^)声明他是变量，他需要参数列表，也可以把(NSComparisonResult (^)(NSString \*value))里的都当做是定义这个参数的类型。
```objc
-(void)userCodeBlock:(NSComparisonResult (^)(NSString \*value)) theBlock;
```
再来看看实现类中如何写
```objc
-(void)userCodeBlock:(NSComparisonResult (^)(NSString \*value)) theBlock{
	if(NSOrderedSame == theBlock(@"foo"))
  	doSomethingIfSame();
  else
  	doSomethingElse();
}
```
#作用域
这点我感觉和大多数函数式变成，如Javascript一样，block的作用域除了传入的参数外，还有他**定义时**作用域的变量栈的只读副本，即代码块拥有对所定义位置程序的完整状态的只读访问权限。
```objc
NSString \*formatStr = @"%s";
void (^firstBlock)(NSSring \*x) = ^(NSString \*x){NSLog(formatStr,x)};
doIt(firstBlock);
```
注意，如果想对变量进行读写，那么可以在声明变量时通过 \_\_block显示地将变量声明为可读写的。
**由于代码块具备了给应用状态拍快照的能力，并通过这种方式使其在应用的其他地方可用，这样就提供了一种可以封装和操作数据的及其强大的机制。**
#内存管理
其内存管理和其他对象类似，由于它是在栈上分配的空间，因此传入的代码块对象需要使用-copy而不是retain，如果需要保留它就必须在对上得到一个副本。

```objc
@interface Person:NSObject
{
	void (^myBlock)(NSString *);
}
  @property (readonly,copy) ((^)(NSString \*)) \*myBlock;

  -(void)doSomethingWithBlock;

@end

@implementation Person

-(void)doSomethingWithBlock{
  myBlock(@"foo");
}

@end

```
#类型定义
使用typedef关键字进行代码块定义，可以使代码块的定义更加易读。
```objc
typedef void (^BlockWithCHarArg)(char \*)
@interface Person:NSObject
{
	BlockWithCHarArg myBlock;
}
  @property (readonly,copy) BlockWithCHarArg myBlock;
  -(void)doSomethingWithBlock;
@end
```
#代码块的用处
##GCD中的应用
GCD是一个Obj-c的框架，他提供了易用的抽象层，这样开发者不需要处理底层的线程管理就可以充分利用多处理器和多核架构。
这样开发者只需要提供代码块，GCD可以提供同步和异步的执行方式。
```objc
dispatch\_async(dispatch\_get\_global\_queue(0, 0),^{doSomethingSlow();});
```
dispatch\_get\_global\_queue方法可以获得全局队列，第二个参数就是一个代码块，这样我们就不需要关心线程应该开多少，都可以交给GCD来完成，队列也可以通过dispatch\_queue\_create函数来创建一个私有的顺序执行的队列。如下代码
```objc
dispatch_queue_t myQueue = dispatch_queue_create("david.queue", DISPATCH_QUEUE_SERIAL);
dispatch_async(_captureQueue, ^{
	doSomething();
});
```
##过滤、映射、处理
如前面javascript的代码段的.filter函数对数组进行过滤，Obj-c也可以通过代码块来实现相应功能，这里就拿映射来举个例子。
```objc
NSArray \*map(NSArray \*items,id (^block)(id item)){
	NSMutableArray \*result = [NSMutableArray array];
  for(id item in items){
  	[result addObject:block(item)];
  }
  return result;
}
```
##标准API中的代码块
常见的类NsArray，NSDictionary、NSIndexSet和NSSet中，都用使用代码块的函数，NSString、NSAttributeString中也提供了使用代码块逐行逐个遍历的方法。

##其他
比如说闭包教程中最常举的例子--计数器，先来看看JavaScript中如何做。
```objc
function getCounter(){
    var counter = 0;
    return function(){
        return ++counter;
    };
}
counter = getCounter();
counter2 = getCounter();
console.log(counter());//1
console.log(counter2());//1
console.log(counter());//2
console.log(counter());//3
console.log(counter2());//2
```
正如前面所说，代码块（闭包）可以保存函数定义时候的状态，利用这种状态就实现了计数器的创建方法。那么Obj-c中能不能这样做呢？还是等我Mac机器弄起来后再试试吧，现在也只能敲敲代码。

还有一些作用比如CPU复杂性的运算，可以通过GCD来完成多线程处理，这个也等Mac搞定后动手试一试。

#不足
但是，Obj-c中的block不像函数式编程（JavaScript、Ruby、Python）中那么的灵活，但是至少是比Java灵活的多了。