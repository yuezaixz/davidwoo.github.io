---
layout: post
title: '我眼里的Bezos&Amazon----Code Journey for Kata Potter'
date: 2014-05-14 13:58
comments: true
categories: [Amazon, bezos, 天道酬勤, KataPotter, CodeJourney]
---
近况
----------------------
感觉现在忙的事情越来越多了。给我家媳妇调毕业论文的日子结束了，紧接上来的就是给媳妇找房子租。然后V1.0版本差不多搞定了，立项、整体计划、下个阶段需求、设计等事情也连绵不绝。然后因为用户开始试用，V1.0版本很多的问题开始暴露，要搜集问题分析问题，紧急的还要打补丁解决，一些用户还需用定制一些特殊功能。
忙的事情实在多，只能拼凑一些晚上时间来做做Code Journey，再利用各种班车上、午间、饭后时间看书。
所以今天博文就是写最近我看的Bezos传记和Kata Potter的编程之旅，Kata Potter暂时未实现，只是有了一些思路，在设计中。

Bezos&Amazon
------------------------
#Amazon
亚马逊开始时候不过就是个网络书商，靠从出版商那里批发图书转卖挣其中的利润。什么当当网之类的不过都是亚马逊的中国制造——山寨。从普通网络书店，到地球上最大的书店，到最大的综合网络零售商，最后到今天这样一个综合性的网络服务商。
亚马逊崛起的平缓而持续，得益于Bezos。
#Bezos
Bezos的经历告诉我们一个事情，天道不一定酬勤，但是会决定你成为一个怎样的人。就像我去年考系统集成项目管理认证，两个月的高压工作下学习，虽然最后差之毫厘，但是这个过程中，**我已经得到了这样的能力与见识，我已经将自己变成了一个不一样的人。外在的评估与期待只是一个结果而已，但是绝不应该是目的**。
如果你真的早已知晓你所希望成就的一切（说到这里可能又会被认为是成功学），并努力一直将生命向这个方向推动，不懈怠，专注思考，穷尽一切可能，你一定会得到你所想要得到的。虽然至于得到多少，能不能闻达于天下，仍然取决于概率，但是这种主动至少能够帮助你战胜概率。在机会来临前，如果你能做好准备，你才有资格去丰收。我不信儒家思想中的天命，天命不过是未来的一种不确定性，会掌控和颠覆我们的生活，但是想一想，这种不确定性在人的一生这七八十年的样本区间里，就被概率抹平了，大部分的你们终会回归预期轨迹。虽然没几个人能达到Bezos的高度，但是主动至少能给我们获得成功的概率，这概率大小就取决于我们的努力有多少以及我们定义的成功有多高。


以下为Bezos2010年在普林斯顿大学的演讲，[全文](http://c.blog.sina.com.cn/profile.php?blogid=bec49e8b8900143l)
>对我来说，这真是个困难的选择（指辞去对冲基金的工作去创建亚马逊），最终，我决定放手一搏。因为，万一试了以后失败，我并不会后悔；但如果不去试试看，我可能永远都会耿耿于怀。考虑了很久，我最后选择了一条比较不安全的路，去追随我的热情。今天，对于这个选择，我感到非常自豪。

>我要斗胆做个预测。当你们活到八十岁，在某个安静的沉思时刻，回到内心深处，想起自己的人生故事时，最有意义的部份，将会是你所做过的那些选择。人生到头来，我们的选择，决定了我们是什么样的人（We are our choices.）。替你们自己写一篇精彩的人生故事吧。

Kata Potter
----------------------------------------
#Problem Description
 Once upon a time there was a series of 5 books about a very English hero called Harry. (At least when this Kata was invented, there were only 5. Since then they have multiplied) Children all over the world thought he was fantastic, and, of course, so did the publisher. So in a gesture of immense generosity to mankind, (and to increase sales) they set up the following pricing model to take advantage of Harry's magical powers.

One copy of any of the five books costs 8 EUR. If, however, you buy two different books from the series, you get a 5% discount on those two books. If you buy 3 different books, you get a 10% discount. With 4 different books, you get a 20% discount. If you go the whole hog, and buy all 5, you get a huge 25% discount.

Note that if you buy, say, four books, of which 3 are different titles, you get a 10% discount on the 3 that form part of a set, but the fourth book still costs 8 EUR.

Potter mania is sweeping the country and parents of teenagers everywhere are queueing up with shopping baskets overflowing with Potter books. Your mission is to write a piece of code to calculate the price of any conceivable shopping basket, giving as big a discount as possible.

For example, how much does this basket of books cost?
  2 copies of the first book
  2 copies of the second book
  2 copies of the third book
  1 copy of the fourth book
  1 copy of the fifth book

(answer: 51.20 EUR) 

#含义
原题目很长，大意就是出版社要促销一套哈利波特图书，该套图书共5集，每集单册购买价8元。若任意两集各买一本，打95折；若任意三集各买一本，打9折；若任意四集各买一本，打8折；若所有这五集都各买一本，打75折。上述优惠之外再购买的单册还是按8元一本计价。比如五集各买一本之外再加一本第一集，五本书打75折，这本另加的第一集按8元计价。若有人第一集买2本，第二集买2本，第三集买2本，第四集买1本，第五集买1本。问打折后最低的优惠价是多少钱？提示，不是51.6元，而是51.2元。

#思路
现在我想了一个办法，就是一个DiscountHandler，集成了95折、9折、8折、75折的handler，然后有一个购物车，从购物车用不同的handler去处理，然后会剩下剩余的购物车（这个购物车会有多种情况，比如1、2、3、4、5、3、4、5在8折策略后可能会是5、3、4、5，也可能是1、3、4、5），然后再把剩余的购物车用DiscountHandler去处理。
具体实现，还需要用业余时间去设计实现。