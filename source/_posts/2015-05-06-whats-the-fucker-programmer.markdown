---
layout: post
title: "程序员何苦为难程序员系列"
date: 2015-05-06 15:56:08 +0800
comments: true
categories: 
---
##呵呵
最近都在做UI，初学IOS那些年系列呵呵的进展很慢，然后又挖了一个坑来作死，充分发扬了一贯不负责任的传统。

##为啥呵呵
最近做的一个功能是把轨迹在服务端生成地图轨迹图片，然后存在小米的CDN上。然后功能都做好了，发现IOS客户端上图片无法显示，使用的SDWebImage，代码如下。

```objc
[cell.leftTrackImageView sd_setImageWithURL:[NSURL URLWithString:leftUrl]];
```

报错信息如下
```
Error Domain=NSURLErrorDomain Code=406 "The operation couldn’t be completed. (NSURLErrorDomain error 406.)"
```

##如何不呵呵
用charles拦截请求,可以看到请求头

```
Host: cdn.fds.api.xiaomi.com
Accept-Encoding: gzip, deflate
Accept: image/webp,image/*;q=0.8
Accept-Language: zh-cn
Connection: keep-alive
User-Agent: Runmove/110 CFNetwork/711.2.23 Darwin/14.0.0
```

然后用python的requests库来请求，发现可以正常获得，那看看python请求和相应的headers把。
请求如下：

```json request.headers
 {'Connection': 'keep-alive', 'Accept-Encoding': 'gzip, deflate', 'Accept': '*/*', 'User-Agent': 'python-requests/2.6.0 CPython/2.7.6 Darwin/14.3.0'}
```

响应如下：

```json response.headers
{'content-length': '68236', 'access-control-allow-headers': 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization,Content-MD5', 'proxy-connection': 'Keep-alive', 'access-control-max-age': '1728000', 'cache-control': 'max-age=86400', 'x-via': '1.1 zw11:8080 (Cdn Cache Server V2.0), 1.1 xiazai199:4 (Cdn Cache Server V2.0)', 'server': 'Tengine', 'content-md5': 'b4f3f2a873822d8f56ef928ef0587b8b', 'last-modified': 'Wed, 06 May 2015 03:20:02 GMT', 'access-control-allow-credentials': 'true', 'date': 'Wed, 06 May 2015 11:55:34 GMT', 'access-control-allow-methods': 'GET, POST, PUT, HEAD, DELETE, OPTIONS', 'content-type': 'image/jpeg'}
```

发现客户端请求的Accept包含上面响应头的content-type（image/webp,image/*;q=0.8和image/jpeg）。而上面请求的请求头的Accept为*/*，这为默认的值，也是范围最广的。那么我在Python把请求头修改下试试。

##偷天换日

```python
headers = {'Accept': 'image/webp,image/*;q=0.8'}
r = requests.get('http://cdn.fds.api.xiaomi.com/ir-heatmap/pathmap/colorline/8122/53d501c213ba564c9958ead983d1b9c014ae2630-s788_640.jpg',headers=headers)
#r:<Response [406]>

```

呵呵重现了，嘿嘿。果然是小米那帮坑货，对请求头做了限制。

##如何不呵呵
既然发现了原理，那么怎么可以不呵呵呢？
首先考虑，如果不设置，Accept应该是\*/\*的，肯定是被修改了，那么来看看SDWebImage的源码把。

```objc SDWebImageDownloader.m
_HTTPHeaders = [NSMutableDictionary dictionaryWithObject:@"image/webp,image/*;q=0.8" 
                                                  forKey:@"Accept"];
```

原来是SDWebImage内部修改了请求头，那么我们把它修改下就好了，SDWebImageDownloader是个单例，修改一次就全局修改了。

```objc
    [SDWebImageDownloader.sharedDownloader setValue:@"text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8"
                                 forHTTPHeaderField:@"Accept"];

```
问题解决。

##呵呵总结
程序员何苦为难程序员啊，挖了这么一个坑，谢谢你祖宗啊。
