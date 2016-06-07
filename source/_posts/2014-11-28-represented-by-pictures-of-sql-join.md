---
layout: post
title: '用几张图片表示Sql的join'
date: 2014-11-28 05:32
comments: true
categories: [sql, join, 连接]
---
#背景
上学的时候，学Sql有一个比较大的苦恼就是分清那一堆的连接，什么左外连接，右外连接，外全连接，内连接……等等一大堆了，不是说他们多难，只是考试会把他们记混掉。最近看到Jeff Wood的一篇文章，他用图片的方式表示，我觉得非常形象，就搬到自己的博客来，自己稍微简化了一下。
##数据表
####表A
1	Pirate
2	Monkey
3	Ninja
4	Spaghetti
####表B
1	Rutabaga
2	Pirate
3	Darth Vader
4	Ninja
#不用连接
多表查询最简单的方式就是下面这句，但是其实这句话的效率非常低，因为他产生的结果集说亮是两个表数量的笛卡尔积，即a\*b，所以from a,b其实是先获取了一个a\*b的结果集，然后再执行where去过滤，再执行group by去分组，再select的。
```sql
select * from a,b;
```
结果无法用图表示，输出为
1  |  Pirate  |  1  |  Rutabaga
2  |  Monkey  |  1  |  Rutabaga
3  |  Ninja  |  1  |  Rutabaga
4  |  Spaghetti  |  1  |  Rutabaga
1  |  Pirate  |  2  |  Pirate
2  |  Monkey  |  2  |  Pirate
3  |  Ninja  |  2  |  Pirate
4  |  Spaghetti  |  2  |  Pirate
1  |  Pirate  |  3  |  Darth Vader
2  |  Monkey  |  3  |  Darth Vader
3  |  Ninja  |  3  |  Darth Vader
4  |  Spaghetti  |  3  |  Darth Vader
1  |  Pirate  |  4  |  Ninja
2  |  Monkey  |  4  |  Ninja
3  |  Ninja  |  4  |  Ninja
4  |  Spaghetti  |  4  |  Ninja

#内连接
![](http://jbcdn2.b0.upaiyun.com/2013/05/iinner-join.png)
如图，内连接即为两个表交叉的部分。
```sql
SELECT * FROM a INNER JOIN b ON a.name = b.name;
```
输出为
1	Pirate	2	Pirate
3	Ninja	4	Ninja

#全外连接
![](http://jbcdn2.b0.upaiyun.com/2013/05/Full-outer-join.png)
如图，全外连接既打印出两个表所有的结果，包括无交叉的部分。
由于mysql不支持full outer join，所以下面列出了两条语句
```sql
---mysql
SELECT * FROM a left JOIN b ON a.name = b.name union SELECT * FROM a right JOIN b ON a.name = b.name;
---other
SELECT * FROM a FULL OUTER JOIN b ON a.name = b.name;
```
输出为
1	Pirate	2	Pirate
2	Monkey	\N	\N
3	Ninja	4	Ninja
4	Spaghetti	\N	\N
\N	\N	1	Rutabaga
\N	\N	3	Darth Vader

#左外连接
![](http://jbcdn2.b0.upaiyun.com/2013/05/Left-outer-join.png)
如图，就是列出左边表所有数据，如果右表与之有交叉，就把数据叠加上。
```sql
SELECT * FROM a left JOIN b ON a.name = b.name;
```
输出为
1	Pirate	2	Pirate
2	Monkey	\N	\N
3	Ninja	4	Ninja
4	Spaghetti	\N	\N

#左外连接过滤交叉
![](http://jbcdn2.b0.upaiyun.com/2013/05/WHERE-TableB.id-IS-nul.png)
如图，就是列出左边表与右表无交叉部分
```sql
SELECT * FROM a left JOIN b ON a.name = b.name WHERE b.id IS null;
```
输出为
2	Monkey	\N	\N
4	Spaghetti	\N	\N

#右外连接 与 右外连接过滤交叉
与左解释差不多，不重复解释。
```sql
SELECT * FROM a right JOIN b ON a.name = b.name;
SELECT * FROM a right JOIN b ON a.name = b.name WHERE a.id IS null;
```
输出为
左外：
\N	\N	1	Rutabaga
1	Pirate	2	Pirate
\N	\N	3	Darth Vader
3	Ninja	4	Ninja
左外去交叉：
\N	\N	1	Rutabaga
\N	\N	3	Darth Vader

