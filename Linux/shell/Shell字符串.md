# Shell 字符串

字符串是shell编程中最常用最有用的数据类型（除了数字和字符串，也没啥其它类型好用了），字符串可以用单引号，也可以用双引号，也可以不用引号。

## 单引号

	str='this is a string'

单引号字符串的限制：

- 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的；
- 单引号字串中不能出现单引号（对单引号使用转义符后也不行）。

## 双引号

	your_name='jack'
	str="Hello, I know you are \"$your_name\"! \n"

双引号的优点：

- 双引号里可以有变量
- 双引号里可以出现转义字符

## 拼接字符串

	your_name="jack"
	greeting="hello, "$your_name" !"
	greeting_1="hello, ${your_name} !"
	echo $greeting $greeting_1

## 获取字符串长度

	str="abcd"
	echo `expr length $str`   # 4
	echo ${#str}       		  # 4
	echo `expr "$str" : ".*"` # 4

## 提取子字符串

	string="alibaba is a great company"
	echo ${string:1:4} #输出liba

	str="abcdef"
	echo `expr substr "$str" 1 3`  # 从第一个位置开始取3个字符， abc
	echo `expr substr "$str" 2 5`  # 从第二个位置开始取5个字符， bcdef
	echo `expr substr "$str" 4 5`  # 从第四个位置开始取5个字符， def
	
	echo ${str:2}           # 从第二个位置开始提取字符串， cdef
	echo ${str:2:3}         # 从第二个位置开始提取3个字符, cde
	echo ${str:(-6):5}      # 从倒数第六个字符开始提取5个字符, abcde
	echo ${str:(-4):3}      # 从倒数第四个字符开始提取3个字符, cde

## 查找子字符串

	string=abcABC123ABCabc
	echo `expr index "$stringZ" C12` # 6
	echo `expr index "$stringZ" 1c`  # 3
	
	string="alibaba is a great company"
	echo `expr index "$string" is`   # 3
	
	str="abc"
	echo `expr index $str "a"`       # 1
	echo `expr index $str "b"`       # 2
	echo `expr index $str "x"`       # 0
	echo `expr index $str ""`        # 0

上面的查字符串你应该有疑问，`C12`查的是`C`的位置没问题，是6,`1c`为什么是3呢，`is`为什么是3呢？

Shell的查字符串不是按整串查的，而是把待查的字符串分成一个一个的，先查到哪个就是它了，`C12`先查到有`C`，所以是6，`1c`先找到`c`所以是3，`is`先找到`i`所以是3.


## 截取子串

	str="abbc,def,ghi,abcjkl"
	echo ${str#a*c}     # ,def,ghi,abcjkl       一个井号(#) 表示从左边截取掉最短的匹配 (这里把abbc字串去掉），(*) 代表任意字符
	echo ${str##a*c}    # jkl                   两个井号(##) 表示从左边截取掉最长的匹配 (这里把abbc,def,ghi,abc字串去掉)
	echo ${str#"a*c"}   # abbc,def,ghi,abcjkl   因为str中没有"a*c"子串
	echo ${str##"a*c"}  # abbc,def,ghi,abcjkl   同上
	echo ${str#*a*c*}   # ,def,ghi,abcjkl       左边开始到第一个c结束（最短匹配去掉）
	echo ${str##*a*c*}  # 空                    最长匹配
	echo ${str#d*f}     # abbc,def,ghi,abcjkl   此字符串不是以d开头，所以无需去任何字符               
	echo ${str#*d*f}    # ,ghi,abcjkl           *代表任意字符到d再到f，匹配最短，所以去掉abbc,def字符串
	
	echo ${str%a*l}     # abbc,def,ghi          一个百分号(%)表示从右边截取最短的匹配，去掉 abcjkl 字符串
	echo ${str%%b*l}    # a                     两个百分号表示(%%)表示从右边截取最长的匹配
	echo ${str%a*c}     # abbc,def,ghi,abcjkl   从右边找不到c结尾的，无需去任何字符串

可以这样记忆, 井号（#）通常用于表示一个数字，它是放在前面的；百分号（%）写在数字的后面; 或者这样记忆，在键盘布局中，井号(#)总是位于百分号（%）的左边(即前面)    :-)

从前匹配，第一个必须匹配上，否则无需去任何字符，同理，从后匹配，最后一个必须匹配上。

## 字符串替换

	str="apple, tree, apple tree"
	echo ${str/apple/APPLE}   # 替换第一次出现的apple
	echo ${str//apple/APPLE}  # 替换所有apple
	 
	echo ${str/#apple/APPLE}  # 如果字符串str以apple开头，则用APPLE替换它
	echo ${str/%apple/APPLE}  # 如果字符串str以apple结尾，则用APPLE替换它

## 字符串比较

	[[ "a.txt" == a* ]]        # 逻辑真 (pattern matching)
	[[ "a.txt" =~ .*\.txt ]]   # 逻辑真 (regex matching)
	[[ "abc" == "abc" ]]       # 逻辑真 (string comparision) 
	[[ "11" < "2" ]]           # 逻辑真 (string comparision), 按ascii值比较

## 字符串方法汇总表

<table>
<tbody>
	<tr>
		<th>
			表达式
		</th>
		<th>
			含义
		</th>
	</tr>
	<tr>
		<td>
			${var}
		</td>
		<td>
			变量var的值, 与$var相同
		</td>
	</tr>
	<tr>
		<td>
			${var-DEFAULT}
		</td>
		<td>
			如果var没有被声明, 那么就以$DEFAULT作为其值 *
		</td>
	</tr>
	<tr>
		<td>
			${var:-DEFAULT}
		</td>
		<td>
			如果var没有被声明, 或者其值为空, 那么就以$DEFAULT作为其值 *
		</td>
	</tr>
	<tr>
		<td>
			${var=DEFAULT}
		</td>
		<td>
			如果var没有被声明, 那么就以$DEFAULT作为其值 *
		</td>
	</tr>
	<tr>
		<td>
			${var:=DEFAULT}
		</td>
		<td>
			如果var没有被声明, 或者其值为空, 那么就以$DEFAULT作为其值 *
		</td>
	</tr>
	<tr>
		<td>
			${var+OTHER}
		</td>
		<td>
			如果var声明了, 那么其值就是$OTHER, 否则就为null字符串
		</td>
	</tr>
	<tr>
		<td>
			${var:+OTHER}
		</td>
		<td>
			如果var被设置了, 那么其值就是$OTHER, 否则就为null字符串
		</td>
	</tr>
	<tr>
		<td>
			${var?ERR_MSG}
		</td>
		<td>
			如果var没被声明, 那么就打印$ERR_MSG *
		</td>
	</tr>
	<tr>
		<td>
			${var:?ERR_MSG}
		</td>
		<td>
			如果var没被设置, 那么就打印$ERR_MSG *
		</td>
	</tr>
	<tr>
		<td>
			${!varprefix*}
		</td>
		<td>
			匹配之前所有以varprefix开头进行声明的变量
		</td>
	</tr>
	<tr>
		<td>
			${!varprefix@}
		</td>
		<td>
			匹配之前所有以varprefix开头进行声明的变量
		</td>
	</tr>
	<tr>
		<td>
			${#string}
		</td>
		<td>
			$string的长度
		</td>
	</tr>
	<tr>
		<td>
			${string:position}
		</td>
		<td>
			在$string中, 从位置$position开始提取子串
		</td>
	</tr>
	<tr>
		<td>
			${string:position:length}
		</td>
		<td>
			在$string中, 从位置$position开始提取长度为$length的子串
		</td>
	</tr>
	<tr>
		<td>
			${string#substring}
		</td>
		<td>
			从变量$string的开头, 删除最短匹配$substring的子串
		</td>
	</tr>
	<tr>
		<td>
			${string##substring}
		</td>
		<td>
			从变量$string的开头, 删除最长匹配$substring的子串
		</td>
	</tr>
	<tr>
		<td>
			${string%substring}
		</td>
		<td>
			从变量$string的结尾, 删除最短匹配$substring的子串
		</td>
	</tr>
	<tr>
		<td>
			${string%%substring}
		</td>
		<td>
			从变量$string的结尾, 删除最长匹配$substring的子串
		</td>
	</tr>
	<tr>
		<td>
			${string/substring/replacement}
		</td>
		<td>
			使用$replacement, 来代替第一个匹配的$substring
		</td>
	</tr>
	<tr>
		<td>
			${string//substring/replacement}
		</td>
		<td>
			使用$replacement, 代替<em>所有</em>匹配的$substring
		</td>
	</tr>
	<tr>
		<td>
			${string/#substring/replacement}
		</td>
		<td>
			如果$string的<em>前缀</em>匹配$substring, 那么就用$replacement来代替匹配到的$substring
		</td>
	</tr>
	<tr>
		<td>
			${string/%substring/replacement}
		</td>
		<td>
			如果$string的<em>后缀</em>匹配$substring, 那么就用$replacement来代替匹配到的$substring
		</td>
	</tr>
</tbody>
</table>

字符串还有好多内容可写，用到的时候再查吧。

## 参考资料

- [shell 字符串处理汇总（查找，替换等等）](http://blog.chinaunix.net/uid-124706-id-3475936.html)
- [Bash Shell字符串操作小结](http://www.cnblogs.com/frydsh/p/3261012.html)
- [Shell字符串](http://c.biancheng.net/cpp/view/7001.html)