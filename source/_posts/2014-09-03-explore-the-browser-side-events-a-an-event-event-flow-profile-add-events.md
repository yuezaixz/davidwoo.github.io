---
layout: post
title: '浏览器端的事件探秘（一）---事件事件流简介、事件添加方式'
date: 2014-09-03 13:17
comments: true
categories: [javascript, 事件流, 事件, 事件冒泡, 添加事件方式]
---
Javascript的事件
------------------------------
#1 事件冒泡
##1.1 事件
在IE4以后，浏览器的开发团队就开始给页面的每个部分都设定了特定的事件。多到鼠标点击，简单到鼠标滑过，甚至敲击键盘都会有特定的事件。
##1.2 事件流
如下图，点击页面的一部分DIV3的时候，那么浏览器就认为你点击这按钮的同时，也点击了这个按钮所在的容器。这个过程就是一个事件流。
![](http://chuantu.biz/t/50/1409750064x1822611033.jpg) 
事件描述的是从页面接受事件的顺序。IE的事件流方式是事件冒泡流，网景公司用的是事件捕获流。
###1.2.1 事件冒泡
即事件最开始有最具体的元素（文档中的嵌套层次最深的元素）接收，然后逐级上传到最不具体的元素（文档）。
比如下面这个程序，当你点按钮的时候，也会认为你点了div，然后在认为你点了body，然后又认为你点了html，然后在认为你点了document。
```html
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
    <title>事件流</title>
    <script type="text/javascript">
        function showMsg(txt){
            alert(txt)
        }

    </script>
</head>
<body>
    <div id="box">
        <input type="button" value="按钮" id="btn" onclick="showMsg('按钮')" />
    </div>
</body>
</html
```
###1.2.2 事件捕获
不太具体的应该更早的接受到事件，具体的应该是最后接受到的。上面例子，document就是最先接收，最后是按钮。
##1.3 事件添加方法
###1.3.1 html事件
上述代码就是一个html事件，html事件会导致html代码和javascript代码连接过紧，如果修改javascript代码的函数名，那么也要同学修改html。
###1.3.2 DOM0级事件处理程序
好处：简单，跨浏览器。
```html
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
    <title>事件流</title>
    <script type="text/javascript">
        function init(){
            var btn1 = document.getElementById("btn")
            btn1.onclick=function(){
                showMsg(this.getAttribute("value"))
            }
            var btn2 = document.getElementById("btn2")
            btn2.onclick=function(){
                showMsg(this.getAttribute("value"))
                btn2.onclick = null
            }
        }
        function showMsg(txt){
            alert(txt)
        }

    </script>
</head>
<body onload="init()" >
    <div id="box">
        <input type="button" value="按钮" id="btn"/>
        <input type="button" value="按钮2" id="btn2"/>
    </div>
</body>
</html>
```
###1.3.3 DOM2级事件处理
定义了2个方法，用于处理指定和删除事件处理程序的操作。分别为addEventListener()和removeEventListener()。接受3个参数：要处理的事件名、作为事件处理程序的函数和布尔值（true是事件捕获，false是事件冒泡）。
addEventListener添加的事件只能通过removeEventListener来删除，且参数一致。
详细参考如下代码：
```javascript
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
    <title>事件流</title>
    <script type="text/javascript">
        function init(){
            var btn1 = document.getElementById("btn")
            btn1.addEventListener('click',clickBtn,false)
            var btn2 = document.getElementById("btn2")
            btn2.addEventListener('click',clickBtn,false)
            btn2.addEventListener('click',function(){
                btn2.removeEventListener('click',clickBtn,false)
            },false)
        }
        function clickBtn(){
            showMsg(this.getAttribute("value"))
        }
        function showMsg(txt){
            alert(txt)
        }

    </script>
</head>
<body onload="init()" >
    <div id="box">
        <input type="button" value="按钮" id="btn"/>
        <input type="button" value="按钮2" id="btn2"/>
    </div>
</body>
</html>
```
DOM2级还有一个好处就是可以给模块增加多个点击事件。如上btn2，就添加了多个click事件。
###1.3.4 IE事件处理程序
定义了2个方法，用于处理指定和删除事件处理程序的操作。分别为attachEvent ()和detachEvent ()。接受2个参数：要处理的事件名、作为事件处理程序的函数，事件处理方式就是冒泡。
```html
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
    <title>事件流</title>
    <script type="text/javascript">
        function init(){
            var btn1 = document.getElementById("btn")
            btn1.attachEvent('onclick',clickBtn)
            var btn2 = document.getElementById("btn2")
            btn2.attachEvent('onclick',clickBtn)
            btn2.attachEvent('onclick',function(){
                btn2.detachEvent('onclick',clickBtn)
            })
        }
        function clickBtn(){
            showMsg(this.value)
            //this.getAttribute("value")
        }
        function showMsg(txt){
            alert(txt)
        }

    </script>
</head>
<body onload="init()" >
    <div id="box">
        <input type="button" value="按钮" id="btn"/>
        <input type="button" value="按钮2" id="btn2"/>
    </div>
</body>
</html>
```
###1.3.5 跨浏览器事件处理
那就要做能力处理判断，那最好把这种处理放在一个对象中。
```html
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
    <title>事件流</title>
    <script type="text/javascript">
        function EventUtil(){
        }
        EventUtil.prototype.addHandler = function(element,type,handler){
            if(element.addEventListener){
                element.addEventListener(type,handler,false);
            }else if(element.attachEvent){
                element.attachEvent('on'+type,handler);
            }else{
                //在javascript中，所有用点的地方都能用中刮号
                element['on'+type] = handler;
            }
        }
        EventUtil.prototype.removeHandler = function(element,type,handler){
            if(element.removeEventListener){
                element.removeEventListener(type,handler,false);
            }else if(element.detachEvent){
                element.detachEvent('on'+type,handler);
            }else{
                //在javascript中，所有用点的地方都能用中刮号
                element['on'+type] = null;
            }
        }
        function init(){
            var eventUtil = new EventUtil();
            var btn1 = document.getElementById("btn")
            eventUtil.addHandler(btn1,'click',clickBtn)
            var btn2 = document.getElementById("btn2")
            eventUtil.addHandler(btn2,'click',clickBtn)
            eventUtil.addHandler(btn2,'click',function(){
                eventUtil.removeHandler(btn2,'click',clickBtn)
            })
        }
        function clickBtn(){
            if(this.value){
                showMsg(this.value)
            }else if(this.attributes){
                showMsg(this.attributes["value"])
            } else{
                showMsg("Hello World")
            }

            //this.getAttribute("value")
        }
        function showMsg(txt){
            alert(txt)
        }

    </script>
</head>
<body onload="init()" >
    <div id="box">
        <input type="button" value="按钮" id="btn"/>
        <input type="button" value="按钮2" id="btn2"/>
    </div>
</body>
</html>
```
