---
layout: post
title: 'Java解析Xml导出Word'
date: 2014-03-28 10:42
comments: true
categories: [java, 模板引擎, 导出Word, 解析Xml, 设计]
---
#需求概述
不知道大家有没做过用xml定义需要导出word的格式和需要的数据，然后通过Java代码来解析Xml完成导出。
本文不考虑解析Xml和怎么导出Word，只考虑如何用解析后的对象来生成需要导出的数据。
本文使用的Xml解析方法为Dom4j，Word导出方法为Itext。
#最初的想法
```java
public static void main(String[] args) {
    	try {
    		Object data = getData();
	    	Element xmlElement = DXmlUtil.getXmlElement("format.xml", "/Doc");
		IDocFactory factory = new DDocFactory();
         Document doc = factory.getDocFactory(out);
	    	proc(xmlElement,data, doc, factory);
	    } catch (Exception e) {
	    	e.printStackTrace();
	    }
	}
    
    public static void proc(Element element,Object obj,Document doc,IDocFactory factory){
    	//TODO 接下来就是见证奇迹的时刻
    }
```

```xml 需解析的模板
<?xml version="1.0" encoding="UTF-8"?>
<Doc title="${app.name}" subtitle="运维服务报告" exportpath="F:\\音乐\\哈哈哈" >
	<chapter level="1" num="1" title="目的">
		<para type="content">本服务报告按月对系统运维情况进行总结，对出现的问题给与说明，以方便用户、部门领导等相关人员对系统状况和项目情况的了解，并为领导决策提供依据。</para>
	</chapter>
	<chapter level="1" num="2" title="数据来源">
		<para type="content">本服务报告的数据来源包括以下几个部分：</para>
		<list type="diamod">
			<item>服务器运行状态巡检数据</item>
			<item>巨龙运维监控工具预警数据</item>
			<item>巨龙运维流程系统处置数据</item>
		</list>
	</chapter>
	<chapter level="1" num="3" title="系统运维工作说明">
		<chapter level="2" num="3.1" title="系统基础情况">
			<para type="contentItalic">${app.name}共有xx台服务器CPU资源使用达到xx%，CPU资源占用xx，CPU处理出现了等待状态；</para>
			<para type="contentItalic">${app.name}多少台内存消耗比例达到XX%，内存消耗了多少G，系统运行时剩余内存多少G，服务器内存出现XXX状态。</para>
			<para type="contentItalic">${app.name}在什么时段出现磁盘IO瓶颈情况，每秒最高达到多少M/s，最低达到多少M/s。</para>
			<para type="contentItalic">综合以上分析，${app.name}整体运行情况，系统运行资源紧张，CPU、内存、磁盘相对处于高负荷状态。其中XX系统的CPU资源使用过高，XX系统的内存使用明显异常，磁盘IO严重，磁盘增长明显过快，建议更换升级服务器资源或增加硬件资源，平衡系统资源。</para>
		</chapter>
		<chapter level="2" num="3.2" title="系统运行状况">
			<chapter level="2" num="3.2.1" title="系统可用情况">
				<loop items="app.appSet">
					<arrow title="${name}">
						<para type="content">故障次数${data.failTotal}次，其中一级故障${data.failHigh}次，二次故障${data.failMiddle}次，三级故障${data.failLow}次。</para>
						<para type="content">可用性说明：本月重点人系统运行正常，业务繁忙时段xx访问测试X次，正常访问X次。</para>
					</arrow>
				</loop>
			</chapter>
			<chapter level="2" num="3.2.2" title="系统可用情况">
				<loop items="app.appSet">
					<arrow title="${name}">
						<para type="content">非计划中断次数${data.interruptNum}次，系统稳定性${steady}。</para>
					</arrow>
				</loop>
			</chapter>
		</chapter>
		
		<chapter level="2" num="3.3" title="系统运行状态趋势分析">
			<chapter level="2" num="3.3.1" title="系统故障趋势分析">
				<chapter level="2" num="3.3.1.1" title="XX系统">
					<arrow title="数据统计">
						<para type="desc">${app.name}${app.twoPrevMonth}故障数{app.twoPreData.failTotal}，${app.onePrevMonth}故障数${app.onePreData.failTotal}，本月${app.month}故障数${app.data.failTotal}。从故障数据分析来看，故障数呈${app.trend}趋势。</para>
					</arrow>
					<arrow title="原因分析">
						<para type="desc">描述本月故障主要集中点，分析造成原因。</para>
					</arrow>
					<arrow title="改进措施">
						<para type="desc">针对本月故障上升的情况，提出改进措施。</para>
					</arrow>
				</chapter>
			</chapter>
		</chapter>
	</chapter>
	<chapter level="1" num="4" title="运维服务情况统计">
		<para type="content">${app.month}共进行系统升级xx次，自动监控${app.data.monitorFail}次，人工巡检${app.data.requestFail}次，事件响应xx次。具体情况如下：</para>
		<arrow title="人工巡检：">
			<para type="content">故障数量${app.data.requestFail}次，已解决数量${app.data.requestSolve}次，未解决数量${app.data.requestUnsolve}次，解决率${app.data.requestSolveRate}%。</para>
		</arrow>
		<arrow title="自动监控：">
			<para type="content">故障数量${app.data.monitorFail}次，已解决数量${app.data.monitorSolve}次，未解决数量${app.data.monitorUnsolve}次，解决率${app.data.requestSolveRate}%。</para>
		</arrow>
		<arrow title="事件响应：">
			<para type="content">故障数量x次，已解决数量x次，未解决数量x次，解决率x%。</para>
		</arrow>
		<arrow title="系统升级：">
			<para type="content">故障数量x次，已解决数量x次，未解决数量x次，解决率x%。</para>
		</arrow>
		<chapter level="2" num="4.1" title="重大事件报告">
			<para type="desc">如本月有运维相关的重大事件，可在此处简略报告。</para>
		</chapter>
	</chapter>
	
	<endChapter num="5" title="用户评价及反馈" >
		<para type="desc">报告打印出来后，由用户方运维工作接口人在本章节对运维工作进行评价。</para>
		<para type="content">本月运维报告用户签收确认：</para>
		<para type="content"></para>
		<para type="content"></para>
		<para type="content">本月运维工作用户评价：</para>
		<para type="content"></para>
		<para type="content"></para>
		<para type="content"></para>
		<para type="right">用户签名：</para>
		<para type="right"   日期：</para>
	</endChapter>
</Doc>
```
#处理元素
```java
	private static void procChild(Document doc, IDocFactory factory,
            Object model, org.dom4j.Element element) {

        List elements = element.elements();
        if (elements != null && !elements.isEmpty()) {
            for (Object obj : elements) {
                org.dom4j.Element ele = (org.dom4j.Element) obj;
                String eleName = ele.getName();
                //..... 这里通过元素的名称来判断下一步通过哪种方法处理
            }
        }
```
#生成无参数的模板内容
这里通过很多工具类，根据标签的不同使用不同的处理方法，方法如下图。
 
