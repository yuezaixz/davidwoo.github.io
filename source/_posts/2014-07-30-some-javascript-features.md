---
layout: post
title: 'Javascript的一些特性'
date: 2014-07-30 01:57
comments: true
categories: [javascript, javascript, prototype, 读书笔记]
---
**参考资料《node.js开发指南》，作者:Byoid**

Javascript的一些特性
--------------------------
#1 作用域
如果习惯Java、C的编程语言，很容易对Javascript的作用域无法理解，比如下面代码，就会输出Hello David，因为JavaScript 的作用域完全是由函数来决定的，if、for 语句中的花括号不是独立的作用域。
```javascript
if (true) {
var name = 'David'
}
console.log('Hello '+name) // 输出 Hello David
```
##1.1 函数作用域
Javascript的函数作用域，又是局部作用域。其他变成语言基本是通过花刮号或者缩进代码块来划定作用域，而Javascript的作用域通过函数来定义。当在一个函数中用到了某个变量的引用，那么会现在当前函数作用域中找，然后再去全局作用域中找。
```
var name = 'David';
var f1 = function() {
	console.log(name); // 输出 David
};
f1();
var f2 = function() {
	var name = 'TangY';
	console.log(name); // 输出 Tangy
};
f2();	
```
###1.1.1 嵌套函数作用域
函数作用域可是可以嵌套的，和Java的局部作用域不一样，Java中不允许局部与父局部作用域有相同的变量(可以同对象作用域重复，对象作用域用this.var来调用)。
```javascript
(function test(){
	var name = "David"
	console.log("Function 1:"+name);//Function 1:David
	(function(){
		var name = "TangY"
		console.log("Function 2:"+name);//Function 2:TangY
	})();
})();
```
###1.1.2 作用域的搜索方式
####1.1.2.1 屏蔽作用域
函数的作用域会先从最近的函数作用域开始，搜索的具体逻辑可以参考如下代码。执行到变量引用的地方，变量还未定义，那变量是取局部，还是取上一次函数的作用域中的变量呢？执行代码后可发现，输出undefined，这是因为，JavaScript 会先搜索里层函数的作用域，恰巧在里层的作用域里面搜索到 scope 变量，所以上层作用域中定义的 scope 就被屏蔽了，但执行到 console.log 语句时，scope 还没被定义，或者说初始化，所以得到的就是 undefined 值了。
```javascript
(function test(){
	var name = "David"
	console.log("Function 1:"+name);//Function 1:David
	(function(){
		console.log("Function 3:"+name);//Function 3:undefined
		var name = "Haha"
	})();
})();
```
####1.1.2.2 定义生效还是调用生效？
如下代码，为什么最后那个也是打印globalScope呢？因为函数的作用域是取定义时候的，而不是调用时候的。
```javascript
var scope = 'globalScope';
var f1 = function() {
	console.log(scope);
};
f1(); // 输出 'globalScope'
var f2 = function() {
	var scope = 'f2Scope';
	f1();
};
f2(); // 输出 'globalScope'
```
##1.2 全局作用域
在 JavaScript 中有一种特殊的对象称为 全局对象。这个对象在Node.js 对应的是 global对象，在浏览器中对应的是 window 对象。
满足以下条件的变量属于全局作用域：

* 在最外层定义的变量；
* 全局对象的属性；
* 任何地方隐式定义的变量（未定义直接赋值的变量，就是没通过var定义，尽可能避免这种情况）。

