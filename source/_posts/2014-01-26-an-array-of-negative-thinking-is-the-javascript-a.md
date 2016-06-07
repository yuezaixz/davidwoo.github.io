---
layout: post
title: '是否该持否定性思维---javascript中的数组（一）'
date: 2014-01-26 04:18
comments: true
categories: [javascript, javascript, 个人随笔, 否定性思维, 自我反思, 数组]
---
总算回到家了，一下车立马和小伙伴们吃久违的邵武紫溪粉去，不说有多好吃，吃的是一种回忆，人们泛着小舟逆流而上时，浪水也不断把我们打回到过去，而我们也努力在这间歇体验那曾经的美好，之后又继续摇起船桨奋力前进，直到有一天我们再也回不到过去了。
成长总是伴随着一些忧伤的，就像曾经的我也考公务员，最终结果是进入面试后差了0.6分。这次回来各种亲戚还依然惦记着我的公务员，不断问我，什么时候再考啊。其实我心里已经不想考了，我对公务员想法已经转变了，我认为让一个社会的优秀人才都去做公务员本来就是一种时代的退步，更何况公务员消减一半也丝毫不会影响政府工作。
但是，当我不断对一个东西持否定态度的时候我就要开始考虑一个问题了，我这是陷入否定性思维的泥沼了吗？否定性思维就是一个人总喜欢对任何事物持否定态度。哎，这人这么有钱，一定是黑商；这女明星这么红，一定是给潜规则了，一定有干爹；互联网金融一定是泡沫，你们这些人就等着泡沫破灭吧……这种人喜欢用否定别人的方式来让自己的失败变的理顺当然。当一个事物出现，有些人很好，他们想办法应对这种改变；而又有一部分人就喜欢做否定者，不断的否定他们，就想把你的智商拉到和他一样的水平线上，然后以他丰富的白痴经验打败你。或许他们的否定是对的，但是，他们这样的人我认为是不会成功的。
所以我就陷入沉思了，我是因为自己没考上，然后才持否定态度吗？我觉得，真不是，至少不完全是。我没有继续考，也主要是因为这个公务员这个稳定的米缸在我看来一是很无趣，让生活失去目标，二是这个米缸的好日子也差不多到头了吧，米缸里面的米已经被大批大批的老鼠吃的差不多了哦。不想谈政治，只是突发奇想的思考了下自己是否变成了一个否定者了，很好的是，我还不完全是，但是，公务员怎么怎么样又关我何事要我去唠叨呢？所以避免言多有失，我还是少说话，多敲代码吧。

------------------------------------分割线-----------------------------------
util篇的第二篇暂时不准备写，因为我觉得还有很多各重要的知识应该要先好好巩固下，比如javascript的数组对象。内容比较多，分几篇博文来写吧。
在Javascript中，我经常用[]的方式来便捷的创建数组（当然也可以用Array构造函数），但是我并不了解他。由于项目中有用到，所以特地找了一些资料学习了下。本文就是javascript数组的学习笔记。
#1 概述
数组对象继承自Object.prototype，对数组执行typeof操作符返回‘object’而不是‘array’。然而执行[] instanceof Array返回true。此外，还有类数组对象使问题更复杂，如字符串对象，arguments对象。arguments对象不是Array的实例，但却有个length属性，并且值能通过索引获取，所以能像数组一样通过循环操作。
 
#2 数组原型的方法

*	循环.forEach
*	断言.some和.every
*	.join和.concat的区别
*	栈和队列.pop，.push，.shift和.unshift
*	模型映射.map
*	查询.filter
*	排序.sort
*	计算.reduce和.reduceRight
*	复制.slice
*	万能的.splice
*	查找.indexOf
*	in操作符
*	reverse

##2.1 循环foreach
该方法IE9+上才支持。所以现在公司不允许使用该函数。
使用形式：[].foreach(callback)
回调函数传入三个参数如下：

*	value 当前操作的数组元素
*	index 当前操作元素的数组索引
*	array 当前数组的引用

```javascript testForeach.js
['_', 't', 'a', 'n', 'i', 'f', ']'].forEach(function (value, index, array) {
    this.push(String.fromCharCode(value.charCodeAt() + index + 2))
    }, out = [])

console.log(out.join(''));                                                      
console.log(out);       
```

``` Console
[David@localhost arraytest]$ node testForeach.js 
awesome
[ 'a', 'w', 'e', 's', 'o', 'm', 'e' ]
```
可以传递可选的第二个参数(上例中的out)，作为每个调用函数的上下文（this）。
我们不能打断foreach，如果需要打断，需要用到some和every，请继续往下看。
##2.2 可退出循环some、every
在java中，你可能会用到break去跳出循环，那么再javascript中要怎么做呢？
Some和every使用形式：[].some(callback)/ [].every(callback)，回调函数传入三个参数与foreach相同。当回调函数返回true时候some就会中断，some的返回结果也就会true，代表找到了寻找的元素，而every想法，当false时候中断，但是every要注意的是，如果要every返回true，需要所有的遍历元素都返回true。
```javascript testSomevery.js
    if(tempStr === value){
        return true;
    }
});
}

function excludeStr(tempStr){
return tempArray.every(function(value,index,array){
    if(tempStr === value){
        return false;
    }else{
        return true;
    }
});

}

var param = process.argv[2];
console.log("Array ["+tempArray+"] contains string '"+param+"'? "+includeStr(param));
console.log("Array ["+tempArray+"] exclude string '"+param+"'? "+excludeStr(param));      
```

