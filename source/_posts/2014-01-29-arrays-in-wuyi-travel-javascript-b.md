---
layout: post
title: '武夷游记-----javascript中的数组（二）'
date: 2014-01-29 03:53
comments: true
categories: [javascript, javascript, 游记, Node.js, 个人随笔, 数组]
---
趁着淡季去武夷山游玩了一圈，还是挺不错的。不用看着如山如海的人头，风景都变得格外好了。由于季节原因，没有玩什么九曲竹筏漂流，但是此行的目的还是达到了。很好的放松了下身心，然后和朋友们一起出行，这路途本身就是一种享受，然后也吃到了一些特色的吃的，比如熏鹅、孝母饼。不过印象最深的就是，在武夷山读书3年的朋友，竟然把我们带错路了，我们只看着玉女峰离我们越来越远，由此可总结，去旅游千万别让路痴当导游，就算是他主场，他也能毫不犹豫的带错路。不过这样一来，我们一路吐槽反而变得非常有乐趣。
一回家就想写博客，写博客真成了一种病了。接下来javascript数组的第二篇，最后还有个第三篇，本系列就结束了。
-----------------------分割线：序号接第一篇-------------------------
##2.7 Map映射
Map映射会将数组中所有有赋值（是赋值，null、false、空串、void 0等都会执行。）的元素都调用回调函数，再将返回的值组成一个数组。
Map和foreach、some、every有点像，只是map会将返回值组成一个新的数组。
在python中可以通过下面这种方式来达到map同样的效果。
```python
sanitize(time_string) for time_string in dataList
```
接着让我们来看看javascript中的例子。
```javascript testmap.js
values = [void 0, null, false, '','5'];                                         
values[7] = void 0;
result = values.map(function(value, index, array){
    console.log('The '+index+'th element is '+value);
    return value;
});
console.log('The new array is '+result);
~                          
```
输出：
``` Console
[David@localhost arraytest]$ node testmap.js 
The 0th element is undefined
The 1th element is null
The 2th element is false
The 3th element is 
The 4th element is 5
The 7th element is undefined
The new array is ,,false,,5,,,
```
可以看出，索引为5和6的元素没有执行回调函数，因为他们并没有被赋值。

我们还可以通过map来获取一个映射模型（python叫字典，java中叫hashmap）。
```javascript Map.js
function Map(){
    this._map={};
}
Map.prototype.getMap = function(items){
    return items.map(function(item,index,array){
        return {
            id:index,                                                           
            name:item
        }
    });
}
module.exports = Map;
~                    
```

``` Console
[David@localhost arraytest]$ node
var Map = require('./Map.js');var map = new Map();var newmap = map.getMap(['1','3','4']);newmap;
[ { id: 0, name: '1' },
  { id: 1, name: '3' },
  { id: 2, name: '4' } ]
>
```

##2.8 Filter过滤
用filter方法，可以将数组的每个有赋值的元素都执行回调函数，将返回true的数组组成一个新的数组集合。
用法：.filter(fn(value, index, array), thisArgument)
例子如下：
```javascript testfilter.js
var oldarray = [void 0, null, false, '', 1];
oldarray[8]='0';
var trueresult = oldarray.filter(function (value) {
    console.log('true filte :'+value);
    return value;
    });
var falseresule = oldarray.filter(function (value) {
    console.log('false flite :'+value);                                         
    return !value;
});

console.log('true array is '+trueresult);
console.log('false array is '+falseresule);
~                                         
~                    
```
输出：
``` Console
[David@localhost arraytest]$ node testfilter.js 
true filte :undefined
true filte :null
true filte :false
true filte :
true filte :1
true filte :0
false flite :undefined
false flite :null
false flite :false
false flite :
false flite :1
false flite :0
true array is 1,0
false array is ,,false,
```
从结果可以看出，有值的才执行回调函数，回调函数返回true的原数组元素会组成一个新的数组。
另外可以看出0在javascript中是true，C语言玩习惯的人还真要注意这个。
##2.9 Sort排序
在sort方法中如果未提供回调函数，那么将使用字典序的顺序排，这样60会排在9的前面。
```javascript
[60,9,23,10,28].sort();
[ 10,  23,  28,  60,  9 ]
```
回调函数有两个参数a,b，返回小于0，a在b前。等于0或者NaN则按数组原始的先后顺序。而且是用冒泡排序的方式进行排序的。
```javascript
['a',60,9,23,70,10].sort(function(a,b){ console.log('a:'+a+'- b:'+b+'='+(a-b)); return a-b;});
a:a- b:60=NaN
a:60- b:9=51
a:a- b:9=NaN
a:60- b:23=37
a:9- b:23=-14
a:60- b:70=-10
a:70- b:10=60
a:60- b:10=50
a:23- b:10=13
a:9- b:10=-1
[ 'a',
  9,
  10,
  23,
  60,
  70 ] 
```

##2.10 Reduce和reduceRight计算
Reduce为从左到右，reduceRight为从右到左。每次回调函数接收到目前为止的部分结果和当前遍历的值，最后返回一个总计值。
用法：reduce(callback(previousValue, currentValue, index, array), initialValue)。

*	previousValue是上一次被调用的回调函数的返回值
*	initialValue是开始时previousValue被初始化的值
*	currentValue 是当前被遍历的元素值
*	index是当前元素在数组中的索引值
*	array是对调用.reduce数组的简单引用。

下例是用reduce来数组的总和：
```javascript
Array.prototype.sum = function(){return this.reduce(function(preValue,value,index){return preValue+value},0)}
[Function]
['a',60,9,23,70,10].sum()
'0a609237010'
[60,9,23,70,10].sum()
172
```

