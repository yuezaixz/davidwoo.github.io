---
layout: post
title: '<2014>---Scrum开发微博的准备'
date: 2014-01-02 13:46
comments: true
categories: [javascript, express, scrum, 感悟]
---
2014来了，就说说2014的第一个工作日的感悟吧，自己都有点受不了了，今天的感悟也稍微多了一点，但是这一天确实是充满了感悟和感动。
##生命之脆弱
![2.jpg](http://user-image.logdown.io/user/3769/blog/3827/post/171771/jbNHulbNSWSeD17Ithd8_2.jpg)
刚到公司打开邮件，就看到邮箱里面躺着数封邮件，这一封立马就让我震惊了。XX父亲心脏不好我很早就知道的，没想到这么重。而且我很难想象，当大家躲在喜迎新年的时候，他在想着什么。
人的生命真是脆弱，大自然的很多事情，对于人来说都是无法抵抗的。
##生命之离别
![1.jpg](http://user-image.logdown.io/user/3769/blog/3827/post/171771/ze9tks5RdSSn3tiy0BbQ_1.jpg)
谁能在床前，陪你走完这一生。相伴百年，也终须一别。人生是一辆巴士，路上总有人上来，有人下去，有的人微笑的来了，有的人却挥手告别，但是一定会有那么一个人，陪你走到旅途的终点。这段旅程虽然短暂，有你陪伴也无悔这一生。
##生命之感动
![3.jpg](http://user-image.logdown.io/user/3769/blog/3827/post/171771/858XcqYQRGihJPTIFNES_3.jpg)
早上去上班的时候，我在桌子上留了张纸条，给女朋友说早餐做好放哪了。晚上回来就看到女朋友同样留了张纸条。心里的暖又岂是保温杯里的热水能比的呢？

----------------------------分割线------------------------------------
#1 实战项目概述
其实就是完成一个简单的例子：微博，并且准备结合敏捷的开发，用用户故事来描述该项目的需求、sprint等来完成该任务。
#2 敏捷开发的准备
##2.1 用户故事
故事1：一个普通的网民，听朋友说起David’s Microblog，就想去看一看，因此用户在未登录的情况下，应该也能看到该微博网站最新的一些微博信息；
故事2：该网民觉得该微博不错，就打算注册一个私人账号，因此需要提供用户注册功能，需要具有用户名唯一性判断和密码验证功能，密码要加密而不是存储明文，并且跳转到首页；
故事3：该网民再次访问该微博的时候，想再次使用上次注册的账号，因此需要提供用户登录功能，需要验证密码；
故事4：登录后，该网民就成为了David’s Microblog的用户了，这时候他可以发布自己的微博信息，因此需要提供微博发布功能；
故事5：由于首页信息过于混杂，该用户希望能点击某一用户名，然后查看该用户的所有发布微博；
故事6：该用户使用的是公共机器，所以使用完微博后，想退出微博已防止他人盗用，因此需要退出功能；
故事7：喜欢上社区网站的用户，一般都是喜欢酷炫的人，因此微博的界面风格最好能比较流行，像twitter那样。
##2.2 任务分解
当产品经理把用户故事准备好后，ScrumMaster这时候就该组织sprint启动会议，然后对productbacklog中的用户故事，拿一些出来进行任务分解，然后根据sprint的周期所能完成的工作量来选择优先级高的、已经确定的需求。
比如一个sprint为2周，一共有5个人，那就有50个人日，但是要考虑到风险。暂时我们团队的风险指数一般都按1.5来算，就是50要除以1.5，大概33个人日。
然后就是进行任务分解，比如刚刚第一个用户故事的首页访问功能进行任务分解，时间是我乱写的，只是强调任务尽量以4h、8h为基准来划分：

1. 后台查询功能 4h
2. 页面设计与实现 4h
3. 路由规则、权限控制 4h

##2.3 之后的阶段
因为没有真实团队，之后的启动sprint、每日站会、设计讨论会、结对编程、总结会、代码评审、验收会等都无法用该例子说明，就稍微带过吧
#3 设计
该实战项目虽然只是完成一些简单的基础功能，但是还是要稍微设计下。比如路由规则、比如页面风格、比如数据库及ORM等
##3.1 路由规则
根据功能设计，我们把路由按照以下方案规划。

* /：首页
* /u/[user]：用户的主页
* /post：发表信息
* /reg：用户注册
* /login：用户登录
* /logout：用户登出

而且由于功能的增多，尽量让规则存在routes中的文件中。所以app.js需要改造下。

```

//新的代码
routes(app);

//注释掉旧的方法，这些东西都被搬到routes/index.js中
//app.get('/', routes.index);
//app.get('/hello', routes.hello);
//app.all('/user/:username', function(req, res,next) {
//console.log('all methods captured');
//next();
//});

```

Index.js中添加规则，但是可以先把实现放空
```javascript routes/index.js
exports.index = function(req, res) {
res.render('index', { title: 'Express' });
};
exports.user = function(req, res) {
};
exports.post = function(req, res) {
};
exports.reg = function(req, res) {
};
exports.doReg = function(req, res) {
};
exports.login = function(req, res) {
};
exports.doLogin = function(req, res) {
};
exports.logout = function(req, res) {
};
```
##3.2 界面设计
这里直接使用Twitter Bootstrap，谁让我们要善哉Twitter。可以登录[这个网站](http://twitter.github.io/bootstrap/)进行下载，但是我记得下载那一步好像是要翻墙的，自己想办法吧^.^，朋友，祝你好运。
将以下文件放到工程中相应的位置：
bootstrap-responsive.min.css
bootstrap.min.css
glyphicons-halflings-white.png
glyphicons-halflings.png
bootstrap.min.js

然后修改layout.js，实现一个拥有顶部工具栏、内容、页脚三大块的简单的Bootstrap风格的首页。
##3.3 数据库MongoDB
MongoDb是一种NOSQL，不是不要sql，而是not only sql。适合使用在查询较多，事务性不是很强大数据库操作中。
MongoDb使用BSON格式文件的形式存储数据的。
具体怎么安装参考 [centos6.3安装使用MongoDb](http://www.cnblogs.com/zhoulf/archive/2013/01/31/2887439.html)。
具体说明是NoSql，什么是MongoDb，什么是BSON，请自己百度谷歌洗刷刷哈。
项目的依赖需要加入MongoDb的依赖，另外，需要会话支持，所以也需要自己添加connect-mongo，具体代码如下：
```json package.json
{                                                                               
  "name": "david's microblog",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "3.4.7",
    "ejs": "*",
    "connect-mongo": ">= 0.1.7",
    "mongodb":">= 0.9.9"
  }
}
```
然后运行 npm install 更新依赖的模块。接下来在工程的目录中创建 settings.js 文件，这个文件用于保存数据库的连接信息。我们将用到的数据库命名为 microblog，数据库服务器在本地，因此Settings.js文件的内容如下：
```javascript setting.js
module.exports = {                                                              
  cookieSecret: 'microblogDavid',
  db: 'microblog',
  host: 'localhost',
};
```
接下来就是创建models/db.js,该模块输出即为数据库连接。
```javascript models/db.js
var settings = require('../settings');                                          
var Db = require('mongodb').Db;
var Connection = require('mongodb').Connection;
var Server = require('mongodb').Server;

module.exports = new Db(settings.db, new Server(settings.host, Connection.DEFAULT_PORT, {}));
```

最后再app.js中要添加以下代码
```javascript app.js
var express = require('express');
……
var partials = require('express-partials');
var MongoStore = require("connect-mongo")(express);
var settings = require('./settings');
var app = express();
……
```
##3.4 Session支持
之前已经说过为了会话支持，因此加入了connect-mongo插件，除了这个插件外，还需要在app.js中配置一个属性。
因为http协议不想tcp协议那样是存在会话的，http是无状态的，想要实现会话就必须借助Cookie，在Cookie中存储一些用户的信息。每次连接时候发送用户唯一标示，就能完成权限控制。

```javascript app.js片段
……
app.use(express.cookieParser());
app.use(express.session({
secret: settings.cookieSecret,
Store: new MongoStore({
db: settings.db
})
……
}));
```

##3.5 Flash支持
Express3.x开始已经不自带flash插件，该插件的主要功能就是服务器端返回客户端信息的时候，有效性只是一次获取，第二次获取时候既不存在了。
要使用该插件就要package.json中添加该插件的引用，然后npm install，最后在app.js中session片段之前添加以下代码:
```javascript app.js片段
var flash = require('connect-flash');
//app.configure(function(){                                                     
app.use(flash());
//});

app.use(express.cookieParser());
app.use(express.session({
secret: settings.cookieSecret,
Store: new MongoStore({
db: settings.db
})
}));
```

##3.6 静态helper
记得上一章说过的静态helper吗，该实战项目中也需要几个静态helper，代码如下：
```javascript app.js片段
app.use(function(req,res,next){
var err = req.flash('error');
var succ = req.flash('success');
//因为只能去一次，所以先取出来

res.locals.headers= req.headers;
res.locals.dyvar = "This is a dynamic var!";
//存放user在会话中
res.locals.user = req.session.user;
//错误提示信息
res.locals.error = err.length? err:null;
//成功提示信息
res.locals.success = succ.length?succ:null;
next();
});
```
#4 说明
到这里准备工作都做完了。下一篇将开始写具体的实现代码了。
另外说明下，本例子来自于 **BYVoid所著《Node.js开发指南》** 。实现是按原著实现，但是一些设计和思路我会用我的想法来做。
由于作者用的是express2.x版本，所以和我的实现是有出入的。我也会强调这些不一样的地方。如果in用的是express3.x的版本，请按我的方法配置，如果不是，希望你能参考原著，因为我没用2.x，所以我不会没有根据的贴出2.x的代码来。


