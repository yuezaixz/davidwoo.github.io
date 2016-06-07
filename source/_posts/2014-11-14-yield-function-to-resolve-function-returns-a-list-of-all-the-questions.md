---
layout: post
title: 'yield函数，解决函数返回列表时的种种问题'
date: 2014-11-14 07:37
comments: true
categories: 
---
在开发中可能经常碰到这样的问题，举个比较简单的场景描述，比如我要获取1-100的整数集合，那么我第一反应会这样想这个函数。
```java
	public void getNumList(int max){
		for (int i = 1; i < max; i++) {
			System.out.println(i);
		}
	}
```
那么这么写的缺点有哪些呢？就是我无法获得这个集合，那么好吧，我就修改下这个函数。
```java
	public List<IntegergetNumList(int max){
		List<IntegernumList = new ArrayList<Integer>();
		for (int i = 1; i < max; i++) {
			numList.add(i);
		}
		return numList;
	}
```
好了，这样是不是问题解决了，这篇博文就到这了呢？不是，这样还有个问题，就是当列表元素占用内存过大，并且列表元素过多时，这样操作是非常占用内存的。
好吧，有人说，他知道这么解决这个问题，多简单，看如下代码。
```java
	class NumList implements Iterable<Integer>{
		private int index;
		private int max;
		public NumList(int max){
			index = 1;
			this.max = max;
		}
		
		public Iterator<Integeriterator() {
			return new NumIterator();
		}
		
		class NumIterator implements Iterator<Integer{
			
			
			public boolean hasNext() {
				return index != max;
			}
			
			public Integer next() {
				return ++index;
			}
			
			public void remove() {
				index++;
			}
			
		}
	}
```
没错，用迭代器这问题是解决了，但是，这样的设计有点过于复杂。感觉小题大做了。在JAVA中有没有方法可以快速解决这个问题呢？我感觉是没有啦，那就让我们看看今天的主角，yield在python中是如何使用的。
```
def numList(max): 
    n = 1
    while n <= max: 
    		# print n
        yield n 
        n = n + 1
def read_file(fpath): 
    BLOCK_SIZE = 1024 
    with open(fpath, 'rb') as f: 
        while True: 
            block = f.read(BLOCK_SIZE) 
            if block: 
                yield block 
            else: 
                return
def main():
    for num in numList(10):
        print num

    for content in read_file('../README.md'):
        print content
```
两个函数，第一个函数就是返回数列，是不是很神奇。简单地讲，yield 的作用就是把一个函数变成一个 generator，带有 yield 的函数不再是一个普通函数，Python 解释器会将其视为一个 generator，调用 fab(5) 不会执行 fab 函数，而是返回一个 iterable 对象！在 for 循环执行时，每次循环都会执行 fab 函数内部的代码，执行到 yield b 时，fab 函数就返回一个迭代值，下次迭代时，代码从 yield b 的下一条语句继续执行，而函数的本地变量看起来和上次中断执行前是完全一样的，于是函数继续执行，直到再次遇到 yield。也可以手动调用 fab(5) 的 next() 方法（因为 fab(5) 是一个 generator 对象，该对象具有 next() 方法），这样我们就可以更清楚地看到 fab 的执行流程：
在一个 generator function 中，如果没有 return，则默认执行至函数完毕，如果在执行过程中 return，则直接抛出 StopIteration 终止迭代。
一个带有 yield 的函数就是一个 generator，它和普通函数不同，生成一个 generator 看起来像函数调用，但不会执行任何函数代码，直到对其调用 next()（在 for 循环中会自动调用 next()）才开始执行。虽然执行流程仍按函数的流程执行，但每执行到一个 yield 语句就会中断，并返回一个迭代值，下次执行时从 yield 的下一个语句继续执行。看起来就好像一个函数在正常执行的过程中被 yield 中断了数次，每次中断都会通过 yield 返回当前的迭代值。

yield 的好处是显而易见的，把一个函数改写为一个 generator 就获得了迭代能力，比起用类的实例保存状态来计算下一个 next() 的值，不仅代码简洁，而且执行流程异常清晰。

如何判断一个函数是否是一个特殊的 generator 函数？可以利用 isgeneratorfunction 判断：
```python
from inspect import isgeneratorfunction 
print isgeneratorfunction(numList) 
```
