---
layout: post
title: '瞎扯工作的乐趣--------javascript中的数组（三）'
date: 2014-01-29 07:10
comments: true
categories: [javascript, javascript, Node.js, 工作, 个人随笔, 数组, 兴趣]
---
明天就除夕了，想到好几天估计都写不了博文，今天就早上写一篇，下午写一篇了。
教育真是个麻烦的问题啊，我是还没那么快要愁这个问题，但是我弟的教育问题还真是棘手啊。由于我是各种野，从小就有机会就往外跑，标准型的非宅男，所以家里觉得我太野了管不住。之后我弟的教育就是限制的很死，一直把他管在家里。但是他现在是赶都赶不出去的节奏。
但是你以为他窝在家里是学习，你就错了。可以玩游戏就玩游戏，不能玩游戏就用手机看游戏对战的录像，不让看录像就玩手机，不让玩手机你以为他就真的会去读书你就更错了。他坐在桌子前剪纸张也不做作业的。就算墙壁着，在旁边监督，他也会各种发呆。
所以经常家里听到的一句话，怎么这么懒啊，当初爸妈读书环境是多么艰苦啥啥啥的bilibala一堆。虽然是为了教育，但是我觉得这种方式不对。
第一，这绝对不是因为懒，是因为兴趣问题，或者是一直就没有一个引起孩子乐于看书的一个契机。从小我就讨厌写作文，高三时候甚至决定，毕业后再也不写作文。但是，后来我就爱上写随笔，写杂文，偶尔还装下B，学下文艺青年写散文。应试教育的目的就是让你对所有的学科都拿不出兴趣来。为什么我对理科情有独钟，我觉得很大一部分因素是小学时候的数学老师我很喜欢，当时有很多有趣的奥数问题我都很爱去请教他，再之后说数学学得好，理化生就一定学得好，所以我理科全部都很好。高中时候有一次考试没考语文和英语，我竟然考进了年段前31，那可是比厦大分数高的多，就差清华不远的分数啊。可见，兴趣多关键，如果启蒙教育的时候能让我对英语、语文感兴趣，我深刻的觉得，我之后的四年应该是坐在清华的校园里面。。。。。。。打DOTA的，想一想还有点小激动哦。开个玩笑，歹势啊，哈哈哈！
第二，为什么一定要提那么多环境因素的苦难，难道是苦难是成功之爹吗？司马迁因为受了宫刑才写成了史记？孙膑因为膝盖XX才著成孙子兵法？奥特曼因为被怪兽暴打一顿最后才能战胜怪兽吗？我觉得都是瞎扯，苦难者到处说自己的苦难无非两种心理，一是希望得到鼓励，二是觉得自己起点低有这样的成就已经是人上人了，你们应该赞同我。
不小心扯远了点，总之，教育不在于说“苦”，而在于“趣”。那如何才能产生兴趣呢？我觉得在于“深入”。一首歌对于一个普通听众来说，感受就是好听，但对于一个专业音乐人来说，他可能就会感叹，这首歌的大三度升KEY实在是太美妙了。可以说，钻研的越深，获得的乐趣也就越大。很多人说程序员苦逼，很多人程序员也就觉得自己苦逼。但是你真的了解到程序的乐趣了吗？成天搬代码能有快乐吗？是否深入考虑下有没有什么更好的实现？为什么要搬一样的代码，可以不可以用模板引擎搞定？java实现一个功能要这么多代码，是否有更高级的语言，更灵活的实现？是否有更好的设计模式？这套架构设计的这么精妙的事情你爸妈知道吗？
开始我就说了吧，我是瞎扯，扯了这么多还是回到标题吧，就是一句话， **做什么你只要认真做，用脑子做，做的深入了，工作的乐趣也就产生了，不要成天抱怨工作了，更不要抱怨别人的年终奖金有多少，与其临渊羡鱼，不如退而结网。** 写给所有人，同时也写给自己，保持对工作的乐趣。

