---
layout: post
title: '百家争鸣（一）---Node.js版FizzBuzzWhizz'
date: 2014-05-07 13:34
comments: true
categories: [javascript, fizzbuzzwhizz, 易中天, 百家争鸣]
---
百家争鸣-读书感记事（一）
----------------------------------------

这本书是我在亚马逊Kindle特价上淘来的，看这本书之前我看过易中天老师两本书，一本是品三国，没有全看就是随便翻阅下，精彩是很精彩，但是没有太大感悟，另一本则是我觉得很有感悟的一本书，书名有好几个版本，书名我是忘了，《美国宪法的诞生和我们的反思》是我百度出来的，大概说的就是美国1785年（时间我也不确定了，大概这个时间吧）的立宪会议。这本书讲述了美国从被统治到独立后的一段混沌的时代，从这本书中不仅提到了麦迪逊、华盛顿的美国开国元勋们制定宪法的会议记录，如果只是这样，我也就不崇拜易中天老师了，他通过联邦、邦联、民主、共和、人民、公民等词来叙述了这个过程，由于本文主题不是这个，所以不细说。
这本书是我看的第三本易中天老师的书，也是通过白话的方式叙述了中国古代的圣人们的救世之道。从孔子孟子仁爱的儒家，到墨子博爱的墨家，到杨朱、庄子、老子一毛不拔的到家，最后到君权至上的法家，易老师从各种视角剖析，可以说精彩有余而感悟甚深。其中我最喜欢庄子，庄子就像禅宗，是不能说的，一说便错。
接下来会陆续的把这本书的读书感发出来。
抄袭庄子的一句话做本文的开头吧。
我：媳妇，你看庄子他真是而自由，就像那浪里白条的鲦鱼一样，在水里尽情的游弋，他其实就是享受着天地之间尽情游弋的“鱼”的快乐啊！
媳妇：你又不是庄子，你怎么知道庄子的快乐呢？
我：你又不是我，你怎么知道我不知道庄子的快乐呢？

Node.js版FizzBuzzWhizz
-----------------------------------------------------------
和python版本比较，该版本只是把一些方法抽成了函数，并无太大改善。主要是在python版本和java版本花了一些心思设计，不罗嗦，直接上代码，有兴趣的研究下
```javascript 
// 获得用户输入                                                                  
var nums = process.argv[2];
if (!nums) die('please enter three numbers. example: 3,5,7');
nums = nums.split(',');
if (nums.length !== 3) die('please enter three numbers. example: 3,5,7');
nums.forEach(function (n) {
  if (!(n 0 && n < 10)) die(n + ' is not an valid number. must be greater than 0 and less than 10');
});

function die (msg) {
  console.log(msg);
  process.exit(-1);
}

function print (msg) {
  console.log(msg);
}


var a = nums[0], b = nums[1], c = nums[2];
var A = 'Fizz', B = 'Buzz', C = 'Whizz';

function isIn (i, n, s) {
  return (i.toString().indexOf(n) !== -1)?s:'';
}

function isTimes (i, n, s) {
  return (i % n === 0) ? s : '';
}

function numberOff(currNum){
  return isIn(currNum,a,A) || isTimes(currNum, a, A) + isTimes(currNum, b, B) + isTimes(currNum, c, C) || currNum;
}

// 计算
for (var i = 1; i <= 100; i++) {
  print(numberOff(i));
}             
```