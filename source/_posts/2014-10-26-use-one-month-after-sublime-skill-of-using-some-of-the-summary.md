---
layout: post
title: '使用Sublime一个月后的一些使用技巧总结'
date: 2014-10-26 01:45
comments: true
categories: [Sublime, 工具, 编辑器, 提升工作效率]
---
#废话
这个月开始从VIM转向使用Sublime，可能很多Sublime脑残粉会说，这工具你现在才开始用啊，也可能很多Vim脑残粉会说，Vim是最高级的编辑器，你竟然自我退化。确实Vim太高级了，我用不好，Sublime至少一个月内让我把IntelliJ的前端功能都摒弃了（以前html、css、JavaScript基本用他操作）。
#Feature
那就来说说我感受到的他的优点
##强大的命令模式
在编辑窗口按ctrl+shift+p就可以进入命令模式，命令模式即为在编辑窗口上方弹出了一个输入框，可以输入Sublime内置和插件所扩展的命令功能，而且支持模糊匹配。比如：

* set synax javascript 命令可以设置当前语法为javascript
* 安装Package Control后可以用Package Install命令来安装插件
* minimap 可以关闭打开编辑器迷你图
* 等等等等

##强大的Goto功能
在编辑窗口输入ctrl+p就可以进入goto命令，与命令模式一样，在编辑窗口上方会弹出了一个输入框，可以输入goto的命令，可以是文件跳转，也可以是文件跳转行，也可以跳转到某个元素（函数、选择器等），也可以在页面中查找。

* @可以快速定位元素，比如js的函数，CSS文件的选择器，支持模糊匹配
* #可以定位到页面内的内容，支持模糊匹配
* ：+行数 跳转到行

##模糊匹配功能
我认为的模糊分两种，一种是模糊缩写自动补全，一种是搜索的时候的模糊匹配，如下：

* 模糊补全：在CSS文件中输入bgc按tab键就是自动键入background-color
* 模糊匹配：比如在命令模式输入ssjs，就会模糊匹配上 **s**et **s**ynax **j**ava **s**cript，粗体的字符为模糊匹配的字段；

##强大的插件功能
基本上所有的语法都能找到很好的插件，比如对于Html就有Emmet，他极大的提升了前端开发的工作效率，这里就不一一细说该插件，有兴趣的可以Google下，这里稍微举个例子来看看。
ctrl+n创建个页面后，输入 !+[tab]，就可以自动生成html5的标准文档，如果需要HTML4的标准，就输入html:4s+[tab]。之后会自动生成如下代码。
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
</head>
<body>
	
</body>
</html>
```
然后在任意位置输入元素名字后按tab都能生成相应名字的元素，比如输入script+[tab]，就会生成script标签。如果输入div.className就会生成```<div class="className"></div>``` ，是不是很神奇。更神奇的是，输入ul>.item$*5+[tab]（ *这里注意，>代表子元素，+代表并列。子元素的情况下默认子元素为div，但是ul的默认为li，tr的默认为td等* ），会自动生成以下代码：

```html
<ul>
        <li class="item1"></li>
        <li class="item2"></li>
        <li class="item3"></li>
        <li class="item4"></li>
        <li class="item5"></li>
</ul>

```
##多行编辑功能
就在上列html代码中，如果要同时编辑li元素中的内容，那么正常情况需要先编辑完一个后复制粘贴4次。Subilme的多行编辑功能很好的帮我们简化了这个操作（编码+鼠标+键盘+再鼠标+再键盘……），提升了我们的编码速率。
进入多行编辑模式有几种方式，如下：

* 选中><按alt+f3可以多行光标模式（这里输入的内容为需要匹配的共同点），这样做的坏处是共同点与其他页面元素相同时候不方便匹配，比较不灵活
* 争对上面的问题就有另一个办法，可以把鼠标放在><之间按住shift用右键往下拖来就可以进行多行编辑，这样做的坏处是，如果需要多行编辑的元素不在一个垂直线并且不是末尾元素，将无法进行
* 再争对以上问题就又有一个办法，可以选中><后用ctrl+d 和ctrl+k来选择跳过选中，这样的缺点就是太慢，尽可能还是在相同点按alt+f3吧

多行选择后可以p{This is content}+[tab]来生成以下代码
```html
		<ul>
        <li class="item1 "><p>This is content</p></li>
        <li class="item2 "><p>This is content</p></li>
        <li class="item3 "><p>This is content</p></li>
        <li class="item4 "><p>This is content</p></li>
        <li class="item5 "><p>This is content</p></li>
    </ul>
```

##其他快捷键
其中粗体的内容为我认为可以大幅度增加工作效率的功能，而且其他浏览器可能没有的（不是绝对没有，请勿喷）。
ctrl+tab切换正在编辑的文件（ctrl+shift+tab可以相反方向）
ctrl+H当前文件查找替换模式（类型其他编辑器的替换查找）
ctrl+f所有文件查找内容(使用完后按esc退出，之前一直不知道这个。。。)
ctrl+shift+f所有文件替换内容
ctrl+shift+d 复制当前或者复制选中内容
ctrl+shift+k 删除当前行
alt+. 闭合标签
**shift+f11** 进入无干扰模式
**ctrl+enter** 在下一行插入并移动光标到那
**ctrl+shift+enter** 在上一行插入并移动光标到那
**ctrl+shift+v** 带自动缩进的粘贴（正常复制后格式会乱，这种方式粘贴后省略了人工排版的问题）

#总结
好用，谁用谁知道！