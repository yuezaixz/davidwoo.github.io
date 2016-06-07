---
layout: post
title: 'javascript中的正则表达式（二）---基础知识、具体实例'
date: 2014-02-04 05:51
comments: true
categories: [javascript, javascript, Regexp, 正则表达式]
---
接上一章，基础知识补充两点，另外举3个实例，最后一个实例为实际应用的模块。
##1.9 小刮号
用小刮号可以把一个正则表达式划分成几个字匹配，比如看下面的例子。
```javascript
reg = /(\w+)@(\w+)\.com/g
/(\w+)@(\w+)\.com/g
mailstr
'xiao303178384@qq.com'
mailstr.replace(reg,"<$1>")
'<xiao303178384>'
mailstr = 'xiao303178384@qq.com'
'xiao303178384@qq.com'
mailstr.replace(reg,"<$1>@$2.com")
'<xiao303178384>@qq.com'
```

$1为第一个小刮号的子匹配，$2就为第二个子匹配。
##1.10 Replace用回调函数
用replace的第二个参数可以为一个回调函数，返回的值为替换的结果。回调函数有3个参数。
```javascript
name
'david wu'
reg
/\b\w+\b/g
name.replace(reg,function(){console.log(arguments)})
{ '0': 'david', '1': 0, '2': 'david wu' }
{ '0': 'wu', '1': 6, '2': 'david wu' }
```
可以看出，第一个参数为匹配的字段，第二个参数为匹配字段的索引，第三个字段为完整字符串。
具体应用可以看以下例子，以下例子将一串字母的每个单词首字母变为大写。
```javascript
name
'david wu'
reg
/\b\w+\b/g
name.replace(reg,function(word){return word.substring(0,1).toUpperCase()+word.substring(1);})
'David Wu'
```

#2 实例
##2.1 是否为URL
这个当前项目中用于验证是否为正确的url的正则。
```javascript
url = 'http://192.168.0.39:9080/'
'http://192.168.0.39:9080/'
isurl
/^((https|http|ftp|rtsp|mms)://)?[0-9]{1,3}.[0-9]{1,3}.[0-9]{1,3}.[0-9]{1,3}(:[0-9]{1,4})?/?$/
isurl.test(url)
true
url = 'http://192.168.0.39:9080'
'http://192.168.0.39:9080'
isurl.test(url)
true
url = 'http://192.168.0.39'
'http://192.168.0.39'
isurl.test(url)
true
url = '192.168.0.39'
'192.168.0.39'
isurl.test(url)
true
```
##2.2 是否为18位身份证号
身份证号都为18位，第一位为非0数字，后16位数字，最后一位可以是数字或者字母。
```javascript
isidcard = /[1-9][\d]{16}[0-9A-Z]/g
var idcard1 = '350781198811140019'; 
undefined
var idcard2 = '35078119881114001Z'; 
undefined
idcard1.match(isidcard)
[ '350781198811140019' ]
idcard2.match(isidcard)
[ '35078119881114001Z' ]
```
##2.3 带校验位的18位身份证号验证
身份证的最后一位是校验位，根据前17位算出最后一位，最后验证最后一位是否正确。下例就为一个实际的身份证验证模块。
```javascript idcard.js
function IdCard(idcard){                                                        
    this._idcard = idcard;
}

module.exports = IdCard;

IdCard.prototype.check = function checkidcard(){
    var result = false;
    var checkchar = ['1','0','X','9','8','7','6','5','4','3','2'];
    var checkweight = [7,9,10,5,8,4,2,1,6,3,7,9,10,5,8,4,2,1,0];
    var isidcard = /[1-9][\d]{16}[0-9A-Z]/g;
    if(isidcard.test(this._idcard)){
        var sumMult = 0;
        for(var i = 0;i<=16;i++){
            var ch = this._idcard.substring(i,i+1);
            sumMult = sumMult + ch * checkweight[i];
        }
        if(this._idcard.substring(17,18) === checkchar[sumMult%11]  ){
            result = true;
        }
    }
    return result;
}
```

```javascript 控制台
var IdCard = require('./idcard');var idCard = new IdCard('350781198811140019');idCard.check();
true
var idCard = new IdCard('350781198811140041');idCard.check();
false
var idCard = new IdCard('350781198811140043');idCard.check();
true
```
