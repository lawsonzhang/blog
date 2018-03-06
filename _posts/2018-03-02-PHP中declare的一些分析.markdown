---
layout: post
title:  "PHP中declare的一些分析"
date:   2018-03-02 13:20:26 +0800
categories: php algorithm
---

#### 手册说明：
- ` (PHP 4, PHP 5, PHP 7) `
- ` declare ` 结构用来设定一段代码的执行指令。` declare ` 的语法和其它流程控制结构相似
``` 
 declare (directive)
     statement
```
 
- `directive` 部分允许设定 `declare` 代码段的行为。目前只认识两个指令：`ticks` 以及 `encoding` （ Note: encoding 是 PHP 5.3.0 新增指令。）
- `declare` 代码段中的 `statement` 部分将被执行——怎样执行以及执行中有什么副作用出现取决于 `directive` 中设定的指令
- `declare` 结构也可用于全局范围，影响到其后的所有代码（但如果有 `declare` 结构的文件被其它文件包含，则对包含它的父文件不起作用）
```
<?php
    declare(ticks=1);
    // entire script here 全局影响
```
```
<?php
    declare(ticks=1) {
        // entire script here
    }
```

***Ticks:***
- `Tick`（时钟周期）是一个在 `declare` 代码段中解释器每执行 N 条可计时的 ***低级语句*** 就会发生的事件。N 的值是在 `declare` 中的 `directive` 部分用 `ticks=N` 来指定的
- 不是所有语句都可计时。通常条件表达式和参数表达式都不可计时
- 在每个 `tick` 中出现的事件是由 `register_tick_function()` 来指定的
 
***Encoding:***
- 可以用 `encoding` 指令来对每段脚本指定其编码方式

***Strict_types:***
- 这个类别在php7以后存在, 用于申明是否严格类型校验

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

1. 为什么先输出 `|test|` 之后才输出 `|here is handler|`
2. 为什么在输出最后一个 `|test|` 后还输出2个 `|here is handler|`
3. declare中的for循环怎么分解成低级语句(low-level)

***分析 ：***

