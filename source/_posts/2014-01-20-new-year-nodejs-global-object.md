---
layout: post
title: '辞旧迎新---javascript的全局对象'
date: 2014-01-20 12:40
comments: true
categories: [javascript, javascript, Node.js, 互联网, 互联网思维, 去中心化, 全局对象]
---
上周六，公司的尾牙终于结束了。由于公司的尾牙的排练，也落下了不上工作，本周也是当前项目的关键一周，还有很多工作要做，因此这时候想撇开过去说辞旧迎新还是有点早的。但是，我就是一个标题党，我今天要说的辞旧迎新和你们想的肯定完全不一样，谁能猜到我要说的是什么呢？我要说的其实就是，工业时代的过去，互联网时代的到来。
自蒸汽机以来250年间，工业思维一直主导着我们的社会，“中心化”、“集中化”、“标准化”、“控制化”等工业品生产领域的概念不仅成为社会管理的方法，也成为商业运作的准则。但是，互联网恰恰相反，是去中心的、异质的、多元的和感性的，它天然就有民主的特性，不巧地解构掉了工业思维。
互联网的本质不是用一个更好的管理去打败原来的管理，也不是用创新去打败原来的创新，更不是用更好的技术去打败原来的技术。而是用人类本能的兴趣爱好，而是以一种工作坊的工作方式，去打败以前的大工业时代。互联网是用了很多去中心化的结点效应，产生了集体智慧，群体智慧。
也许有人不理解这种去中心化的结点效应是什么？那仔细听了，让我来告诉你。其实嘛，我也不懂，我最多算大概懂个意思。其实我们也已经在这种去中心化的结点效应中生活了好久了，只是没人告诉你这么回事，也就不以为然了。比如我现在写博文就是这样的意思，为整个社区的协同创作做贡献。任何参与者，均可提交内容，网民共同进行内容协同创作或贡献。
而twitter则把这个事情做到了一个极致。也许写博文还需要的时间比较多，那发个twitter就简单多了，可以简单的就是一个图片，或者简单的就是一个字。使得为互联网生产或贡献内容更加简便、更加多元化，从而提升了网民参与贡献的积极性、降低了生产内容的门槛。最终使得每一个网民均成为了一个微小且独立的信息提供商，使得互联网更加扁平、内容生产更加多元化。碎片化的信息同样也符合快节奏现代人的阅读习惯。

先就写这些，最近几篇估计都会乱侃互联网时代的改变。
-----------------------分割线-------------------------
# 全局对象
全局对象在javascript中比较特殊，在程序中随时都能访问，类似Java中的静态变量，随时都能 ClassName.STATIC_VAR_NAME 来访问。熟悉客服端JS的应该都知道window，他就是全局对象，而javascript中的全局对象是global，所有的全局变量都是global的属性。
在javascript中可以直接访问全局对象的属性，比如在客户端javascript中我们访问最多的就是document变量。而javascript中，console、process是访问较多的全局变量。
##1 定义全局变量
满足以下条件则为全局变量：

* 在最外层定义的变量；
* 全局对象的属性；
* 隐式定义的变量（未定义直接赋值的变量）。

注意：一定要避免最后一种情况，即未使用 var 定义变量。因为全局变量会污染命名空间，提高代码的耦合风险。
##2 常用的全局变量
###2.1 Process
process 是一个全局变量，即 global 对象的属性。它用于描述当前 Node.js 进程状态的对象，提供了一个与操作系统的简单接口。
下面将会介绍 process 对象的一些最常用的成员方法。
####2.1.1 process.argv
process.argv是命令行参数数组，第一个元素是 node，第二个元素是脚本文件名，从第三个元素开始每个元素是一个运行参数。
看以下例子：
```javascript argv.js
console.log(process.argv);
```
在控制台执行的结果为：
```
[David@localhost javascriptStudy]$ node argv.js 19881114 name=David -V XMUT 
[ 'node',
  '/home/David/workspace/javascriptStudy/argv.js',
  '19881114',
  'name=David',
  '-V',
  'XMUT' ]
```
####2.1.2 process.stdout
标准输出流，通常我们使用的 console.log() 向标准输出打印字符，而 process.stdout.write() 函数提供了更底层的接口。下面这个例子我直接在控制台输出了。
``` Console
[David@localhost javascriptStudy]$ node -e 'process.stdout.write("aaa")'
aaa
```
####2.1.3 process.stdin
标准输入流，初始时它是被暂停的，要想从标准输入读取数据，你必须恢复流，并手动编写流的事件响应函数。
```javascript testStdin.js
process.stdin.resume();
process.stdin.setEncoding('utf8');
process.stdin.on('data', function(data) {
process.stdout.write('read from console: ' + data.toString());
if(data.toString().substring(0,3) == 'bye'){
    process.stdin.emit('end');                                                  
}
});
process.stdin.on('end', function () {
  process.stdout.write('end!');
});

```

