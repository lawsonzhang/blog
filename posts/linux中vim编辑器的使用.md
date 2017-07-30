<link href="https://cdn.bootcss.com/bootstrap/3.3.0/css/bootstrap-theme.css" rel="stylesheet">
### __纯文本文件__
> ###### 什么纯文本文件？纯文本文件就是文件的记录是0与1，通过编码系统将这些数字转换成我们认识的文字
>
>

### __Linux下的文本编辑器__
> ###### Emacs、Pico、Nano、Joe、Vim等
>
>

### __模式__
> ###### 一般模式、编辑模式、命令行模式
>
>

![模式关系](/static/img/landing/vim.png)
`
`
### __快捷键说明__


#### 一般模式：

<table class="table table-condensed table-bordered" id="part0">
	<tr><td><h4>快捷键</h4></td><td><h4>描述</h4></td></tr>
	<tr><td colspan="2" class="text-center"><h4><a href="#part0">移动光标的方法</a></h4></td></tr>
	<tr><td>h 或 ←</td><td>光标向左移动一个字符</td></tr>
	<tr><td>j 或 ↓</td><td>光标向下移动一个字符</td></tr>
	<tr><td>k 或 ↑</td><td>光标向上移动一个字符</td></tr>
	<tr><td>l 或 →</td><td>光标向右移动一个字符</td></tr>
	<tr><td>{n} + [方向键]</td><td>光标向某个方向移动 n 次</td></tr>
	<tr><td>[ctrl] + f / PageDown</td><td>屏幕向下移动一页</td></tr>
	<tr><td>[ctrl] + b / PageUp</td><td>屏幕向上移动一页</td></tr>
	<tr><td>[ctrl] + d</td><td>屏幕向下移动半页</td></tr>
	<tr><td>[ctrl] + u</td><td>屏幕向上移动半页</td></tr>
	<tr><td>+</td><td>光标移动到非空格符的下一行</td></tr>
	<tr><td>-</td><td>光标移动到非空格符的上一行</td></tr>
	<tr><td>{n} Space</td><td>光标向右移动 n 个字符</td></tr>
	<tr><td>0 / Home</td><td>光标移动到行最前</td></tr>
	<tr><td>$ / End</td><td>光标移动到行最后</td></tr>
	<tr><td>H</td><td>光标移动到屏幕最上方的第一行的第一个字符</td></tr>
	<tr><td>M</td><td>光标移动到屏幕中央的那行的第一个字符</td></tr>
	<tr><td>L</td><td>光标移动到屏幕最下方的哪一行的第一个字符</td></tr>
	<tr><td>G</td><td>光标移动到该文件的最后一行</td></tr>
	<tr><td>{n} G</td><td>光标移动到这个文件的第 n 行</td></tr>
	<tr id="part1"><td>g g</td><td>光标移动到这个文件的第一行</td></tr>
	<tr><td>{n} Enter</td><td>光标向下移动n行</td></tr>
	<tr><td colspan="2" class="text-center"><h4><a href="#part1">查找与替换</a></h4></td></tr>
	<tr><td>/ {word}</td><td>向下寻找一个名为word的字符串</td></tr>
	<tr><td>? {word}</td><td>向上寻找一个名为word的字符串</td></tr>
	<tr><td>n</td><td>重复前一个查找的操作</td></tr>
	<tr><td>N</td><td>反向进行前一个查找的操作</td></tr>
	<tr><td>:{n1},{n2}s/{word1}/{word2}/g</td><td>在第n1到n2行之间寻找word1这个字符串，并且将该字符串替换为word2</td></tr>
	<tr id="part2"><td>:1,$s/{word1}/{word2}/g</td><td>在第一到最后一行之间寻找word1这个字符串，并且将该字符串替换为word2</td></tr>
	<tr><td>:1,$s/{word1}/{word2}/gc</td><td>在第一到最后一行之间寻找word1这个字符串，并且将该字符串替换为word2。并且在替换前显示字符给用户确认</td></tr>
	<tr><td colspan="2" class="text-center"><h4><a href="#part2">删除、复制与黏贴</a></h4></td></tr>
	<tr><td>x,X</td><td>在一行字符中，x为向后删除一个字符(相当于Del)；X为向前删除一个字符(相当于Backspace)</td></tr>
	<tr><td>{n} x</td><td>连续向后删除n个字符</td></tr>
	<tr><td>d d</td><td>删除光标所在的一整行</td></tr>
	<tr><td>{n} d d</td><td>删除光标所在的向下n行</td></tr>
	<tr><td>d 1 G</td><td>删除光标所在到第一行的所有数据</td></tr>
	<tr><td>d G</td><td>删除从光标所在到最后一行的所有数据</td></tr>
	<tr><td>d $</td><td>删除从光标所在处到该行的最后一个字符</td></tr>
	<tr><td>d 0</td><td>删除从光标所在处到该行的最前面一个字符</td></tr>
	<tr><td>y y</td><td>复制光标所在的那一行</td></tr>
	<tr><td>{n} y y</td><td>复制光标所在的向下n行</td></tr>
	<tr><td>y 1 G</td><td>复制光标所在行到第一行的所有数据</td></tr>
	<tr><td>y G</td><td>复制光标所在行到最后一行的所有数据</td></tr>
	<tr><td>y 0</td><td>复制光标所在所在处到该行行首的所有数据</td></tr>
	<tr id="part3"><td>y $</td><td>复制光标所在所在处到该行行尾的所有数据</td></tr>
	<tr><td>p,P</td><td>p为将已经复制的数据在光标下一行黏贴；P则是黏贴在上一行</td></tr>
	<tr><td colspan="2" class="text-center"><h4><a href="#part3">移动光标的方法</a></h4></td></tr>
	<tr><td>J</td><td>将光标所在行的数据与下一行结合成同一行</td></tr>
	<tr><td>{n} c [方向键]</td><td>重复向某个方向删除n个数据</td></tr>
	<tr><td>u</td><td>复原上一个操作</td></tr>
	<tr><td>[Ctrl] + r</td><td>重做上一个操作</td></tr>
	<tr><td>.</td><td>重复前一个操作</td></tr>
