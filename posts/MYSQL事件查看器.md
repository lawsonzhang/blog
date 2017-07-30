# MYSQL事件查看器

###### __文档撰写人：__ 张志鸿 (2017-07-13)

---

## 概述
> MySQL也有自己的事件调度器，简单地可以理解为linux的crontab job，不过对于SQL应用来说，它的功能更齐全，也更易于维护。个人感觉如果数量创建太多的话，也可能影响DB性能，且不易调试。

## MySQL事件调度器的主要内容
#### 总开关
> 参数 `event_scheduler` 为事件调度器的总开关，一般来说设置为ON或者OFF就好，不建议设置成disabled，如果设置为ON，`show processlist` 可看到该线程

#### 创建，修改，查看等语法
> 关于如何创建，修改event这里不做叙述，创建语法如下，具体的含义可参考下面关于event信息表介绍。也可以参考官网文档链接，http://dev.mysql.com/doc/refman/5.6/en/create-event.html
> 
> 查看创建好的event，在进入当前db后，`show create event xxx`


#### event的信息查询和含义

> 查看某个event的状态信息，可查看 `mysql.event` 或者 `information_schema.events` ，或者简单地切到当前DB后执行 `show events` ; 三者的内容基本一致， `information_schema` 无法做了下数据复制，更改了下列名称和starts时间以便更好的阅读。这里已 `information_schema.events` 里的信息为例解释
> __语句：__ ` SELECT * FROM INFORMATION_SCHEMA.EVENTS WHERE EVENT_NAME = 'call_generate_daily_assets' ;` 查询事件详细信息
> 
> __EVENT_CATALOG：__ 一般都是def，不管
> __EVENT_SCHEMA：__ event所在的schema
> __EVENT_NAME：__ event的名称
> __DEFINER：__ event的定义者，和定义这个event时，默认selectcurrent_user()的结果一致，如果该user有super权限，可以指定为其他用户
> __TIME_ZONE：__ event使用的时区，默认是system，建议别做修改
> __EVENT_BODY：__ 一般都是SQL，不用管
> __EVENT_DEFINITION：__ 该event的内容，可以是具体的insert等SQL，也可以是一个调用存储过程的操作
> __EVENT_TYPE：__ 这个参数比较重要，定义的时候指定，有两个值：RECURRING和ONE TIME，RECURRING表示只要符合条件就会重复执行，而ONE TIME只会调用一次
> __EXECUTE_AT：__  针对one-time类型的event有效，如果是RECURRING类型的event一般为NULL，表示该event的预计执行时间
> __INTERVAL_VALUE：__ 针对RECURRING类型的event有效，表示执行间隔长度
> __INTERVAL_FIELD：__  针对RECURRING类型的event有效，表示执行间隔的单位，一般是SECOND，DAY等值，可参考创建语法 SQL_MODE：当前event采用的SQL_MODE
> __STARTS：__ 针对RECURRING类型的event有效，表示一个event从哪个时间点点开始执行，和one-time的EXECUTE_AT功能类似。为NULL表示一符合条件就开始执行
> __ENDS：__ 针对RECURRING类型的event有效，表示一个event到了哪个时间点后不再执行，如果为NULL就是永不停止
> __STATUS：__ 一般有三个值，ENABLED, DISABLED和 SLAVESIDE_DISABLED，其中ENABLED表示激活这个event，该event只要符合其他条件就会执行；DISABLED状态改event将不会执行，SLAVESIDE_DISABLED表示在从库上不执行该event。需要特别注意在从库上不要执行任何形式的event，因为如果主库执行一次，复制到从库后，从库再执行一次的话，那就数据不一致了，一般来说直接禁用掉从库上的总开关event_scheduler就行。
> __ON_COMPLETION：__ 只有两种值，PRESERVE和NOT PRESERVE，PRESERVE
> __CREATED：__ event的创建时间
> __LAST_ALTERED：__ event最新一次被修改的时间
> __LAST_EXECUTED：__ event最近一次执行的时间，如果为NULL表示从未执行过
> __EVENT_COMMENT：__ event的注释信息
> __ORIGINATOR：__ 当前event创建时的server-id，用于主从上的处理，比如SLAVESIDE_DISABLED
> __CHARACTER_SET_CLIENT：__ event创建时的客户端字符集，即character_set_client
> __COLLATION_CONNECTION：__ event创建时的连接字符校验规则，即collation_connection
> __DATABASE_COLLATION：__ event创建时的数据库字符集校验规则

#### EVENT的权限管理
> 1. 设置event_scheduler系统变量，需要super_priv权限
> 2. 创建，修改和删除event需要该user用户EVENT权限，该权限是schema级别的
> 3. 对应于event的具体内容，需要对应的权限。比如event里有对某张表的insert操作，那么该user需要对该表的insert操作，不然LAST_EXECUTED一直会是NULL

#### EVENT的状态查询
> 通过以下命令查看DB启动以来的event的相关信息统计
```
mysql> showglobal status like '%event%';
+--------------------------+-------+
|Variable_name | Value |
+--------------------------+-------+
|Com_alter_event | 0 |
|Com_create_event | 2 |
|Com_drop_event | 2 |
|Com_show_binlog_events | 0 |
|Com_show_create_event | 191 |
|Com_show_events | 40 |
|Com_show_relaylog_events | 0 |
+--------------------------+-------+
7 rows in set(0.00 sec)
```

## 使用建议
> 1. 如果主库已经执行过，从库上务必要保证event不会执行(除非故意在slave上创建的event)
> 2. 创建，删除等操作严禁直接操作mysql.event表，而是通过create等正规语法实现，不然会导致元数据混乱，各种莫名其妙的问题随之产生，比如event不执行或者重复执行。这时一般只有重启DB才能解决 了。
> 3. 创建的event涉及到海量数据变更的话，要做好充分测试，确保不影响现网服务
> 4. 如果需要备份带有event的DB，mysqldump时需要加上--event参数
以上这篇老生常谈mysql event事件调度器(必看篇)就是小编分享给大家的全部内容了，希望能给大家一个参考，也希望大家多多支持脚本之家。