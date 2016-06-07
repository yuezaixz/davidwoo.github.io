---
layout: post
title: "初学IOS那些年踩过的坑（七）"
date: 2015-05-07 14:21:16 +0800
comments: true
categories: 
---
##地图的View退到后台后导致闪退
错误信息为gpus_ReturnNotPermittedKillClient
###事因
情况大概是这样的，中间有个版本因为地图占用了内存，所以在4S上会有内存不足闪退的原因，然后做了一次修改就是进入后台的时候把地图销毁了，具体代码如下。

```objc
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(applicationDidEnterBackgroundNotification:) name:RMApplicationDidEnterBackgroundNotification object:nil];
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(applicationDidWillEnterForegroundNotification:) name:RMApplicationDidWillEnterForegroundNotification object:nil];
```

销毁代码如下

```objc RMMapView.m
- (void)applicationDidEnterBackgroundNotification:(NSNotification *)notification {
    if ([self viewController] == [self topViewController]) {
        [self setHiddenMap:YES];
    }
}

- (void)applicationDidWillEnterForegroundNotification:(NSNotification *)notification {
    if ([self viewController] == [self topViewController]) {
        [self setHiddenMap:NO];
    }
}

- (void)setHiddenMap:(BOOL)hiddenMap {
    if (hiddenMap == _hiddenMap) {
        return;
    }
    self.hidden = hiddenMap;
    _hiddenMap = hiddenMap;

    if (hiddenMap) {
        if (_mapViewClient) {
            _latitudeDelta = [_mapViewClient getMapRegionLatitudeDelta];
            _longitudeDelta = [_mapViewClient getMapRegionLatitudeDelta];
            _coordinate = [_mapViewClient getMapCenterLocationCoordinate2D];
            [_mapViewClient removeMapView];
            _changedMapLevel = NO;
            _mapViewClient = nil;
        }
    }
    else {
        if ([_mapViewClient getMapView] == nil) {
//            [self performSelector:@selector(createMap) withObject:nil afterDelay:0.5];
            _changedMapLevel = NO;
            [self createMap];
            [self drawLineForMapView];
        }
    }
}

```


```objc MapViewClient.m
- (void)removeMapView {
    if (nil != mainMapView) {
        [mainMapView removeFromSuperview];
    }
    mainMapView = nil;
}

```

然后就偶尔会出现退到后台会闪退的原因。Google了一些资料算是把自己误导了。
回答的意思基本上是销毁的方法不对（可能确实是销毁的有问题），但是回答也基本上都是设置成nil，要不就把delegate=nil，尝试了都没效果，还是一直会闪退。

###解决
最后换了个思路，放弃销毁，把销毁的代码全部去了，只是在地图的代理方法里加判断，判断是否为Active，不是则不执行。暂时测试还未发现闪退问题

```objc
    if ([UIApplication sharedApplication].applicationState != UIApplicationStateActive) {
        return;
    }
```