# Shell 数组 函数

Shell在编程方面比Windows批处理强大很多，无论是在循环、运算，并且定义函数也非常方便好用。

## 数组

bash支持一维数组（不支持多维数组），并且没有限定数组的大小。类似与C语言，数组元素的下标由0开始编号。获取数组中的元素要利用下标，下标可以是整数或算术表达式，其值应大于或等于0。

### 定义数组

在Shell中，用括号来表示数组，数组元素用“空格”符号分割开。定义数组的一般形式为：

	array_name=(value1 ... valuen)

例如：

	array_name=(value0 value1 value2 value3)

或者：

	array_name=(
	value0
	value1
	value2
	value3
	)

还可以单独定义数组的各个分量：

	array_name[0]=value0
	array_name[1]=value1
	array_name[2]=value2

可以不使用连续的下标，而且下标的范围没有限制。

### 读取数组

读取数组元素值的一般格式是：

    ${array_name[index]}

例如：

	value=${array_name[2]}

使用 `@` 或 `*` 可以获取数组中的所有元素，例如：

	#!/bin/sh
	NAME[0]="Zara"
	NAME[1]="Qadir"
	NAME[2]="Mahnaz"
	NAME[3]="Ayan"
	NAME[4]="Daisy"
	echo "First Method: ${NAME[*]}"
	echo "Second Method: ${NAME[@]}"

运行脚本：

	First Method: Zara Qadir Mahnaz Ayan Daisy
	Second Method: Zara Qadir Mahnaz Ayan Daisy

### 获取数组的长度

获取数组长度的方法与获取字符串长度的方法相同，例如：

	# 取得数组元素的个数
	length=${#array_name[@]}
	# 或者
	length=${#array_name[*]}
	# 取得数组单个元素的长度
	lengthn=${#array_name[n]}

### 循环数组

循环数组使用for循环：

	for name in $array
	do
		echo $name
	done

## 函数

函数可以让我们将一个复杂功能划分成若干模块，让程序结构更加清晰，代码重复利用率更高。像其他编程语言一样，Shell 也支持函数。Shell 函数必须先定义后使用。

### 定义函数

Shell 函数的定义格式如下：

	function_name () {
	    list of commands
	    [ return value ]
	}

如果你愿意，也可以在函数名前加上关键字 function：

	function function_name () {
	    list of commands
	    [ return value ]
	}

### 函数返回值

函数返回值，可以显式增加return语句；如果不加，会将最后一条命令运行结果作为返回值。

Shell 函数返回值只能是整数，一般用来表示函数执行成功与否，0表示成功，其他值表示失败。如果 return 其他数据，比如一个字符串，往往会得到错误提示：“numeric argument required”。

如果一定要让函数返回字符串，那么可以先定义一个变量，用来接收函数的计算结果，脚本在需要的时候访问这个变量来获得函数返回值。

### 函数的调用

	#!/bin/bash
	
	Hello () {
	   echo "Hello World!"
	}
	
	Hello

运行结果：

	Hello World!

调用函数只需要给出函数名，不需要加括号。

如果你希望直接从终端调用函数，可以将函数定义在主目录下的 .profile 文件，这样每次登录后，在命令提示符后面输入函数名字就可以立即调用。

再来看一个带有return语句的函数：

	#!/bin/bash
	funWithReturn(){
	    echo "The function is to get the sum of two numbers..."
	    echo -n "Input first number: "
	    read aNum
	    echo -n "Input another number: "
	    read anotherNum
	    echo "The two numbers are $aNum and $anotherNum !"
	    return $(($aNum+$anotherNum))
	}
	funWithReturn
	# Capture value returnd by last command
	ret=$?
	echo "The sum of two numbers is $ret !"

运行结果：

	The function is to get the sum of two numbers...
	Input first number: 25
	Input another number: 50
	The two numbers are 25 and 50 !
	The sum of two numbers is 75 !

函数返回值在调用该函数后通过 `$?` 来获得。

再来看一个函数嵌套的例子：

	#!/bin/bash
	# Calling one function from another
	number_one () {
	   echo "This is one."
	   number_two
	}
	number_two () {
	   echo "This is two."
	}
	number_one

运行结果：

	This is one.
	This is two.

### 函数参数

在Shell中，调用函数时可以向其传递参数。在函数体内部，通过 $n 的形式来获取参数的值，例如，$1表示第一个参数，$2表示第二个参数...

	#!/bin/bash
	funWithParam(){
	    echo "The value of the first parameter is $1 !"
	    echo "The value of the second parameter is $2 !"
	    echo "The value of the tenth parameter is $10 !"
	    echo "The value of the tenth parameter is ${10} !"
	    echo "The value of the eleventh parameter is ${11} !"
	    echo "The amount of the parameters is $# !"  # 参数个数
	    echo "The string of the parameters is $* !"  # 传递给函数的所有参数
	}
	funWithParam 1 2 3 4 5 6 7 8 9 34 73

运行结果：

	The value of the first parameter is 1 !
	The value of the second parameter is 2 !
	The value of the tenth parameter is 10 !
	The value of the tenth parameter is 34 !
	The value of the eleventh parameter is 73 !
	The amount of the parameters is 11 !
	The string of the parameters is 1 2 3 4 5 6 7 8 9 34 73 !"

注意，`$10` 不能获取第十个参数，获取第十个参数需要${10}。当 n>=10 时，需要使用`${n}`来获取参数。

### 删除函数

像删除变量一样，删除函数也可以使用 unset 命令，不过要加上 .f 选项，如下所示：

	unset .f function_name

## 参考资料

- [Shell函数：Shell函数返回值、删除函数、在终端调用函数](http://c.biancheng.net/cpp/view/7011.html)
- [Shell数组：shell数组的定义、数组长度](http://c.biancheng.net/cpp/view/7002.html)
- [Shell函数参数](http://c.biancheng.net/cpp/view/2491.html)