```javascript
function test(){
	name = 'David'
}
test()//这里如果不执行，就会保存因为隐式定义的代码未运行
console.log(name);//David
```
#2 闭包
对未接触过函数编程的人，闭包一直都是一个很神秘的东西，但是接触以后你会发现，其实闭包也很容易，但是也会发现，闭包是如此的神奇好用。
闭包的严格定义是“由函数（环境）及其封闭的自由变量组成的集合体。”这个定义对于大家来说有些晦涩难懂，所以让我们先通过例子和不那么严格的解释来说明什么是闭包。
##2.1 What
在javascript中，每个其实都是一个闭包，只是嵌套的函数比较容易体现闭包的特性。按照前面讲解的作用域，函数中的变量引用，应该取的是其定义时候作用域的值，那么， Counter返回的函数在外部调用的时候，count变量应该已经失效了，按Java的理解来说，这里应该会报错（就算Java中用匿名内部类，也需要定义成final）。这里非但没有出错，反而每次调用 counter() 时还修改并返回了 count。这是怎么回事呢？
这正是所谓闭包的特性。当一个函数返回它内部定义的一个函数时，就产生了一个闭包，闭包不但包括被返回的函数，还包括这个函数的定义环境。上面例子中，当函数Counter () 的内部函数被一个外部变量 counter 引用时，counter 和Counter () 的局部变量就是一个闭包。
```javascript
function Counter(){
	var count = 1;
	return function(){
		return count++;
	}
}
var counter = Counter();
console.log(counter());//1
console.log(counter());//2
console.log(counter());//3
console.log(counter());//4
console.log(counter());//5
```
##2.2 Usage
###2.2.1 回调函数嵌套
这里举前端使用jQuery开发最常用的方法举例。这里定义的时候，不会立即执行，只是传入了一个回调函数（闭包），当点击事件发生的时候，才会去调用该回调函数。因为前端嵌套的层度不够，在node.js的mongoDb中，会嵌套N层的回调。
这样做还有一个好处就是隐藏内部逻辑，使用方根部不需要知道onclick这个事件怎么去监听。
```javascript
$('#div_id').click(function(){
//点击操作
});
```
###2.2.2 访问器
闭包还可以作为一个访问器，比如前面的Counter的例子，只有调用 counter() 才能访问到闭包内的 count 变量，并按照规则对其增加1，除此之外决无可能用其他方式找到 count 变量。
在javascript中是没有私有变量这概念的，不像java中有private，在javascript中只有个约定“_”开始的为私有变量，比如_name，这样很容易被修改了属性，比较不安全。
#3 对象
Java里的对象全是基于类的，而javascript不同，他都是基于原型的。如下代码可以简单的创建一个对象。
```
var man = {}
man.name = "David"
man.test = function(){
	console.log(this.name);//David
}

man.test.testFun = function(){
	console.log(this.name);//无输出
}
man.test();
man.test.testFun();
```

##3.1 构造函数
以上代码构造对象都是一次性的，如何像Java那样用固定的构造方法和参数去构造呢？以下代码为创建复杂对象的实例。
```
function User(name, age) {
	this.name = name;
	this.age = age;
	this.sayHi = function() {
		console.log("Hello World,My Name is "+this.name+",I'm "+age+" years old!")
	}
}
var somebody = new User('david','25')
somebody.sayHi()//Hello World,My Name is david,I'm 25 years old!
```
##3.2 上下文对象
这个和Java中类似，this就是在对象内部调用他本身的指针。
把上面的例子改造了下，因为User中定义的sayHi方法的name是调用对象内部的上下文，而age则是调用原型内部的局部变量。因此，在somebody中的this指针指向的是somebody对象，而tangy对象中指向的是tangy对象。而age则是闭包原理，变量的引用的作用域为定义时候的作用域，因为构造的时候才真正定义，所以age就是new User()时候的变量了。
最后一个函数，由于函数是全局的，所以this指向的是全局变量。
```javascript
function User(name, age) {
	this.name = name;
	this.age = age;
	this.sayHi = function() {
		console.log("Hello World,My Name is "+this.name+",I'm "+age+" years old!")
	}
}
var somebody = new User('david','25')
somebody.sayHi()//Hello World,My Name is david,I'm 25 years old!
var tangy = {
	sayHi:somebody.sayHi,
	name:'tangy',
	age:23
}
tangy.sayHi()//Hello World,My Name is tangy,I'm 25 years old!

name = "global"
sayHi = tangy.sayHi//Hello World,My Name is global,I'm 25 years old!
sayHi() 
```