``` Console
[David@localhost javascriptStudy]$ node testStdin.js 
我是David!
read from console: 我是David!
bye
read from console: bye
end!
```

还有一种方式就是把输入写到文件中重定向，比如我创建一个文件叫in，里面内容是“我是吴迪玮，我是中国人！”。然后执行下面命令。In中的内容会重定向输出到out中去。

``` Console
[David@localhost javascriptStudy]$ cat in
我是吴迪玮，是个中国人！
[David@localhost javascriptStudy]$ cat out
cat: out: 没有那个文件或目录
[David@localhost javascriptStudy]$ node testStdin.js < in out
[David@localhost javascriptStudy]$ cat out
read from console: 我是吴迪玮，是个中国人！
```
####2.1.4 process.nextTick(callback)
process.nextTick(callback) 的功能是为事件循环设置一项任务，Node.js 会在下次事件循环调响应时调用 callback。
先解释下下次时间循环响应时调用是什么意思。
任何javascript应用都是一个单线程，任何时刻都只有一个事件在执行，因此javascript应用应该尽可能缩短事件的执行时间。process.nextTick(callback)提供了一个这样的工具，可以吧复杂的工作拆散，变成一个一个较小的时间。
```javascript
function doSomething(args, callback) {
somethingComplicated(args);
process.nextTick(callback);
}
doSomething(function onEnd() {
compute();
});
```
上面的程序中，somethingComplicated和compute都执行时间较长，因此，后面的process.nextTick(callback);会把耗时的操作拆分，减少每个时间的执行时间，提高响应速度。
你也可以使用setTimeout()函数来达到貌似同样的执行效果： 
```javascript
function doSomething(args, callback) {
somethingComplicated(args);
setTimeout(callback, 0);
}
doSomething(function onEnd() {
compute();
});
```
但是 **注意** ：不要使用setTimeout（fn,0）来代替，nextTick效率高的多。
#####2.1.4.1 process.nextTick()和setTimeout(fn, 0)的不同
在内部的处理机制上，process.nextTick()和setTimeout(fn, 0)是不同的，process.nextTick()不是一个单纯的延时，他有更多的[特性](https://gist.github.com/mmalecki/1257394)。
更精确的说，process.nextTick()定义的调用会创建一个新的子堆栈。在当前的栈里，你可以执行任意多的操作。但一旦调用netxTick，函数就必须返回到父堆栈。然后事件轮询机制又重新等待处理新的事件，如果发现nextTick的调用，就会创建一个新的栈。 
#####2.1.4.2 nextTick()的具体用途
可以参考该文，戳[这里]( http://www.oschina.net/translate/understanding-process-next-tick?cmp)

####2.1.5 其他
除此之外process还展示了process.platform、process.pid、process.execPath、process.memoryUsage() 等方法，以及 POSIX进程信号响应机制。有兴趣的读者可以查看[API]( http://javascript.org/api/process.html)了解详细内容。
###2.2 Console
####2.2.1  console.log()
两个简单例子带过。
``` Console
[David@localhost javascriptStudy]$ node -e 'console.log("abcd%shijk","efg")'
abcdefghijk
[David@localhost javascriptStudy]$ node -e 'console.log("abcd%dhijk","1234")'
abcd1234hijk
```
####.2.2.2 console.error()
标准错误流输出，用法同上。
####2.2.3 console.trace()
向标准错误流输出当前调用栈。
``` Console
[David@localhost javascriptStudy]$ node -e 'console.trace()'
Trace
    at Object.eval (eval at <anonymous(eval:1:82))
    at Object.<anonymous(eval:1:70)
    at Module._compile (module.js:449:26)
    at evalScript (node.js:358:25)
    at startup (node.js:77:7)
    at node.js:699:3
```
