---
layout: post
title: '初学IOS那些年踩过的坑(五)'
date: 2015-03-07 06:33
comments: true
categories: 
---
#集成友盟后报错
LOG：
15-02-12 09:26:47.206 Runmove[2532:13249] NSScanner: nil string argument
2015-02-12 09:26:47.206 Runmove[2532:13249] Umeng: Read binary image info failed!

其实不是友盟问题，是自己程序中有了个错误，只是打开了友盟的调试开关，所以报了友盟错误。
```objc
//    [MobClick setLogEnabled:YES]; 
```
注释该行代码后可以正常报告出程序中的异常

#tableview下方空白处的seprator line去除
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


#使用nib做界面，但是layoutAttribute有些又需要根据适配来适配，nib+手写混写好烦恼啊，感觉睡不着觉了
其实错了，混写其实很简单的，可以像拉button一样的拉你的约束.nslayoutattribute也是可以拉线的，这样一切都太平了

#画面不流畅可能是因为一些透明视图影响了fps，可以去掉这些透明视图
方法：Product->Profile->Core Animation->右下第二个选项卡中勾上Color Blended Layers
然后看到红色的就都是透明的视图，不需要透明效果的可以去掉透明

#UITableView拉倒底部最后行只显示一半问题
原因：计算UITableView高度的时候按整个屏幕来计算了，NavigationBar的高度被忽略。所以就少算了NavigationBar高度。

#UILongPressGestureRecognizer会触发两次，分别是开始和结束
所以如果只想触发一次，就需要加个判断了。
```objc
-(void)longPress:(UILongPressGestureRecognizer *)sender
{
    if (sender.state == UIGestureRecognizerStateBegan) {
        [self yourMethod];
    }
} 
```

#简单实现下拉到底刷新数据
```
-(void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView
{
    if([_photos count] >= LOAD_COUNT_DEFAULT*(_loadPage+1) && scrollView.contentOffset.y + (scrollView.frame.size.height) > scrollView.contentSize.height*0.8){
        [[RunService sharedInstance] loadPhotosWithPage:(_loadPage+1) count:LOAD_COUNT_DEFAULT complete:^(NSArray *photos) {
            _loadPage++;
            [_photos addObjectsFromArray:photos];
            [self loadPhoto];
        }                                         fail:^(NSString *msg) {
            
        }];
    }
}
```

#数据持久化到NSUserDefaults 
需要数据持久化的对象必须实现NSCoding
```objc
#import "NSObject+NSCoding.h"

@interface ServerPhoto : NSObject <NSCoding>
```
实现文件中还要重写2个方法
```objc
- (void)encodeWithCoder:(NSCoder *)coder {
    [self autoEncodeWithCoder:coder];
}

- (id)initWithCoder:(NSCoder *)coder {
    if (self = [super init]) {
        [self autoDecode:coder];
    }
    return self;
}
```
持久化
```objc
        NSUserDefaults *udf = [NSUserDefaults standardUserDefaults];
        NSData *photosData = [NSKeyedArchiver archivedDataWithRootObject:_photos];
        [udf setObject:photosData forKey:_dataCacheName];
        [udf synchronize];
```
反持久化
```objc
    NSUserDefaults *udf = [NSUserDefaults standardUserDefaults];
    NSData *data = [udf objectForKey:_dataCacheName];
    NSMutableArray *photoData = [NSKeyedUnarchiver unarchiveObjectWithData:data];
    if (photoData && photoData.count > 0) {
        _photos = photoData;
        [_photoTableView reloadData];
    }
    else {
        [SVProgressHUD showWithStatus:@"加载数据中"];
    }
```

10、SDWebImage
options用SDWebImageHighPriority，优先级高的话，会及时写入到磁盘
```objc
[leftImageView sd_setImageWithURL:[NSURL URLWithString:leftPhotoUrl] placeholderImage:[UIImage new] options:SDWebImageHighPriority];
```