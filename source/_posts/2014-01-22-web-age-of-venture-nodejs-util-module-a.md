---
layout: post
title: '互联网时代如何创业----Node.js的Util模块（一）'
date: 2014-01-22 13:18
comments: true
categories: 
---
最近打车应用的斗争已经从群雄逐鹿，到现在快的和滴滴打车双雄争霸的市场格局了。但是，两家创业公司的斗争实际并没有那么简单，他们背后是“财付通VS支付宝”，“微信VS来往”，“理财通VS余额包”，最幕后的黑手就是“腾讯VS阿里巴巴”。
本文不想过多篇幅来写两家巨头是如何烧钱大战的，不过他们把补贴做到了一种极端上，靠着疯狂砸广告、大力补贴司机、变态似的补贴用户等行为，可以说把整个市场都占据了。打车软件作为最容易被市场接受的LBS服务之一，是巨头们测试其O2O能力和渗透率的绝佳工具，也为未来更多O2O业务提供接口，所以才这么的疯狂。
因此，对于两家初创公司，都靠着背后的干爹烧着钱，让别的创业公司基本上没活路，但是别忘了黄花梨做的椅子终究是要让人坐在屁股底下的。也预示着大多数创业公司的最终结果——被收购。
那么，在巨头们垄断式的市场里，如何脱颖而出呢？以下的内容基本上是读KK（凯文凯利）的文章的个人理解，理解有误的话请见谅。
KK说了，最具有破坏力的竞争来自于边缘而非中心。边缘是不被看好的市场。赢利低、不稳定、未经验证、市场狭小、风险系数极高。小公司不得不进入这样的市场。
就像上篇博文中提到的，蒸汽船开始时候故障多、昂贵等等原因都一度是他成为了摆设，但是经过几十年的改造，他把原来的造船行业全部打败了。60年代底特律的第一代汽车被一个叫本田的日本人用自动化的汽车打败了。IBM作为电脑公司也被微软这样做软件的公司不断蚕食，而微软如今又不得不面对一个做搜索引擎的公司（谷歌）的竞争。这些边缘公司却可以逐步打败那些老家伙，都是因为他们没有从正面进行较量，而是从老家伙们没有涉足的边缘进行发展（因为老家伙们需要完善现有产品，强化现有优势）。
作为一个赛车游戏的忠实粉丝，我深刻的知道想要超车必须从弯道，所有追赶都是没用的（除非你用外挂，你的车比别人快的多，也就是你有干爹的意思）。微创新这会，就是大公司用来扼杀小公司的手段，一定不是小公司用来颠覆大公司的武器。
360是从瑞星看不上眼的免费杀毒市场起家的，苹果是从IBM看不上眼的PC机起家的，微软也是从IBM看不上眼的操作系统起家，谷歌是从微软看不上眼的网络服务起家。但是现在的大公司都聪明了，像打车市场一出现点苗头，就被他们盯上了，所以，在这个时代想要从巨头们的眼皮底下做出点雷声大雨点更大的事情，就是一个字，难！
03年有几家闷声做大事的互联网公司是有起色，不过那真的就是巨头们不愿意涉及的边缘行业。谁能扎到下一个缝隙，爬到巨头们的头上呢？2014，让我们拭目以待。
--------------------------分割线---------------------------
# 常用工具Util
util 是一个 Node.js 核心模块，提供常用函数的集合，用于弥补核心 JavaScript 的功能过于精简的不足。Util模块的函数比较多，今天先说两个。
## 1 util.inherits
util.inherits(constructor, superConstructor)是一个实现对象间原型继承的原型复制函数。JavaScript 的面向对象特性是基于原型的，与常见的基于类的不同。不像Java中的对面对象特性是基于类的，可以用extends来实现继承，Javascript中需要通过原型复制来实现。这个问题可能要牵扯到javascript的原型概念，下次专门写一篇博文介绍javascript的原型。
先简单的打个比方，类就相当于一个模具，通过模具产生对象。而原型就好比达芬奇的蒙娜丽莎，其他艺术家通过高超的技艺复杂成一个一个对象。
看下面继承的例子：
```javascript testInherits.js
var util = require('util');  
function Base() {
this.name = 'base';
this.base = 1991;
this.sayHello = function() {
console.log('Hello ' + this.name);
};
}
Base.prototype.showName = function() {
console.log(this.name);
};
function Sub() {
this.name = 'sub';
}
util.inherits(Sub, Base);
var objBase = new Base();
objBase.showName();
objBase.sayHello();
console.log(objBase);
var objSub = new Sub();
objSub.showName();
//objSub.sayHello();
console.log(objSub);
```

``` Console
[David@localhost javascriptStudy]$ node testInherits.js 
base
Hello base
{ name: 'base', base: 1991, sayHello: [Function] }
sub
{ name: 'sub' }
```
Javascript的对象分三种：原型对象、构造对象、实例对象。
objSub是实例对象，如果//objSub.sayHello();不注释则会报错，因为只能继承Base原型的属性，不能继承Base构造对象的属性。
具体关于原型，还是请看之后写的关于原型的博文吧，想简单的解释实在太绕了。
## 2 util.inspect
util.inspect(object,[showHidden],[depth],[colors])是一个将任意对象转换为字符串的方法，通常用于调试和错误输出。它至少接受一个参数 object，即要转换的对象。
showHidden 是一个可选参数，如果值为 true，将会输出更多隐藏信息。
depth 表示最大递归的层数，如果对象很复杂，你可以指定层数以控制输出信息的多少。如果不指定depth，默认会递归2层，指定为 null 表示将不限递归层数完整遍历对象。
如果color 值为 true，输出格式将会以 ANSI 颜色编码，通常用于在终端显示更漂亮的效果。
从下面例子可以看出，就算有toString方法，也不会直接调用他来转换对象。
```javascript testInspect.js
var util = require('util');
function Person() {
this.name = 'byvoid';
this.toString = function() {
return this.name;
};
}
var obj = new Person();
console.log(util.inspect(obj));
console.log(util.inspect(obj, true));   
```

``` Console
[David@localhost javascriptStudy]$ node testInspect.js 
{ name: 'byvoid', toString: [Function] }
{ toString: 
   { [Function]
     [caller]: null,
     [arguments]: null,
     [length]: 0,
     [prototype]: { [constructor]: [Circular] },
     [name]: '' },
  name: 'byvoid' }
```
## 3 其他
util还提供了util.isArray()、util.isRegExp()、util.isDate()、util.isError() 四个类型测试工具，以及 util.format()、util.debug() 等工具。有兴趣的读者可以访问 [API](http://javascript.org/api/util.html)了解详细内容。有空我也会把util这个工具继续写下去。