</table>


#### 编辑模式：

<table class="table table-condensed table-bordered" id="part4">
	<tr><td><h4>快捷键</h4></td><td><h4>描述</h4></td></tr>
	<tr><td colspan="2" class="text-center"><h4><a href="#part4">进入插入或替换的编辑模式</a></h4></td></tr>
	<tr><td>i</td><td>从光标所在处插入</td></tr>
	<tr><td>I</td><td>从光标所在行的第一个非空格字符处插入</td></tr>
	<tr><td>a</td><td>从目前光标所在的下一个字符处插入</td></tr>
	<tr><td>A</td><td>从目前光标所在行的最后一个字符处插入</td></tr>
	<tr><td>o</td><td>在光标所在的下一行插入新的一行</td></tr>
	<tr><td>O</td><td>在光标所在的上一行插入新的一行</td></tr>
	<tr><td colspan="2" class="text-center">替换模式</td></tr>
	<tr><td>r</td><td>替换光标所在的那个字符一次</td></tr>
	<tr><td>R</td><td>一直往下替换光标所在的那个字符，按Esc终止</td></tr>
</table>


#### 命令行模式：

<table class="table table-condensed table-bordered" id="part5">
	<tr><td><h4>快捷键</h4></td><td><h4>描述</h4></td></tr>
	<tr><td colspan="2" class="text-center"><h4><a href="#part5">命令行的保存、离开等命令</a></h4></td></tr>
	<tr><td>:w</td><td>将编辑的数据写入硬盘文件中</td></tr>
	<tr><td>:w!</td><td>若属性为只读时，强制写入。前提是用户有权限</td></tr>
	<tr><td>:q</td><td>离开vi</td></tr>
	<tr><td>:q!</td><td>强制离开不保存</td></tr>
	<tr><td>:wq</td><td>保存后离开</td></tr>
	<tr><td>ZZ</td><td>保存后离开。若文件未变更，则不保存离开</td></tr>
	<tr><td>:w [filename]</td><td>另存为新文件</td></tr>
	<tr><td>:r [filename]</td><td>读入另一个文件的数据。添加到光标所在行的后面</td></tr>
	<tr><td>:{n1},{n2} w [filename]</td><td>将 n1 到 n2 行之间的内容另存为</td></tr>
	<tr><td>:! [command]</td><td>暂时离开vi，并且执行 command 的显示结果</td></tr>
	<tr><td colspan="2" class="text-center">vim环境的更改</td></tr>
	<tr><td>:set nu</td><td>显示行号</td></tr>
	<tr><td>:set nonu</td><td>取消行号</td></tr>
	<tr><td>ZZ</td><td>在光标所在的上一行插入新的一行</td></tr>
</table>