``` Console
[David@localhost arraytest]$ node testSomeEvery.js David
Array [David,Wu,Love,TY,forever,hahaha] contains string 'David'? true
Array [David,Wu,Love,TY,forever,hahaha] exclude string 'David'? false
[David@localhost arraytest]$ node testSomeEvery.js ty
Array [David,Wu,Love,TY,forever,hahaha] contains string 'ty'? false
Array [David,Wu,Love,TY,forever,hahaha] exclude string 'ty'? true
[David@localhost arraytest]$ node testSomeEvery.js Ty
Array [David,Wu,Love,TY,forever,hahaha] contains string 'Ty'? false
Array [David,Wu,Love,TY,forever,hahaha] exclude string 'Ty'? true
```
##2.3 Join，数组插入分隔符
join(分隔符)方法创建一个字符串，会将数组里面每个元素用分隔符连接。如果没有提供分隔符，默认的分隔符为“,”。
```
[David@localhost arraytest]$ node
 var tempArray = ['David','Wu','Love','TY','forever','hahaha'];
undefined
 tempArray.join();
'David,Wu,Love,TY,forever,hahaha'
 tempArray.join('');
'DavidWuLoveTYforeverhahaha'
 tempArray.join(' ');
'David Wu Love TY forever hahaha'
```
##2.4 Concat,数组浅复制
concat方法创建一个新数组，其是对原数组的浅拷贝。
```
 var David = {firstname:'david',lastname:'wu'}
undefined
/ var src = [1,2,3,David]
undefined
/var target = src.concat()
undefined
/src === target
false
/src[3] === a
/src[3] === David
true
/target[3] === David
true
```
===在比较对象的时候是比较引用（值类型是比较值，比如number,string等是值类型，而一般对象都是引用类型，比较引用），就是是否为同一个指针的指向，也就是内存地址是否相同。所以拷贝后的target与src比较返回false，因为target是拷贝后的结果。
但是，因为是浅拷贝，所以数组内的元素还是用相同的指针指向，所以是相同的。
Concat还能加入不限制个数参数，参数可以是值，也可以是一个数组，都会把它加入到数组的末端。

```javascript
src.concat(target)
[ 1,
  2,
  3,
  { firstname: 'david', lastname: 'wu' },
  1,
  2,
  3,
  { firstname: 'david', lastname: 'wu' } ]
src.concat('tang','ying','hahaha')
[ 1,
  2,
  3,
  { firstname: 'david', lastname: 'wu' },
  'tang',
  'ying',
  'hahaha' ]

```

##2.5 Pop、push实现栈
使用push可以压入一个元素或者一个数组。
而用pop则移除数组的末尾元素，并移除。如果数组是空返回返回void 0（undefined）。用这两个方法可以实现LIFO的栈数据结构。

```javascript
function Stack(){this._statck=[]}
Undefined
Stack.prototype.next = function(){return this._statck.pop();}
[Function]
Stack.prototype.add = function(arrayx){return this._statck.push(arrayx);}
[Function]
stack = new Stack()
{ _statck: [] }
stack
{ _statck: [] }
stack.add(1);
1
stack.add('David');
2
stack.add('Wu');
3
stack.add('tang');
4
stack.add('ying');
5
stack
{ _statck: 
   [ 1,
     'David',
     'Wu',
     'tang',
     'ying' ] }
stack.next()
'ying'
stack.next()
'tang'
stack.next()
'Wu'
stack.next()
'David'
stack.next()
1
stack
{ _statck: [] }

```

这里可以用apply稍微改造下该方法。至于apply是什么，可以参考该文章[Apply方法详解](http://www.cnblogs.com/KeenLeung/archive/2012/11/19/2778229.html)。

```javascript Stack.js
function Stack(){
    this._stack = [];
}

Stack.prototype.next = function(){
    return this._stack.pop();
}
Stack.prototype.add = function(){
    return this._stack.push.apply(this._stack,arguments);
}
module.exports = Stack;                                                         

```
```
var Stack = require('./Stack')
undefined
stack = new Stack();
{ _stack: [] }
stack.add('David','Wu',1988,1114)
4
stack.next();
1114
stack
{ _stack: [ 'David', 'Wu', 1988 ] }
stack.next();
1988
stack.next();
'Wu'
stack
{ _stack: [ 'David' ] }
>
```

##2.6 shift和unshift实现队列
用unshift 和 shift可以很好的模拟FIFO（先进先出）队列。

```javascript Queue.js
function Queue(){
    this._queue = [];
}

Queue.prototype.next = function(){
    return this._queue.shift();
}
Queue.prototype.add = function(){
    return this._queue.unshift.apply(this._queue,arguments);                    
}
module.exports = Queue;

```

```
var Queue = require('./Queue');var queue = new Queue();
undefined
queue.add(1,3,5,7);
4
queue
{ _queue: [ 1, 3, 5, 7 ] }
queue.next()
1
```
