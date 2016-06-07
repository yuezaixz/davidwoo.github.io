---
layout: post
title: 'IOS从头开始之GCD基本方法介绍'
date: 2014-12-15 09:04
comments: true
categories: [iOS, GCD, obj-c]
---
#概述
任务支持函数和块，GCD比NSOperation更底层，可以是并发的或串行的。他是纯C语言编写，性能很好，而是是对象化的，内存也是通过计数管理。
#队列类型

* 主线程队列：提交到该队列的任务会在主线程中执行。可以调用dispatch_get_main_queue()来获得。因为main queue是与主线程相关的，所以这是一个串行队列
* 全局队列：是并发队列，并由整个进程共享。存在4个全局队列：高（DISPATCH_QUEUE_PRIORITY_HIGH ）、中（默认，DISPATCH_QUEUE_PRIORITY_DEFAULT ）、低（DISPATCH_QUEUE_PRIORITY_LOW ）、后台（DISPATCH_QUEUE_PRIORITY_BACKGROUND ）四个优先级队列。可以调用dispatch_get_global_queue函数传入优先级来访问队列。
```objc
dispatch_queue_t concurrentQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
```
* 自创建队列：该队列是用函数 dispatch_queue_create 创建的队列，提交到队列中的任务是串行的，各个队列间是串行的。dispatch_queue_create 函数的第一个参数是一个标签，是提供唯一标示方便调试。官方建议我倒置包名来命名队列，比如“com.dragonsoft.subsystem.task”。这些名字会在崩溃日志中被显示出来，也可以被调试器调用，这在调试中会很有用。第二个参数是queue的类型，设为NULL时默认是DISPATCH_QUEUE_SERIAL，将创建串行队列，在必要情况下，你可以将其设置为DISPATCH_QUEUE_CONCURRENT来创建自定义并行队列。
```objc
dispatch_queue_t firstSerialQueue = dispatch_queue_create("com.pixolity.GCD.serialQueue1", 0);
```
注意：全局的队列，不用retain或release，自创建通过ARC进行，也不需要，如果想保留，就需要。

#dispatch_async
将任务推到子线程中异步执行，注册的任务完成时间是随机不确定的。该函数会立即返回，任务会在后台异步执行。
该方法使用场景按我理解类似做Web前端开发时候，可能会有一个操作需要去后台请求数据，然后再对主页面进行局部刷新。在Web开发可以利用Ajax和Javascript的事件循环异步机制来轻松实现回调而在IOS程序中，这就意味着需要将数据的获取交给子线程去做，数据返回后再把任务推回到主线程来执行一些代码。使用GCD的dispatch_async可以简单地完成这个任务。使用嵌套的dispatch，在外层中执行后台任务，在内层中将任务dispatch到main queue，以下代码为网上拔来的示例。
```objc
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSURL * url = [NSURL URLWithString:@"http://avatar.csdn.net/2/C/D/1_totogo2010.jpg"];
        NSData * data = [[NSData alloc]initWithContentsOfURL:url];
        UIImage *image = [[UIImage alloc]initWithData:data];
        if (data != nil) {
            dispatch_async(dispatch_get_main_queue(), ^{
                self.imageView.image = image;
             });
        }
    });
```
还有一个场景他的应用会比较有效，比如要计算1000以内的素数，那就要循环1000次，每次都是同步执行的，性能很差。可以应用如下代码，doSomethingIntensiveWith:obj改为判断当前循环到的数字是否为素数，是就添加到一个集合中。
```objc
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
for(id obj in array)
    dispatch_async(queue,^{[self doSomethingIntensiveWith:obj];});
```
#dispatch_sync
该函数干的事和dispatch_async相同，差异就是它不会立即返回，它会等待传入的block中的代码执行完成并返回。结合\_\_block类型修饰符，可以用来从执行中的block获取一个值。例如，你可能有一段代码在后台执行，而它需要从界面控制层获取一个值。那么你可以使用dispatch\_sync简单办到。
```objc
__block NSString *stringValue;
dispatch_sync(dispatch_get_main_queue(), ^{
        stringValue = [[textField stringValue] copy];
});
[stringValue autorelease];
// 操作stringValue
```
有一个使用的例子就是替换锁。以下代码为something属性的getter&setter，具体可以看程序中的注释。
```objc
    - (id)something
    {
        __block id localSomething;
        //会等待传入的block执行完后才返回，但是由于自定义队列是FIFO执行的，所以如果前面有多个setter操作，那么现在就在等待中。
        dispatch_sync(queue, ^{
            localSomething = [something retain];
        });
        return [localSomething autorelease];
    }

    - (void)setSomething:(id)newSomething
    {
          //直接返回，把任务添加到队列中执行。
        dispatch_async(queue, ^{
            if(newSomething != something)
            {
                [something release];
                something = [newSomething retain];
               //修改缓存可能非常耗时间，所以立即返回完全不影响性能
                [self updateSomethingCaches];
            }
        });
    }
```
不过这里要注意，小心有代码会造成互斥锁，导致getter中的dispatch_sync一直无法返回。
#dispatch_group_async
它可以实现监听一组任务是否完成，完成后得到通知执行其他的操作。
可以想象这么一个场景，比如你执行10下载任务，然后程序将每个下载任务又分成了2步操作，第二步合并必须在第一步完成后才能进行，那么前面dispatch_async的那个循环的例子就不管用了。看以下代码
```objc
  dispatch_queue_t queue = dispatch_get_global_qeueue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
  dispatch_group_t group = dispatch_group_create();
  for(id obj in array){
    dispatch_group_async(group,queue, ^{
      [self doDownloadIntensiveWith:obj];
    });
     //group中的任务都结束了，就会去通知该block中的任务去执行
    dispatch_group_notify(group,queue, ^{
      [self doDownloadCombaineWith:obj];
    });
    //IOS 6.0后不需要自己去释放，函数的变量栈在函数退出时候会自己释放。这个释放不同于ARC，他是在运行时
    dispatch_release(group);
   }

```
#dispatch_barrier_async
在前面的任务执行结束后它才执行，而且它后面的任务等它执行完成之后才会执行
还是前面那个场景，比如下载必须等前面的任务完成后才进行合并操作，合并结束后才进行提示操作，那么dispatch_group_async又不是很好用了，看以下代码

```objc
    dispatch_queue_t queue = dispatch_queue_create("com.dragonsoft.queue1", DISPATCH_QUEUE_CONCURRENT);
    for(id obj in array){
    dispatch_async(queue, ^{
        [self doDownloadIntensiveWith:obj];
    });
    dispatch_async(queue, ^{
        [self doDownload2IntensiveWith:obj];
    });
    dispatch_barrier_async(queue, ^{
        [self doDownloadCombaineWith:obj];
    });
    dispatch_async(queue, ^{
        [self doNoticeWith:obj];
    });
     }
    
```
#dispatch_apply
执行某个代码片段N
```objc
dispatch_apply(5, globalQ, ^(size_t index) {
    // 执行5次
});

```
使用场景还是刚刚那个下载，这下连循环都省了
```objc
  dispatch_queue_t queue = dispatch_get_global_qeueue(DISPATCH_QUEUE_PRIORITY_DEFAULT,  0);
  //推到后台去
  dispatch_async(queue,^{
    dispatch_apply([array count], queue, ^(size_t index){
      [self doDownloadIntensiveWith:obj];
    });
  });
```
