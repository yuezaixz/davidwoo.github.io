---
layout: post
title: "用mysql对运动记录进行分时段统计"
date: 2015-05-07 22:11:52 +0800
comments: true
categories: 
---
##为什么写
之前做企业级应用都是使用Oracle的存储过程来做统计，用Job来定时执行。换成MySQL后一样用存储过程、视图、Event来完成相应的功能，只是函数和语法变得不大熟悉了，写篇博文来记录下。
这篇文章没什么特殊的，就是简单做点工作记录。
##做什么
其实很简单，就是每周定时执行，统计上一周各时段的运动情况总和。先来看看效果把。
![](http://blog.davidandty.com/images/15/1.jpg)

##怎么做
mysql存储过程和Oracle差别还是很大的，游标不能用了，除基础函数外的一个函数都不一致了，语法也不一样了，不好还好这些东西点点数据谷歌百度就可以查到了。
###存储过程
```sql
CREATE DEFINER=`runmove`@`%` PROCEDURE `week_runcount_statistics`(IN last_count int)
BEGIN
	Declare tempDayBegin int;
	Declare tempNightBegin int;
	Declare timeRouteCount int;
	Declare curr_week int;
    Declare dateBegin Date;
    Declare dateEnd Date;
	Declare vSQL varchar(2000);
	Declare stmt varchar(2100);
    set tempDayBegin = 0;
    set tempNightBegin = 24;
    if last_count = 0 then
		set dateEnd = date(curdate());
		set dateBegin = date(date_sub(curdate(),interval WEEKDAY(curdate()) day));
    else 
		if last_count = 1 then
			set dateEnd = date(date_sub(curdate(),interval WEEKDAY(curdate())+1 day));
			set dateBegin = date(date_sub(dateEnd ,interval 7 day));
		else 
			set dateEnd = date(date_sub(curdate(),interval WEEKDAY(curdate())+1+(last_count-1)*7 day));
			set dateBegin = date(date_sub(dateEnd ,interval 7 day));
		end if;
    end if;
    delete from route_statistics_by_week where week = YEARWEEK(dateBegin);
    
    set vSQL = ' ';
	set vSQL=CONCAT(vSQL,' insert into route_statistics_by_week');
	set vSQL=CONCAT(vSQL,' values ( date("',date_format(dateBegin,'%Y-%m-%d'),'"),date("',date_format(dateEnd,'%Y-%m-%d'),'"),',YEARWEEK(dateBegin));
    
    while tempDayBegin < tempNightBegin do
        
		SELECT count(r.id) into timeRouteCount FROM runmove.route r
		WHERE r.device_detail not like '%Simulator%' and r.flag = 1 and 
        YEARWEEK(date_format(r.create_date,'%Y-%m-%d')) = YEARWEEK(now())-last_count and 
        (DATE_FORMAT(r.create_date,'%H') between tempDayBegin and (tempDayBegin+1) )
		order by create_date desc;
		set vSQL=CONCAT(vSQL,',',timeRouteCount);
        
        set tempDayBegin = tempDayBegin + 1;
        
    end while;
    set vSQL=CONCAT(vSQL,' )');
    set @sqltext:=vSQL;
	prepare stmt from @sqltext;
	execute stmt;
    
END
```

###定时作业
event和Oracle的job定义逻辑也差不多，就是语法不大一样。

```sql event
CREATE  EVENT IF NOT EXISTS  STAT
ON  SCHEDULE  EVERY  7  DAY  STARTS DATE_ADD(DATE_ADD(DATE_SUB(CURDATE(),INTERVAL WEEKDAY(CURDATE())-1 DAY), INTERVAL 7 DAY),INTERVAL 1 HOUR)
ON  COMPLETION  PRESERVE  ENABLE
DO CALL week_runcount_statistics(1);
```

为了显示效果，所以再加了一个视图去显示。

###视图

```sql view
CREATE 
    ALGORITHM = UNDEFINED 
    DEFINER = `runmove`@`%` 
    SQL SECURITY DEFINER
VIEW `v_route_statistics_by_week` AS
    SELECT 
        `route_statistics_by_week`.`week` AS `week`,
        `route_statistics_by_week`.`start_date` AS `start_date`,
        `route_statistics_by_week`.`end_date` AS `end_date`,
        `route_statistics_by_week`.`time_zone0` AS `0~1`,
        `route_statistics_by_week`.`time_zone1` AS `1~2`,
        `route_statistics_by_week`.`time_zone2` AS `2~3`,
        `route_statistics_by_week`.`time_zone3` AS `3~4`,
        `route_statistics_by_week`.`time_zone4` AS `4~5`,
        `route_statistics_by_week`.`time_zone5` AS `5~6`,
        `route_statistics_by_week`.`time_zone6` AS `6~7`,
        `route_statistics_by_week`.`time_zone7` AS `7~8`,
        `route_statistics_by_week`.`time_zone8` AS `8~9`,
        `route_statistics_by_week`.`time_zone9` AS `9~10`,
        `route_statistics_by_week`.`time_zone10` AS `10~11`,
        `route_statistics_by_week`.`time_zone11` AS `11~12`,
        `route_statistics_by_week`.`time_zone12` AS `12~13`,
        `route_statistics_by_week`.`time_zone13` AS `13~14`,
        `route_statistics_by_week`.`time_zone14` AS `14~15`,
        `route_statistics_by_week`.`time_zone15` AS `15~16`,
        `route_statistics_by_week`.`time_zone16` AS `16~17`,
        `route_statistics_by_week`.`time_zone17` AS `17~18`,
        `route_statistics_by_week`.`time_zone18` AS `18~19`,
        `route_statistics_by_week`.`time_zone19` AS `19~20`,
        `route_statistics_by_week`.`time_zone20` AS `20~21`,
        `route_statistics_by_week`.`time_zone21` AS `21~22`,
        `route_statistics_by_week`.`time_zone22` AS `22~23`,
        `route_statistics_by_week`.`time_zone23` AS `23~00`
    FROM
        `route_statistics_by_week`
```