--------------------------------------分割线：序号延续二篇----------------------------
##2.11 转换成数组slice
Slice把类数组浅拷贝转换成新的数组，他需要两个参数，一个是开始位置，一个是结束位置。
```javascript
Array.prototype.slice.call({ 0: 'David', 1: 'Wu',3:'Hello', length: 2 })
//['David', 'Wu']
```

除此之外，另一个常见用途是从参数列表中移除最初的几个元素，并将类数组对象转换为真正的数组。比如下面例子中，就是在用call调用方法的时候，用来从指定开始位置截取传入参数。
```javascript format.js
function format (text, bold) {
    if (bold) {
        text = '<b>' + text + '</b>'
    }
    console.log(arguments)                                                      
    var values = Array.prototype.slice.call(arguments, 2);
    values.forEach(function (value) {
        text = text.replace('%s', value);
    })

    return text;
}
module.exports = format;
~        
```
输出如下:
``` Console
var format = require('./format.js');format('some%sthing%s %s', true, 'some', 'other', 'things');
{ '0': '<b>some%sthing%s %s</b>',
  '1': true,
  '2': 'some',
  '3': 'other',
  '4': 'things' }
'<b>somesomethingother things</b>'
```

##2.12 万能删除插入函数splice
splice允许你在index索引后删除howmany个元素，并在该位置插入任意数量新元素，这种操作是在原数组上的修改，返回值为被删除的元素。
用法：arrayObject.splice(index,howmany,item1,.....,itemX)
例子：
```javascript
[David@localhost arraytest]$ node
var source = [1,2,3,8,8,8,8,8,9,10,11,12,13];
var spliced = source.splice(3, 4, 4, 5, 6, 7)
//console.log(spliced);
[ 8, 8, 8, 8 ]
//console.log(source);
[ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13 ]
```

source.splice(3, 4, 4, 5, 6, 7)意思是在索引为3的位置开始删除4个元素，插入4,5,6,7这几个元素。

##2.13 查找函数indexOf
可以在数组中寻找指定变量的索引（值类型比较值，引用类型比较引用），如果没找到返回-1。函数第一个参数为查找的变量，第二个参数为开始查找的位置。Java中的数组、字符串等也都有indexOf函数，特别是字符串的该函数，用法很类似。看以下例子：
```javascript
var a = {'firstname':'David','lastname':'Wu'}
var man = {'firstname':'David','lastname':'Wu'}
var woman = {'firstname':'Ying','lastname':'Tang'}
var family = [man,woman,'baby is null']
//查找不存在的变量
family.indexOf('1')
-1
//查找存在变量值相同但引用不同的引用类型对象，结果是找不阿东
family.indexOf({'firstname':'David','lastname':'Wu'})
-1
//同上，只是换了种形式
family.indexOf(a)
-1
//相同引用的引用类型对象，可以找到
family.indexOf(man)
0
//同上
family.indexOf(woman)
1
//相同值的值类型对象，可以找到
family.indexOf('baby is null')
2
//从索引1开始找，所以0位置的变量找不到
family.indexOf(man,1)
-1
family.indexOf(woman,1)
1
```

2.14 索引检索函数in
别把它和indexOf混淆了。in操作符通过检索对象的键(key)来寻值，而不是搜索值。当然，这在性能上比.indexOf快得多。还是上面的例子的变量。
```javascript
1 in family
true
man in family
false
2 in family
true
3 in family
false
```

2.15 数组反转函数reverse
将数组反转，原数组改变，并返回。
```javascript
family
[ { firstname: 'David', lastname: 'Wu' },
  { firstname: 'Ying', lastname: 'Tang' },
  'baby is null' ]
family.reverse()
[ 'baby is null',
  { firstname: 'Ying', lastname: 'Tang' },
  { firstname: 'David', lastname: 'Wu' } ]
family
[ 'baby is null',
  { firstname: 'Ying', lastname: 'Tang' },
  { firstname: 'David', lastname: 'Wu' } ]
```