##3.3 call与apply
这两个函数挺神奇的，其作用有点像Java中的反射（其实是Java通过反射来模仿call与apply的操作），call 和 apply 的功能是一致的，两者细微的差别在于 call 以参数表来接受被调用函数的参数，而 apply 以数组来接受被调用函数的参数。call 和 apply 的语法分别是：
func.call(thisArg[, arg1[, arg2[, ...]]])
func.apply(thisArg[, argsArray])
用Java来帮助理解的话，第一个参数就是Java的Method对象的invoke方法的第一个参数，也就是代理对象。
如果不用Java描述，那理解就是func 是函数的引用（javascript中函数是第一等公民），thisArg 是 func 被调用时的上下文对象，arg1、arg2 或argsArray 是传入 func 的参数，看一下例子，可以看出这两个函数的作用。
```javascript
function User(name) {
	this.name = name;
	this.sayHi = function(age,favorite) {
		console.log("Hello World,My Name is "+this.name+",I'm "+age+" years old! I like "+favorite+"!")
	}
}
name = 'global'
var david = new User('David')
david.sayHi(25,'basketball and football')
//Hello World,My Name is David,I'm 25 years old! I like basketball and football!
var tangy = {
	name:'tangy'
}
david.sayHi.call(tangy,23,'shopping and movie')//Hello World,My Name is tangy,I'm 23 years old! I like shopping and movie!
var lily = {
	name:'lily'
}
david.sayHi.apply(lily,[46,'travel'])//Hello World,My Name is lily,I'm 46 years old! I like travel!
```
##3.4 bind函数
但是，如果重复使用，每次都要传递对象作为函数的上下文，那么多不方便。这时候就可以使用bind函数。语法为：
func.bind(thisArg[, arg1[, arg2[, ...]]])
参数与call一致。实例代码如下，最后一个输出，出现了重复绑定，这时候，bind已经无效了。如果要解释，就是第一次bind的时候，对象内部还是有this这个指针的，第二次bind，那么这个函数已经类似function(){func.call(david,25)},这时候已经不存在this，所以bind是无效的。这里可能有点难理解，无法理解的话就记住，bind不能重新绑定。
```
function User(name) {
	this.name = name;
	this.sayHi = function(age,favorite) {
		console.log("Hello World,My Name is "+this.name+",I'm "+age+" years old! I like "+favorite+"!")
	}
}
var david = new User('David')
name = 'global'
david.sayHi(25,'basketball and football')
//Hello World,My Name is David,I'm 25 years old! I like basketball and football!
var tangy = {
	name:'tangy'
}
tangy.sayHi = david.sayHi
tangy.sayHi(23,'shopping and movie')//Hello World,My Name is tangy,I'm 23 years old! I like shopping and movie!
tangy.sayHi2 = david.sayHi.bind(david,25,'make a joke')
tangy.sayHi2()//Hello World,My Name is David,I'm 25 years old! I like make a joke!
tangy.sayHi3 = david.sayHi.bind(david,25)
tangy.sayHi3('reading')//Hello World,My Name is David,I'm 25 years old! I like reading!
var lily = {
	name:'lily'
}
lily.sayHi = tangy.sayHi3.bind(lily)
lily.sayHi(46,'travel')//Hello World,My Name is David,I'm 25 years old! I like 46!
```
##3.5 原型
这是javascript面向对象的核心，javascript的对象都是基于原型的。这就相当于一件艺术品，一件蒙娜丽莎的微笑，javascript就是匠心独运的画匠，他100%的复制了蒙娜丽莎，并且可以通过自己的创意和灵感，赋予她新的东西。下面是一个简单的例子：
```javascript
function Person() {
	this.func = function(){
		console.log(1);
	}
}
Person.prototype.name = 'David';
Person.prototype.showName = function () {
	console.log('My name is '+this.name)
}
var person = new Person()
person.showName()//My name is David
person.name = 'tangy'
person.showName()
var person2 = new Person()
console.log(person.func == person2.func);//false
console.log(person.showName == person2.showName);//true
```
上面这段代码使用了原型而不是构造函数初始化对象。这样做与直接在构造函数内定义属性有什么不同呢？可以从最后看出，原型中的函数是同一个应用，而构造函数每次都会重复创建，由于闭包的局部变量上下文，所以有额外开销，因此可以得出下面结论：

* 构造函数内定义的属性继承方式与原型不同，子对象需要显式调用父对象才能继承构造函数内定义的属性；
* 构造函数内定义的任何属性，包括函数在内都会被重复创建，同一个构造函数产生的两个对象不共享实例；
* 构造函数内定义的函数有运行时闭包的开销，因为构造函数内的局部变量对其中定义的函数来说也是可见的。

尽管如此，并不是说在构造函数内创建属性不好，而是两者各有适合的范围。那么我们什么时候使用原型，什么时候使用构造函数内定义来创建属性呢？

* 除非必须用构造函数闭包，否则尽量用原型定义成员函数，因为这样可以减少开销；
* 尽量在构造函数内定义一般成员，尤其是对象或数组，因为用原型定义的成员是多个实例共享的。

