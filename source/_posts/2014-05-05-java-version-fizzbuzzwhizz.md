---
layout: post
title: 'Java版FizzBuzzWhizz'
date: 2014-05-05 15:26
comments: true
categories: [java, maven, TDD, fizzbuzzwhizz, 责任链, 模板方法, 设计模式, 趣味编程]
---
#个人近况
最近忙着替媳妇写论文，还忙招聘，忙写jQuery插件，忙写python，忙这忙那，今天赶紧利用晚上的1个多小时把五一期间做好的Java版本FizzBuzzWhizz博文也写了。有空还把Node.js的也贴上来，不过和Python差不多，到时候就大概贴代码了，不详细说明。
#Java版FizzBuzzWhizz
##1 概述
上个系列博文已经讲述了Python版本的FizzBuzzWhizz的思考、设计和实现思路。那么Java版本的肯定不是在做一个C2C（copy to clipboard）的事情。
##2 问题
###2.1 开闭原则
一个软件实体如类、模块和函数应该对扩展开放，对修改关闭。
那么在python中虽然已经把规则的逻辑条理分的很细了，但是所有的规则还是集中在一个函数里面，如果新增加一个规则那么就必然要修改到原来已经实现的代码。那么这样的设计是有风险的，任何的新规则很可能会影响到老规则的逻辑。
###2.2 封装
可以将信息封装为需要的数据对象，将数据与操作分离所带来的种种问题消除，提高了程序的可复用性和可维护性。
###2.3 自动构建
Python版本中并没有对项目进行自动的构建，Java版本中引入Maven来做项目构建管理。
##3 设计
在构思了下，用责任链模式、模板模式、工厂模式来完成这个设计。类设计图如下：
 ![](http://i1.tietuku.com/698717c0da7c4af0.jpg)
 
其中，IHandler是项目中的核心接口，其只有一个方法handler，是提供给外部的调用的处理方法。在python文章已经介绍，该游戏的规则分为“或”和“与”两种，所以设计中也定义了两种实现功能，其中wordHandler是为接口实现了“或”的特性，而UnitHandler则是实现了“与”的特性。
“或”的特性是通过责任链路的模式去实现的，了解servlet的人都知道，filter的filterChain.doFilter(request, response)方法可以将一个请求在一系列的处理类中传递request。本项目中的请求就是报数的数字i。
##4 代码
先看看核心接口IHandler。
```java IHandler.java
public interface IHandler {
 	   public Word handle(int number);
}
```
通过这个接口，很好的体现了面对对象多态的特性。
在看看两个核心的实现类。
```java WordHandler.java
/**
 * User: David
 * Date: 14-04-18
 * Time: 下午7:03
 */
public abstract class WordHandler implements IHandler {
    private IHandler wordHandler;

    public WordHandler(IHandler wordHandler) {
        this.wordHandler = wordHandler;
    }

    public Word handle(int number){
        String result = run(number);
        Word resultWord = null;
        if(result == null || "".equals(result)){
            resultWord = wordHandler.handle(number);
        }else{
            resultWord = Word.getWord(result);
        }
        return resultWord;
    }
	//钩子方法，类似Thread中的run方法
    protected abstract String run(int number);
    
    protected IHandler getWordHandler() {
        return wordHandler;
    }
}
```
```java UnitHandler.java

/**
 * User: David Date: 14-05-2 Time: 上午9:42
 */
public class UnitHandler extends WordHandler {
    
    private IHandler[] unitHandlers;
    
    public UnitHandler(IHandler wordHandler,IHandler ... unitHandlers) {
        super(wordHandler);
        this.unitHandlers = unitHandlers;
    }
    
    protected IHandler[] getUnitHandlers() {
        return unitHandlers;
    }
    
    @Override
    protected String run(int number) {
        StringBuilder result = new StringBuilder();
        for (IHandler handler : getUnitHandlers()) {
            if(handler instanceof WordHandler){
                result.append(((WordHandler) handler).run(number));
            }
        }
        
        return result.toString();
    }
    
}
```
个人不是很爱写注释，力求把代码写的简单易懂。WordHandler这个类就是通过在handler方法中调用run这个模板方法，而run方法在子类中去实现，当前handler如果没有结果，则会通过责任链将请求转发给下一个handler。
而UnitHandler则是构造函数运行传入任意数量个WordHandler，UnitHandler的run方法则是组合每个子handler的run方法来组合“与”规则，来达到与的效果。
具体的handler就不具体举例了，实现方法很简单。直接来看看如何调用吧。
```java
public static String translate(int i) {
        //这里还可以用单例工厂模式就创建，这里就不画蛇添足了
        IHandler commonNumberHandler = new CommonNumberHandler();
        IHandler fizzHandler = new FizzHandler(null);
        IHandler buzzHandler = new BuzzHandler(null);
        IHandler whizzHandler = new WhizzHandler(null);
        IHandler fizzBuzzWhizzHandler = new UnitHandler(
                commonNumberHandler, fizzHandler, buzzHandler, whizzHandler);
        IHandler containOneNumHandler = new ContainOneNumHandler(
                fizzBuzzWhizzHandler);

        Word word = containOneNumHandler.handle(i);
        return word.say();
    }
```
这个静态接口方法中通过组装需要的handler完成题目中所需要实现的游戏规则。handler的创建可以使用工厂模式，这里就不实现了，本次编码的重心并不在滥用设计模式上。
##5 测试
本项目用Junit4来做单元测试，单元测试没什么新意，和Python中差不多。但是代码经过了很多次重构，重构后直接运行单元测试就明确知道代码没有问题，足以体现单元测试在敏捷开发中的重要性，就算不是敏捷开发，单元测试也对提高工作效率减少测试工作量监测返工有很大的作用。
```java
/**
 * User: David
 * Date: 14-04-18
 * Time: 下午1:30
 */
@RunWith(JUnit4.class)
public class FizzBuzzTest {
    @Test
    public void GIVEN_One_SHOULD_SayOne() {
        assertEquals("Failure - 1 should be 1", "1", WordMaker.translate(1));
    }

    @Test
    public void GIVEN_Three_SHOULD_SayFizz() {
        assertEquals("Failure - 3 should be Fizz", Constants.WORD1, WordMaker.translate(3));
    }

    @Test
    public void GIVEN_Five_SHOULD_SayBuzz() {
        assertEquals("Failure - 5 should be Buzz", Constants.WORD2, WordMaker.translate(5));
    }

    @Test
    public void GIVEN_Fifteen_SHOULD_SayFizzBuzz() {
        assertEquals("Failure - 15 should be FizzBuzz", Constants.WORD1+Constants.WORD2, WordMaker.translate(15));
    }

    @Test
    public void GIVEN_Thirty_SHOULD_SayFizzBuzz() {
        assertEquals("Failure - 30 should be FizzBuzz", Constants.WORD1, WordMaker.translate(30));
    }

    @Test
    public void GIVEN_Seven_SHOULD_SayWhizz() {
        assertEquals("Failure - 7 should be Whizz", Constants.WORD3, WordMaker.translate(7));
    }

    @Test
    public void GIVEN_TwentyThree_SHOULD_SayFizz() {
        assertEquals("Failure - 23 should be Fizz", Constants.WORD1, WordMaker.translate(23));
    }

    @Test
    public void GIVEN_HundredFive_SHOULD_SayFizzBuzzWhizz() {
        assertEquals("Failure - 105 should be FizzBuzzWhizz", Constants.WORD1+Constants.WORD2+Constants.WORD3, WordMaker.translate(105));
    }

}
```	
##6 构建
完成以后直接用mvn clean test package就好了，构建完成就可以在输入目录找到FizzBuzzWhizz-0.0.1-SNAPSHOT.jar，本文中就不介绍Maven的使用，详细可以参考之前写过的Maven系列博文（可能没有从Iteye迁移到LogDown上），控制台输出日志如下：

[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building FizzBuzzWhizz 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:2.5:resources (default-resources) @ FizzBuzzWhizz ---
[debug] execute contextualize
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory D:\backup\wudw\code_temp_workspace\FizzBuzzWhizz\src\main\resources
[INFO] 
[INFO] --- maven-compiler-plugin:2.3.2:compile (default-compile) @ FizzBuzzWhizz ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-resources-plugin:2.5:testResources (default-testResources) @ FizzBuzzWhizz ---
[debug] execute contextualize
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory D:\backup\wudw\code_temp_workspace\FizzBuzzWhizz\src\test\resources
[INFO] 
[INFO] --- maven-compiler-plugin:2.3.2:testCompile (default-testCompile) @ FizzBuzzWhizz ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-surefire-plugin:2.10:test (default-test) @ FizzBuzzWhizz ---
[INFO] Surefire report directory: D:\backup\wudw\code_temp_workspace\FizzBuzzWhizz\target\surefire-reports

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.dragonsoft.fbw.FizzBuzzTest
Tests run: 8, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.132 sec

Results :

Tests run: 8, Failures: 0, Errors: 0, Skipped: 0

[INFO] 
[INFO] --- maven-jar-plugin:2.3.2:jar (default-jar) @ FizzBuzzWhizz ---
[INFO] Building jar: D:\backup\wudw\code_temp_workspace\FizzBuzzWhizz\target\FizzBuzzWhizz-0.0.1-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.128s
[INFO] Finished at: Mon May 05 23:21:47 CST 2014
[INFO] Final Memory: 3M/7M
[INFO] ------------------------------------------------------------------------

