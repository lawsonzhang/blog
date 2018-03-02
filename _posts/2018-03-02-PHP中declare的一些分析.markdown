---
layout: post
title:  "PHP中declare的一些分析"
date:   2018-03-02 13:20:26 +0800
categories: php algorithm
---

#### 手册说明：
> ` (PHP 4, PHP 5, PHP 7) `
> ` declare ` 结构用来设定一段代码的执行指令。` declare ` 的语法和其它流程控制结构相似
> > ``` 
> > declare (directive)
> >     statement
> > ```
> 
> ` directive ` 部分允许设定 ` declare ` 代码段的行为。目前只认识两个指令：` ticks `以及 ` encoding `（` Note: encoding 是 PHP 5.3.0 新增指令。`）
> ` declare ` 代码段中的 ` statement ` 部分将被执行——怎样执行以及执行中有什么副作用出现取决于 ` directive ` 中设定的指令
> ` declare ` 结构也可用于全局范围，影响到其后的所有代码（但如果有 ` declare ` 结构的文件被其它文件包含，则对包含它的父文件不起作用）
> > ```
> > <?php
> >     declare(ticks=1);
> >     // entire script here 全局影响
> > ```
> > 
> > ```
> > <?php
> >     declare(ticks=1) {
> >         // entire script here
> >     }
> > ```
> 
> ***Ticks:***
> ` Tick `（时钟周期）是一个在 ` declare ` 代码段中解释器每执行 N 条可计时的 ***低级语句*** 就会发生的事件。N 的值是在 ` declare ` 中的 ` directive ` 部分用 ` ticks=N ` 来指定的
> 不是所有语句都可计时。通常条件表达式和参数表达式都不可计时
> 在每个 ` tick ` 中出现的事件是由 ` register_tick_function() ` 来指定的
> 
> ***Encoding:***
> 可以用 ` encoding ` 指令来对每段脚本指定其编码方式

#### Ticks
***上代码 ：***

```
<?php
    echo "start: \n";
    register_tick_function('_handler');
    declare(ticks = 1) {
        for ($i = 0; $i < 5; $i++) { 
            echo '|test|' . "\n";
        }
     }
     echo ":end \n";

     function _handler()
     {
         echo '|here is handler|' . "\n";
     }
```
 
***运行结果 ：***
```
start: 
|test|
|here is handler|
|test|
|here is handler|
|test|
|here is handler|
|test|
|here is handler|
|test|
|here is handler|
|here is handler|
:end
```

***疑问 ：***

1. 为什么先输出 '|test|' 之后才输出 '|here is handler|' 
2. 为什么在输出最后一个 '|test|' 后还输出2个 '|here is handler|'
3. declare中的for循环怎么分解成低级语句(low-level)

***分析 ：***

1. 通过手册中对于 ` tick ` 中的描述 可以得知： ` 代码段中解释器每执行 N 条可计时的 ***低级语句*** 就会发生的事件 `