随便找一个方法出来举个例子
```java
	private static void procPara(Document doc, IDocFactory factory,
            Object model, org.dom4j.Element ele) {
    	//获取该段的样式类型，如正文、斜体、初体、居右、注释
        String type = getElementStr(ele, model,DocConstants.PARA_TYPE);
        //获取段落的内容
        String text = getElementData(model, ele);
        //通过抽象工厂，根据类型生成段落
        Paragraph para = factory.getParagraph(text, type);
        //将段落添加到文档输出流中,等close的时候统一输出
        add(doc, para);
        //处理子节点，如果有的话
        procChild(doc, factory, model, ele);
    }
```
#处理循环
需求要求支持循环，类似<c:forEach>标签的效果。如下就是遍历平台的所有子系统。
```xml
<loop items="app.appSet">
					<arrow title="${name}">
						<para type="content">故障次数${data.failTotal}次，其中一级故障${data.failHigh}次，二次故障${data.failMiddle}次，三级故障${data.failLow}次。</para>
						<para type="content">可用性说明：本月重点人系统运行正常，业务繁忙时段xx访问测试X次，正常访问X次。</para>
					</arrow>
				</loop>
```
#代码实现
```Java
	   String itemsName = getElementStr(ele, model, DocConstants.LOOP_ITEMS);
   Set apps = (Set) getObjectAttribute(model, itemsName);
         if (apps.isEmpty()) {
                procChild(doc, factory, model, ele);
         } else {
                for (Object app : apps) {
                    procChild(doc, factory, app, ele);
                }
     }
```
#处理对象
首先，在模板中有很多数据，我们要获取，就一定要有一个通过反射来获取数据的方法。
```java
		Object result = null;
        Method method = obj.getClass().getMethod(
               "get" + StringUtils.firstUpperCase(tempAttr));
        // 判断调用方法的返回类型
        if (method != null) {
            result = method.invoke(obj);
        }
    return result;
```
#嵌套对象处理
如果需要处理的参数属于存在对象嵌套的，那要先处理这层关系。
```java
private static String getDupObjectAttribute(Object model, String val) {
        String result = null;
        String[] vals = val.split("\\.", 2);
        Object object = getObjectAttribute(model, vals[0]);
        if(object == null){
            return "-1";
        }
        if (vals[1].contains(".")) {
            result = getDupObjectAttribute(object, vals[1]);
        }
        else {
            result = (String) getObjectAttribute(object, vals[1]);
        }
        return result;
    }
```
#处理参数
那么可以处理数据了，那参数要怎么从模板中寻找出来呢？第一步就是使用正则表达式来寻找需要处理的参数。
```java
	//REGEX = "\\$\\{([^\\}]+)?\\}"
		Pattern pat = Pattern.compile(REGEX);
        Matcher mat = pat.matcher(text);
        String result = text;
        while (mat.find()) {
            String val = mat.group(1);
            String resultVal = null;
            try {
                if (val.contains(".")) {
				//如果存在嵌套，则通过反射先获取
                    resultVal = getDupObjectAttribute(model, val);
                }
                else {
                    resultVal = (String) getObjectAttribute(model, val);
                }
                if(StringUtils.isNotBlank(resultVal) && !"-1".equals(resultVal)){
                    result = result.replaceFirst("\\$\\{" + val + "\\}", resultVal);
                }
            }
            catch(Throwable e) {
                e.printStackTrace();
            }
        }
        return result;
```
#思考
用Java显然对于制作模板引擎不是很擅长。之前介绍Javascript正则的时候，用20行代码，就可以实现更强大的功能，不过安全性会有一点问题。存在被植入代码的可能。