##3.6 原型链
Object 与 Function是两个特殊对象，它们都是构造函数，用于生成对象，参加如下代码。Object.prototype 是所有对象的祖先，Function.prototype 是所有函数的原型，包括构造函数。
```javascript
Function.prototype.testfunc = function(){
	console.log('this is a test func!');
}
var o = new Object();
var f = new Function();
function func(){

}
console.log(typeof o)
console.log(typeof f)
console.log(typeof func)
console.log(f.testFunc == func.testFunc)
```
###3.6.1 对象分类
javascript中的对象有三类，一类是用户创建的对象，一类是构造函数对象，一类是原型对象。用户创建的对象，即一般意义上用 new 语句显式构造的对象。构造函数对象指的是普通的构造函数（第一类是该函数构造的对象，这里就是这构造函数），即通过 new 调用生成普通对象的函数。原型对象特指构造函数 prototype 属性指向的对象。
```javascript
function constructor(){

}
var object = new constructor()
console.log(typeof constructor);//function
console.log(typeof object);//object
console.log(constructor.prototype);//{}
console.log(object.prototype);//undefined
console.log(object.__proto__);//{}
console.log(object.__proto__ === constructor.prototype);//true
```
###3.6.2 __proto__属性
上面的例子也稍微提到了__proto__属性，可以看出，他指向该对象的原型，从任何对象沿着它开始遍历都可以追溯到 Object.prototype。
构造函数对象有 prototype 属性，指向一个原型对象，通过该构造函数创建对象时，被创建对象的 __proto__ 属性将会指向构造函数的 prototype 属性。原型对象有 constructor属性，指向它对应的构造函数。
```
function Constructor(){

}
Object.prototype.name = "ObjectPrototypeAttr"
Constructor.prototype.name = "ConstructorPrototypeAttr"
var constructor = new Constructor()
var object = new Object()
console.log(constructor.name);//ConstructorPrototypeAttr
console.log(object.name);//ObjectPrototypeAttr
console.log(constructor.__proto__.name);//ConstructorPrototypeAttr
console.log(constructor.__proto__.__proto__.name);//ObjectPrototypeAttr
console.log(constructor.__proto__.constructor.prototype.name);//ConstructorPrototypeAttr
```
##3.7 对象复制
###3.7.1 浅复制
浅复制只是重新创建了一个指针（同类型变量的引用），实际上引用还是相同的对象。浅复制实现如下代码。
```javascript
Object.prototype.clone = function() {
var newObj = {};
for (var i in this) {
newObj[i] = this[i];
}
return newObj;
}
var obj = {
name: 'David',
likes: ['basketball']
};
var newObj = obj.clone();
obj.likes.push('football');
console.log(obj.likes); //[ 'basketball', 'football' ]
console.log(newObj.likes); // [ 'basketball', 'football' ]
```
###3.7.2 深复制
深复制比较复杂，因为每个对象的内部逻辑都不一样，需要循环递归，而且对于函数和数组需要做特殊处理。
```
Object.prototype.clone = function() {
	var newObj = {};
	for (var i in this) {
		if (typeof(this[i]) == 'object' || typeof(this[i]) == 'function') {
			newObj[i] = this[i].clone();
		} else {
			newObj[i] = this[i];
		}
	}
	return newObj;
};
Array.prototype.clone = function() {
	var newArray = [];
	for (var i = 0; i < this.length; i++) {
		if (typeof(this[i]) == 'object' || typeof(this[i]) == 'function') {
			newArray[i] = this[i].clone();
		} else {
			newArray[i] = this[i];
		}
	}
	return newArray;
};
Function.prototype.clone = function() {
	var that = this;
	var newFunc = function() {
		return that.apply(this, arguments);
	};
	for (var i in this) {
		newFunc[i] = this[i];
	}
	return newFunc;
};
var obj = {
	name: 'david',
	likes: ['basketball'],
	display: function() {
	console.log(this.name);
},
};
var newObj = obj.clone();
newObj.likes.push('football');
console.log(obj.likes); // [ 'basketball' ]
console.log(newObj.likes); // [ 'basketball', 'football' ]
console.log(newObj.display == obj.display); //false
```
###3.7.3 深复制的问题
如上代码其实存在一个问题，如果存在循环引用就会报错（如下代码加到上面那个代码）。要实现真正的深复制，那么就需要分析对象的依赖关系，像Java中的Spring初始化那样。
```javascript
var obj1 = {
ref: null
};
var obj2 = {
ref: obj1
};
obj1.ref = obj2;
obj1.clone();//RangeError: Maximum call stack size exceeded
```
