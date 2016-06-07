---
layout: post
title: 'AFNetworking传输对象时，将参数的类型转换成str的坑'
date: 2015-01-08 02:47
comments: true
categories: [iOS, AFNetworking]
---
#背景
先描述下背景，现在要增加一个功能，用户在地图上触摸操作，移动或放大缩小了地图，这时候要根据新的region去服务器端拿范围的数据。
#问题
之前是用IOS自带的网络传输，数据的序列化方式是 '''NSData\* data = [NSJSONSerialization dataWithJSONObject:[location toDictionary] options:0 error:NULL];''' 后直接塞进去作为参数。
但是，后面把相应的网络功能全部替换，改成了AFNetWorking进行传输，他的POST函数接受的params为dictionary或者字符串。所以就直接把[location toDictionary]传了进去。
这之后就导致mongo查询db.locations.find({"location":{"$geoWithin":{"$box":[[0,0],[150,100]]}}}) 再查不到数据了，因为数据变了,下面第一条是正常数据，第二条是异常数据，coordinates里的值编程了字符串。
```json
{ "_id" : ObjectId("54acfef52390d9a68247df6b"), "details" : "amoy", "location" : { "type" : "Point", "coordinates" : [ 118.1189864873886, 24.48821130558777 ] }, "categories" : [ ], "placename" : "福建省厦门市思明区嘉莲街道怡祥大厦", "name" : "福建省厦门市思明区嘉莲街道怡祥大厦", "imageId" : "54abdd38f648dbff427cbf0b" }
{ "_id" : ObjectId("54acff2b4a07474a1d8467d4"), "categories" : [ "03" ], "details" : "fdfdfdfdfdf", "location" : { "coordinates" : [ "118.1164839863777", "24.48796232996047" ], "type" : "Point" }, "name" : "福建省厦门市思明区筼筜街道莲岳路173之10号", "placename" : "福建省厦门市思明区筼筜街道莲岳路173之10号", "created_at" : ISODate("2015-01-07T09:40:59.815Z") }
```
发现是字符串引擎的问题就找了很久。之后开始尝试解决。

#在APP端直接给AFNetworking传字符串。
如下代码，但是新问题就来了，服务器端接收的数据格式为{"(null)":"{\"details\":\"fffdfd\",\"categories\":[],\"name\":\"福建省厦门市思明区开元街道厦门歌仔戏研习中心\",\"location\":{\"type\":\"Point\",\"coordinates\":[118.0860650539398,24.45868229051323]},\"placename\":\"福建省厦门市思明区开元街道厦门歌仔戏研习中心\"}”}，多了一层。
```
    //IOS自己的JSON转换方式
    NSData* data = [NSJSONSerialization dataWithJSONObject:[location toDictionary] options:0 error:NULL];
    NSString *jsonString = [NSString stringWithUTF8String:[data bytes]];
    
    NSString* urlStr = isExistingLocation ?[locations stringByAppendingPathComponent:location._id]:locations;
    AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];
    [manager.requestSerializer setValue:@"application/json" forHTTPHeaderField:@"Accept"];
    [manager  POST:urlStr parameters:jsonString success:successCallback failure:^(AFHTTPRequestOperation *operation, NSError *error) {
            NSLog(@"%@",error);
     }];
```
#在服务器端操作
APP端不变，在服务器端接收时候做转换，int(inputStr)。好，问题就这么简单解决了。