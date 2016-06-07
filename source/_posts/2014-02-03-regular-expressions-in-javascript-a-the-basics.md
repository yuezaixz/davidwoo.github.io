---
layout: post
title: 'javascript中的正则表达式（一）---基础知识'
date: 2014-02-03 13:11
comments: true
categories: [javascript, javascript, Regexp, 码年, 正则表达式]
---
#码年快乐
过年可以算是玩疯了，初一到初三，就没咋闲着。不过，可以确定的是，过年也不能得瑟啊，除夕一朋友打麻将输了1000个字，我信誓旦旦的说，我这辈子打麻将都没输过1000个字。然后，初一下午我就输了1030个字= =满脸黑线，这绝对是诅咒，没想到小伙伴你还会诅咒啊。
过年还有个比较快乐的事情就是和好基友打DOTA各种虐菜，我们这些无耻的人，就在大过年把自己的快乐建立在别人的悲催纸上，海牙红毛、小精灵红猫、跳刀AA和跳刀军团，游走的飞起。
然后还玩了游乐场，而且还做了旋转木马，可以说，下巴差点笑脱臼，脸部肌肉已经笑抽筋了。
顺便还去看了电影，爸爸去哪儿，还是挺有笑料的，哈哈哈。
其他开心的事情还做了很多，就不一一晒出来了。
初四了，该稍微收一收心了，今天去外婆家住3天，陪陪老人家，也顺便调整下状态。今天除了教外婆用我新买个外婆的智能机以外，就是学习了。最近觉得javascript的正则有些薄弱，就好好的补课了下。接下来的两篇博文就写正则吧。第一篇为基础知识篇，第二篇我会整理写项目中用到的正则来说明下。

#1 基本知识
是对字符串执行模式匹配的强大工具。使用正则表达式有两种方法：RegExp对象和直接量语法，如果觉得自己基本知识都OK，那跳过这段别看吧，很多东西我也就是从官方文档搬的。
##1.1 RegExp对象
创建RegExp对象语法：new RegExp(pattern, attributes)

```javascript
var  reg=new RegExp("[0-9]","g");
var str="abd1afa4sdf";
str.replace(reg,"num");
'abdnumafanumsdf'

```

Regexp对象有test(是否匹配)、exec（查找下一个匹配）这两个方法。

```javascript
reg = /\d/g
reg.test(str)
reg.exec(str)
//[ '4',  index: 7,  input: 'abd1afa4sdf' ]
reg = /\d/
reg.exec(str)
//[ '1',  index: 3,  input: 'abd1afa4sdf' ]
reg.exec(str)
//[ '1',  index: 3,  input: 'abd1afa4sdf' ]
```


```javascript
reg
/<%([^%>]+)?%>/g
str = 'This is <%a%test <%template%>'
'This is <%a%test <%template%>'
reg.exec(str)
[ '<%a%>',
  'a',
  index: 8,
  input: 'This is <%a%test <%template%>' ]
reg.exec(str)
[ '<%template%>',
  'template',
  index: 19,
  input: 'This is <%a%test <%template%>' ]
```


##1.2 String对象的正则方法
| Tables        |方法 	|描述|
|--------------------|:---------:|:-----:|
|col1|search 	|检索与正则表达式相匹配的值|
|col2|match 	|找到一个或多个正则表达式的匹配|
|col3|replace 	|替换与正则表达式匹配的子串|
|col4|split 	|把字符串分割为字符串数组|
```javascript
str
'abd1afa4sdf'
reg
/\d/g
str.search(reg)
3
str.match(reg)
[ '1', '4' ]
str.split(reg)
[ 'abd', 'afa', 'sdf' ]
str.replace(reg,'num')
'abdnumafanumsdf'
```

##1.3 直接量语法
直接量语法：/pattern/attributes
```javascript
var  reg = /[0-9]/g
var str="abd1afa4sdf";
str.replace(reg,"num");
'abdnumafanumsdf'
```

##1.4 参数说明

* 	参数 pattern 是一个字符串，指定了正则表达式的模式或其他正则表达式。
* 	参数 attributes 是一个可选的字符串，包含属性 "g"、"i" 和 "m"，分别用于指定全局匹配、区分大小写的匹配和多行匹配。ECMAScript 标准化之前，不支持 m 属性。如果 pattern 是正则表达式，而不是字符串，则必须省略该参数。