- 通过手册中对于 `tick` 中的描述 可以得知： `代码段中解释器每执行 N 条可计时的低级语句就会发生的事件` 那 `低级语句` 怎么理解呢？
- 我们从源码中搜索到 生成 `ZEND_TICKS` 指令的唯一函数是 `zend_do_ticks` (实现在 `zend_compile.c中` ) 发现总共以下几个地方用到: 
```
statement:
  unticked_statement { zend_do_ticks(TSRMLS_C); }
| ...
;
function_declaration_statement:
  unticked_function_declaration_statement { zend_do_ticks(TSRMLS_C); }
;
class_declaration_statement:
  unticked_class_declaration_statement { zend_do_ticks(TSRMLS_C); }
;
```
- 也就是说在 statement, function_declaration_statement, class_declaration_statement 的时候会执行 tick
- function_declaration_statement 表示函数申明, class_declaration_statement 表示类定义
- statement 最麻烦 代码在以下
```
unticked_statement:
  '{' inner_statement_list '}'
| T_IF '(' expr ')' { zend_do_if_cond(&$3, &$4 TSRMLS_CC); } statement { zend_do_if_after_statement(&$4, 1 TSRMLS_CC); } elseif_list else_single { zend_do_if_end(TSRMLS_C); }
| T_IF '(' expr ')' ':' { zend_do_if_cond(&$3, &$4 TSRMLS_CC); } inner_statement_list { zend_do_if_after_statement(&$4, 1 TSRMLS_CC); } new_elseif_list new_else_single T_ENDIF ';' { zend_do_if_end(TSRMLS_C); }
| T_WHILE '(' { $1.u.opline_num = get_next_op_number(CG(active_op_array));  } expr  ')' { zend_do_while_cond(&$4, &$5 TSRMLS_CC); } while_statement { zend_do_while_end(&$1, &$5 TSRMLS_CC); }
| T_DO { $1.u.opline_num = get_next_op_number(CG(active_op_array));  zend_do_do_while_begin(TSRMLS_C); } statement T_WHILE '(' { $5.u.opline_num = get_next_op_number(CG(active_op_array)); } expr ')' ';' { zend_do_do_while_end(&$1, &$5, &$7 TSRMLS_CC); }
| T_FOR
   '('
    for_expr
   ';' { zend_do_free(&$3 TSRMLS_CC); $4.u.opline_num = get_next_op_number(CG(active_op_array)); }
    for_expr
   ';' { zend_do_extended_info(TSRMLS_C); zend_do_for_cond(&$6, &$7 TSRMLS_CC); }
    for_expr
   ')' { zend_do_free(&$9 TSRMLS_CC); zend_do_for_before_statement(&$4, &$7 TSRMLS_CC); }
   for_statement { zend_do_for_end(&$7 TSRMLS_CC); }
| T_SWITCH '(' expr ')' { zend_do_switch_cond(&$3 TSRMLS_CC); } switch_case_list { zend_do_switch_end(&$6 TSRMLS_CC); }
| T_BREAK ';'    { zend_do_brk_cont(ZEND_BRK, NULL TSRMLS_CC); }
| T_BREAK expr ';'  { zend_do_brk_cont(ZEND_BRK, &$2 TSRMLS_CC); }
| T_CONTINUE ';'   { zend_do_brk_cont(ZEND_CONT, NULL TSRMLS_CC); }
| T_CONTINUE expr ';'  { zend_do_brk_cont(ZEND_CONT, &$2 TSRMLS_CC); }
| T_RETURN ';'      { zend_do_return(NULL, 0 TSRMLS_CC); }
| T_RETURN expr_without_variable ';' { zend_do_return(&$2, 0 TSRMLS_CC); }
| T_RETURN variable ';'    { zend_do_return(&$2, 1 TSRMLS_CC); }
| T_GLOBAL global_var_list ';'
| T_STATIC static_var_list ';'
| T_ECHO echo_expr_list ';'
| T_INLINE_HTML   { zend_do_echo(&$1 TSRMLS_CC); }
| expr ';'    { zend_do_free(&$1 TSRMLS_CC); }
| T_UNSET '(' unset_variables ')' ';'
| T_FOREACH '(' variable T_AS
  { zend_do_foreach_begin(&$1, &$2, &$3, &$4, 1 TSRMLS_CC); }
  foreach_variable foreach_optional_arg ')' { zend_do_foreach_cont(&$1, &$2, &$4, &$6, &$7 TSRMLS_CC); }
  foreach_statement { zend_do_foreach_end(&$1, &$4 TSRMLS_CC); }
| T_FOREACH '(' expr_without_variable T_AS
  { zend_do_foreach_begin(&$1, &$2, &$3, &$4, 0 TSRMLS_CC); }
  variable foreach_optional_arg ')' { zend_check_writable_variable(&$6); zend_do_foreach_cont(&$1, &$2, &$4, &$6, &$7 TSRMLS_CC); }
  foreach_statement { zend_do_foreach_end(&$1, &$4 TSRMLS_CC); }
| T_DECLARE { $1.u.opline_num = get_next_op_number(CG(active_op_array)); zend_do_declare_begin(TSRMLS_C); } '(' declare_list ')' declare_statement { zend_do_declare_end(&$1 TSRMLS_CC); }
| ';'  /* empty statement */
| T_TRY { zend_do_try(&$1 TSRMLS_CC); } '{' inner_statement_list '}'
  T_CATCH '(' { zend_initialize_try_catch_element(&$1 TSRMLS_CC); }
  fully_qualified_class_name { zend_do_first_catch(&$7 TSRMLS_CC); }
  T_VARIABLE ')' { zend_do_begin_catch(&$1, &$9, &$11, &$7 TSRMLS_CC); }
  '{' inner_statement_list '}' { zend_do_end_catch(&$1 TSRMLS_CC); }
  additional_catches { zend_do_mark_last_catch(&$7, &$18 TSRMLS_CC); }
| T_THROW expr ';' { zend_do_throw(&$2 TSRMLS_CC); }
| T_GOTO T_STRING ';' { zend_do_goto(&$2 TSRMLS_CC); }
;
```
- 根据以上定义可知 statement 包括:
  - 简单语句：空语句（就一个；号），return,break,continue,throw, goto,global,static,unset,echo, 内置的HTML文本，分号结束的表达式等均算一个语句。
  - 复合语句：完整的if/elseif,while,do...while,for,foreach,switch,try...catch等算一个语句。
  - 语句块：{} 括出来的语句块。
  - 最后特别的：在php7之前 declare块本身也算一个语句，但是PHP7以后将其排除在外。
- 所有的statement, function_declare_statement, class_declare_statement就构成了所谓的低级语句(low-level statement)。
