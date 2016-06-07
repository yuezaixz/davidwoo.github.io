---
layout: post
title: 'vert.x,JVM上的Node.js解决方案'
date: 2014-01-13 14:23
comments: true
categories: [javascript, jvm, vertx]
---
最近忙的已经要混乱了。项目赶的死，主持人排练赶的死，各种红包炸弹赶的死，女朋友也还是要陪的，恨不出变出几个我来帮我去做这些事情，我就躺在被窝睡觉好了。今天的博文就分享下我用坐班车上班的时间看的一篇文章吧，说的主要是Node.js在JVM上的替代者，原文请戳这里[vert.x——JVM上的Node.js替代者](http://www.infoq.com/cn/news/2012/05/vertx).
**PS.本文斜体部分为引用原文。**
####其他参考资料:
[vert.x官网](http://vertx.io/)
[维基](http://en.wikipedia.org/wiki/Vert.x)
#什么是Vert.x
##官网解释
Vert.x is a lightweight, high performance application platform for the JVM that's designed for modern mobile, web, and enterprise applications.
##渣翻译
Vert.x是一个为智能手机、WEB、企业级应用设计的运行于JVM的轻量级高性能的应用平台。
##原文解释
Vert.x是一个用于下一代异步、可伸缩、并发应用的框架，旨在为JVM提供一个Node.js的替代方案。开发者可以通过它使用JavaScript、Ruby、Groovy、Java、甚至是混合语言来编写应用。
#相比Node.js
首先，他的开发流程与node.js是非常类似的。但是，vert.x的性能与可伸缩性都远远超越了node.js。同样是基于EDA事件驱动编程，Vert.x.竟然比Node.js快。[Inside Vert.x. Comparison with Node.js. | Architects Zone](http://architects.dzone.com/articles/inside-vertx-comparison-javascript)一文测试如下：
![](http://www.cubrid.org/files/attach/images/220547/772/629/vertx_javascript_performance_comparison_200_ok.png)
#为什么Vert.x如此快
##事件驱动编程模型
和Node.js一样，Vert.x提供一个事件驱动编程模型，使用Vert.x作为服务器时，程序员只要编写事件处理器event handler即可。当TCP socket有数据时，event handler立即被创建调用，另外它还可以在以下几种情况激活： '当事件总线Event Bus接受到消息时,' '当接收到HTTP消息时,' 当一个连接断开时',' '当计时器超时时.'
##事件循环
*一个vert.x实例会管理着一个小的线程集合，每个线程针对服务器上的一个可用内核。基本上每个线程都实现了一个事件循环。当部署一个vert.x应用实例（又叫做verticle）时，服务器会选择一个事件循环分配给该实例。接下来针对该实例的任务都会通过该线程进行分配。由于在某一时刻可能会有成千上万个verticle在运行，因此在同一时刻会将单个事件循环指定给多个verticle。*
Vert.x 内部有一个线程池. Vert.x会根据CPU核数匹配线程池的数目。
每个线程执行一个Event Loop。Event Loop能确保事件在循环中轮回。例如，它会确证socket是否有数据可读事件，如果有， Vert.x 将调用相应的event handler。

##消息传递
*Verticle可与运行在相同或不同vert.x实例中的其他verticle进行通信，这是通过消息事件总线实现的，它类似于Erlang的actor模型。消息传递旨在让系统能够在多个可用核心上进行扩展而无需以多线程的方式来执行verticle代码。
事件总线是分布式的，并不只会跨越服务器，还会渗透进客户端的JavaScript以处理“实时”的Web应用。*
一个Vert.x中有很多Verticles， Verticles之间使用事件总线Event Bus联系. Verticle类似Erlang/scala中的actor模型（actor模型大概解释就是一个用于消息传递和共享数据的模型，具体我也不是很懂）。
##共享数据
消息传递当然有用，但是它并不总是并发环境最好的，缓存受到普遍使用， Vert.x提供一个分享的缓存Map。而node.js中则需要通过消息传递来在进程间交换数据。
#其他特性

* TCP/SSL服务器与客户端
* HTTP/HTTPS服务器与客户端
* WebSockets/SockJS支持

#代码
如下代码展示了Web服务器是如何通过vert.x来处理静态文件的：
```
// JavaScript
load('vertx.js')
vertx.createHttpServer().requestHandler(function(req) {
  var file = req.path === '/' ? 'index.html' : req.path;
  req.response.sendFile('webroot/' + file);
}).listen(8080)

# Ruby
require "vertx"
Vertx::HttpServer.new.request_handler do |req|
  file = req.uri == "/" ? "index.html" : req.uri
  req.response.send_file "webroot/#{file}"
end.listen(8080)

// Groovy
vertx.createHttpServer().requestHandler { req ->
  def file = req.uri == "/" ? "index.html" : req.uri
  req.response.sendFile "webroot/$file"
}.listen(8080)

// Java
import org.vertx.java.core.Handler;
import org.vertx.java.core.http.HttpServerRequest;
import org.vertx.java.deploy.Verticle;
public class Server extends Verticle {
  public void start() {
    vertx.createHttpServer().requestHandler(new Handler() {
      public void handle(HttpServerRequest req) {
        String file = req.path.equals("/") ? "index.html" : req.path;
        req.response.sendFile("webroot/" + file);
      }
    }).listen(8080);
  }
}
```

#再见
先写到这里吧，困了睡觉去了。最近几天有空就写写博文吧。Z.Z zzzZZZ