##1.5 Attributes修饰符
| Tables        |修饰符 	|描述|
|--------------------|:---------:|:-----:|
|col1|i 	|执行对大小写不敏感的匹配。|
|col2|g 	|执行全局匹配（查找所有匹配而非在找到第一个匹配后停止）。|
|col3|m 	|执行多行匹配。|
##1.6 方刮号
| Tables        |表达式 	|描述|
|--------------------|:---------:|:-----:|
|col1| [abc] 	|查找方括号之间的任何字符。|
|col2| [^abc] 	|查找任何不在方括号之间的字符。|
|col3| [0-9] 	|查找任何从 0 至 9 的数字。|
|col4| [a-z] 	|查找任何从小写 a 到小写 z 的字符。|
|col5| [A-Z] 	|查找任何从大写 A 到大写 Z 的字符。|
|col6| [A-z] 	|查找任何从大写 A 到小写 z 的字符。|

```javascript
reg = /[0-9]/g
var str="abd1afa4sdf";
str.match(reg)
[ '1', '4' ]
reg = /[^1-9]/g
/[^1-9]/g
str.match(reg)
[ 'a',  'b',  'd',  'a',  'f',  'a',  's',  'd',  'f' ]
```

##1.7 元字符
| Tables        |元字符 	|描述|
|--------------------|:---------:|:-----:|
|col1|. 	|查找单个字符，除了换行和行结束符。|
|col2| \w 	|查找单词字符。|
|col3| \W 	|查找非单词字符。|
|col4| \d 	|查找数字。|
|col5| \D 	|查找非数字字符。|
|col6| \s 	|查找空白字符。|
|col7| \S 	|查找非空白字符。|
|col8| \b 	|匹配单词边界。|
|col9| \B 	|匹配非单词边界。|
|col10| \0 	|查找 NUL 字符。|
|col11| \n 	|查找换行符。|
|col12| \f 	|查找换页符。|
|col13| \r 	|查找回车符。|
|col14| \t 	|查找制表符。|
|col15| \v 	|查找垂直制表符。|
|col16| \xxx 	|查找以八进制数 xxx 规定的字符。|
|col17| \xdd 	|查找以十六进制数 dd 规定的字符。|
|col18| \uxxxx 	|查找以十六进制数 xxxx 规定的 Unicode 字符。|

```javascript
reg = /./g
/./g
str.match(reg)
[ 'a',  'b',  'd',  '1',  'a',  'f',  'a',  '4',  's',  'd',  'f' ]
reg = /\d/g
/\d/g
str.match(reg)
[ '1', '4' ]
reg = /\D/g
/\D/g
str.match(reg)
[ 'a',  'b',  'd',  'a',  'f',  'a',  's',  'd',  'f' ]
reg = /\w/g
/\w/g
str.match(reg)
[ 'a',  'b',  'd',  '1',  'a',  'f',  'a',  '4',  's',  'd',  'f' ]
reg = /\W/g
/\W/g
str.match(reg)
null
```

##1.8 量词
| Tables        量词 	|描述|
|--------------------|:---------:|:-----:|
|col1|n+ 	|匹配任何包含至少一个 n 的字符串。|
|col2|n\* 	|匹配任何包含零个或多个 n 的字符串。|
|col3|n? 	|匹配任何包含零个或一个 n 的字符串。|
|col4|n{X} 	|匹配包含 X 个 n 的序列的字符串。|
|col5|n{X,Y} 	|匹配包含 X 或 Y 个 n 的序列的字符串。|
|col6|n{X,} 	|匹配包含至少 X 个 n 的序列的字符串。|
|col7|n$ 	|匹配任何结尾为 n 的字符串。|
|col8|^n 	|匹配任何开头为 n 的字符串。|
|col9|?=n 	|匹配任何其后紧接指定字符串 n 的字符串。|
|col10|?!n 	|匹配任何其后没有紧接指定字符串 n 的字符串。|

```javascript
str.match(reg)
[ 'abd', 'afa', 'sdf' ]
str
'abd1afa4sdf'
reg = /(\D+)/g
/(\D+)/g
str.match(reg)
[ 'abd', 'afa', 'sdf' ]
```

测试邮箱
```javascript testMail
var mailstr = 'xiao303178384@gmail.com'
undefined
reg = /(\w+)@(\w+)\.com/g
/(\w+)@(\w+)\.com/g
reg.test(mailstr)
true
mailstr = 'xiao303178384@gmail.co'
'xiao303178384@gmail.co'
reg.test(mailstr)
false
mailstr = 'xiao303178384@gmailcom'
'xiao303178384@gmailcom'
reg.test(mailstr)
false
mailstr = 'xiao303178384@qq.com'
'xiao303178384@qq.com'
reg.test(mailstr)
true
```
