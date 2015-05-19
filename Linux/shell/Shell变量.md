# Shell 变量

Shell支持自定义变量。

## 定义变量

定义变量时，变量名不加美元符号（`$`），如：

	myName="panda"

**注意：**变量名和等号之间不能有空格，这可能和你熟悉的所有编程语言都不一样。同时，变量名的命名须遵循如下规则：

- 首个字符必须为字母（a-z，A-Z）。
- 中间不能有空格，可以使用下划线（_）。
- 不能使用标点符号。
- 不能使用bash里的关键字（可用`help`命令查看保留关键字）。

## 使用变量

使用一个定义过的变量，只要在变量名前面加美元符号（`$`）即可，如：

	myName="panda"
	echo $myName
	echo ${myName}

变量名外面的花括号是可选的，加不加都行，加花括号是为了帮助解释器识别变量的边界。

例如：

	soft="java"
	echo "$javascript"

解释器会把 `$javascript` 当做一个变量，输出为空。

推荐给所有变量加上花括号，这是个好的编程习惯。

## 重新定义变量

已定义的变量，可以被重新定义，如：

	myName="panda"
	echo ${myName}
	myName="lion"
	echo ${myName}

分别输出：

	panda
	lion

**注意：**重新定义变量时，不需要美元符号，只有使用变量的时候才加美元符号（这是比较常犯的错误）。

## 只读变量

使用 `readonly` 命令可以将变量定义为只读变量，只读变量的值不能被改变。

下面的例子尝试更改只读变量，结果报错：

	#!/bin/bash

	myName="panda"
	readonly myName
	myName="lion"

运行脚本会报错：

	./test.sh: line 5: myName: readonly variable

## 删除变量

使用 `unset` 命令可以删除变量。语法：

	unset myName

## 特殊变量

变量名只能包含数字、字母和下划线，因为某些包含其他字符的变量有特殊含义，这样的变量被称为特殊变量。

例如：`$` 表示当前shell进程的ID，即pid。

	#!/bin/bash
	echo $$

输出：

	3680

> 特殊变量列表

<table>
<tr>
<th>变量</th><th>含义</th>
</tr>
<tr><td>$0</td><td>当前脚本的文件名</td></tr>
<tr><td>$n</td><td>传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。</td></tr>
<tr><td>$#</td><td>传递给脚本或函数的参数个数。</td></tr>
<tr><td>$*</td><td>传递给脚本或函数的所有参数。</td></tr>
<tr><td>$@</td><td>传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同，下面将会讲到。</td></tr>
<tr><td>$?</td><td>上个命令的退出状态，或函数的返回值。</td></tr>
<tr><td>$$</td><td>当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。</td></tr>
</table>

例如：获取脚本参数

	#!/bin/bash
	echo "File Name: $0"
	echo "First Parameter : $1"
	echo "Second Parameter : $2"
	echo "Paramters: $@"
	echo "Paramters: $*"
	echo "Total Number of Parameters : $#"
	echo "Shell pid: $$"

运行：

	./test.sh tom jack jim

输出：

	File Name: ./test.sh
	First Parameter : tom
	Second Parameter : jack
	Paramters: tom jack jim
	Paramters: tom jack jim
	Total Number of Parameters : 3
	Shell pid: 3783

`$*` 和 `$@` 都表示传递给函数或脚本的所有参数，不被双引号(`" "`)包含时，都以"$1" "$2" … "$n" 的形式输出所有参数。但他们被双引号（`" "`）包含时，`$*` 会将所有参数作为一个整体，以"$1 $2 … $n"的形式输出所有参数；`$@` 会将各个参数分开，以"$1" "$2" … "$n" 的形式输出所有参数。

看例子：

	#!/bin/bash

	echo "print each param from \$*"
	for var in $*
	do
	    echo "$var"
	done
	echo "print each param from \$@"
	for var in $@
	do
	    echo "$var"
	done
	echo "print each param from \"\$*\""
	for var in "$*"
	do
	    echo "$var"
	done
	echo "print each param from \"\$@\""
	for var in "$@"
	do
	    echo "$var"
	done

运行脚本：

	./test.sh a b c

结果：

	print each param from $*
	a
	b
	c
	print each param from $@
	a
	b
	c
	print each param from "$*"
	a b c
	print each param from "$@"
	a
	b
	c

可见，不加 `" "` 俩者是一样的。


## 变量类型

运行Shell时，会同时存在三种变量：

### 局部变量

局部变量在脚本或命令中定义，仅在当前shell实例中有效，其他shell启动的程序不能访问局部变量。

### 环境变量

所有的程序，包括shell启动的程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候shell脚本也可以定义环境变量。

### shell变量

shell变量是由shell程序设置的特殊变量。shell变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了shell的正常运行。

## 参考资料

- [Shell变量：Shell变量的定义、删除变量、只读变量、变量类型](http://c.biancheng.net/cpp/view/6999.html)
- [Shell特殊变量：Shell $0, $#, $*, $@, $?, $$和命令行参数](http://c.biancheng.net/cpp/view/2739.html)






