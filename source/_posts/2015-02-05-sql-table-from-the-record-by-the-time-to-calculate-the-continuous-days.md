---
layout: post
title: '用SQL从记录表中根据时间算出连击天数'
date: 2015-02-05 05:05
comments: true
categories: [蛋疼, 折腾, 作死]
---
#概述
这是一个蛋疼的折腾，just for fun。
问题是这样的，有一张表A，有user_id,create_time这两个属性，要用sql查询，查询的结果是user_id,continue_days。
#首先
数据库是mysql，没有oracle那么强大的函数集，连row_num都是要自己实现的，也没有什么好用的聚合函数，逻辑控制函数，用sql做这个操作是很蛋疼，不如用代码去查询，但是，俺本着just for fun的精神折腾了下。
然后想了下要怎么操作，就想到了一个法子。
我的思路是这样的：先根据时间去group by，把同一天内相同的数据合并，然后排序并编号，那么这张表自身内联下，t1.create_date - t2.create_date = t1.row_num - t2.row_num 的记录，取他的t1.create_date - t2.create_date结果，那就是连续的天数。
#问题
实现过程碰到两个问题，一个是t1.create_date - t2.create_date的结果要加上1，这才是正确的结果，否则就忽略了起始日。
另一个问题是开始直接用yyyymmdd形式，忽略了跨月，所以后来改成了TO_DAYS函数做转化。
#实现
实现就看几个sql把。
```sql view_route
CREATE
    ALGORITHM = UNDEFINED
    DEFINER = `team_xm`@`%`
    SQL SECURITY DEFINER
VIEW `view_route` AS
    SELECT
        (TO_DAYS(`t`.`create_date`) - TO_DAYS('2000-01-01')) AS `r_timestamp`,
        `t`.`user_id` AS `user_id`
    FROM
        `route` `t`
    WHERE
        (`t`.`user_id` IN ('3' , '7', '17', '1'))
```
![](http://chuantu.biz/t/65/1423113571x1822611171.jpg)
```sql

CREATE
    ALGORITHM = UNDEFINED
    DEFINER = `team_xm`@`%`
    SQL SECURITY DEFINER
VIEW `view_route_bydate` AS
    SELECT
        `t`.`r_timestamp` AS `r_timestamp`,
        `t`.`user_id` AS `user_id`
    FROM
        `view_route` `t`
    GROUP BY `t`.`r_timestamp` , `t`.`user_id`
    ORDER BY `t`.`r_timestamp`
```
![](http://chuantu.biz/t/65/1423113597x1822611171.jpg)
```sql

CREATE
    ALGORITHM = UNDEFINED
    DEFINER = `team_xm`@`%`
    SQL SECURITY DEFINER
VIEW `view_route_bydate_withnum` AS
    SELECT
        `t`.`r_timestamp` AS `r_timestamp`,
        `t`.`user_id` AS `user_id`,
        (SELECT
                (COUNT(0) + 1) AS `row_num`
            FROM
                `view_route_bydate` `t2`
            WHERE
                ((`t2`.`r_timestamp` < `t`.`r_timestamp`)
                    AND (`t2`.`user_id` = `t`.`user_id`))) AS `row_num`
    FROM
        `view_route_bydate` `t`
```
![](http://chuantu.biz/t/65/1423113616x1822611171.jpg)
```sql

CREATE
    ALGORITHM = UNDEFINED
    DEFINER = `team_xm`@`%`
    SQL SECURITY DEFINER
VIEW `view_user_riding_day_count` AS
    SELECT
        `t1`.`user_id` AS `user_id`,
        (MAX((`t1`.`row_num` - `t2`.`row_num`)) + 1) AS `count`
    FROM
        (`view_route_bydate_withnum` `t1`
        JOIN `view_route_bydate_withnum` `t2`)
    WHERE
        (((`t1`.`r_timestamp` - `t2`.`r_timestamp`) = (`t1`.`row_num` - `t2`.`row_num`))
            AND (`t1`.`user_id` = `t2`.`user_id`))
    GROUP BY `t1`.`user_id`

```

![](http://chuantu.biz/t/65/1423113634x1822611171.jpg)