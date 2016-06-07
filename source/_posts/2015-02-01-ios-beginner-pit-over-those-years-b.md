---
layout: post
title: '初学IOS那些年踩过的坑(二)'
date: 2015-02-01 10:02
comments: true
categories: 
---
#结论：
在ViewController的自定义viewDidLayoutSubviews方法中，调用viewDidLayoutSubviews方法后进行了addSubview操作后，需要调用[self.view layoutSubviews]来触发子视图布局，否则否导致子视图不知道如何布局而应用崩溃。
#坑描述：
为了给IPHONE做适配，实现了ViewController的 viewDidLayoutSubviews方法，该方法是在试图控制器对试图进行auto layout的时候调用其来触发子视图的auto layout的。
```
- (void)viewDidLayoutSubviews {
    [super viewDidLayoutSubviews];
    NSString *deviceInfo = [CommonUtils getDeviceInfo];
    int mapHeight = ROUTE_MAP_SCREEN_MAP_HEIGHT_5;
    if ([deviceInfo isEqualToString:@"iPhone 4S"] || [deviceInfo isEqualToString:@"iPhone 4"]) {
        mapHeight = ROUTE_MAP_SCREEN_MAP_HEIGHT_4S;
    }

    self.downNavBar.frame = CGRectMake(0, mapHeight, self.view.bounds.size.width, 95);
    [self.view addSubview:self.downNavBar];

    [self.menuView setHidden:YES];
    [self.view addSubview:self.menuView];
    [self.view bringSubviewToFront:self.menuView];

    self.menuView.frame = CGRectMake(38, mapHeight - 110, 80, 110);
    [self.mapBed setFrame:CGRectMake(0, 0, self.view.bounds.size.width, mapHeight)];
}

```
但是，调用该视图的时候应用崩溃了，后台报错
```
'NSInternalInconsistencyException', reason: 'Auto Layout still required after sending -viewDidLayoutSubviews to the view controller. RouteMapViewController's implementation needs to send -layoutSubviews to the view to invoke auto layout.'

```
#坑解释
自动布局的方式为自上而下，也就是从父视图向子视图，调用了[super viewDidLayoutSubviews]就是为了触发子视图的布局，但是之后又增加了2个子视图，那么这两个子视图的布局就没有触发。所以，需要在该方法最后增加一行代码来触发子视图的布局。
```
- (void)viewDidLayoutSubviews {
    ...
    [self.mapBed setFrame:CGRectMake(0, 0, self.view.bounds.size.width, mapHeight)];
    [self.view layoutSubviews];
}

```