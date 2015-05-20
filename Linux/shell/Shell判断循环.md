# Shell 判断 循环

学习任何语言好像都离不开条件和循环吧，Shell当然也不例外。

## if 判断

if 语句通过关系运算符判断表达式的真假来决定执行哪个分支。Shell 有三种 if ... else 语句：

- if ... fi 语句；
- if ... else ... fi 语句；
- if ... elif ... else ... fi 语句。

### if ... else 语句

	if [ expression ]
	then
	   Statement(s) to be executed if expression is true
	fi

如果 expression 返回 true，then 后边的语句将会被执行；如果返回 false，不会执行任何语句。

最后必须以 fi 来结尾闭合 if，fi 就是 if 倒过来拼写，后面也会遇见。

注意：expression 和方括号([ ])之间必须有空格，否则会有语法错误。

	#!/bin/sh
	a=10
	b=20
	if [ $a == $b ]
	then
	   echo "a is equal to b"
	fi
	if [ $a != $b ]
	then
	   echo "a is not equal to b"
	fi

执行脚本：

	a is not equal to b

### if ... else ... fi 语句

	if [ expression ]
	then
	   Statement(s) to be executed if expression is true
	else
	   Statement(s) to be executed if expression is not true
	fi

如果 expression 返回 true，那么 then 后边的语句将会被执行；否则，执行 else 后边的语句。

### if ... elif ... else ... fi 语句

对多个条件进行判断，语法为：

	if [ expression1 ]
	then
	   Statement(s) to be executed if expression1 is true
	elif [ expression2 ]
	then
	   Statement(s) to be executed if expression2 is true
	elif [ expression3 ]
	then
	   Statement(s) to be executed if expression3 is true
	else
	   Statement(s) to be executed if no expression is true
	fi

哪一个 expression 的值为 true，就执行哪个 expression 后面的语句；如果都为 false，那么不执行任何语句。

if ... else 语句也可以写成一行，以命令的方式来运行，像这样：

	if test $[2*3] -eq $[1+5]; then echo 'The two numbers are equal!'; fi;

`test` 命令用于检查某个条件是否成立，与方括号([ ])类似。

## case 判断

case ... esac 与其他语言中的 switch ... case 语句类似，是一种多分枝选择结构。

case 语句匹配一个值或一个模式，如果匹配成功，执行相匹配的命令。case语句格式如下：

	case 值 in
	模式1)
	    command1
	    command2
	    command3
	    ;;
	模式2）
	    command1
	    command2
	    command3
	    ;;
	*)
	    command1
	    command2
	    command3
	    ;;
	esac

每一模式必须以右括号结束，取值可以为变量或常数，匹配发现取值符合某一模式后，其间所有命令开始执行直至 `;;`。`;;` 与其他语言中的 `break` 类似，意思是跳到整个 case 语句的最后。

取值将检测匹配的每一个模式。一旦模式匹配，则执行完匹配模式相应命令后不再继续其他模式。如果无一匹配模式，使用星号 ` * ` 捕获该值，再执行后面的命令。

下面的脚本提示输入1到4，与每一种模式进行匹配：

	echo 'Input a number between 1 to 4'
	echo 'Your number is:\c'
	read aNum
	case $aNum in
	    1)  echo 'You select 1'
	    ;;
	    2)  echo 'You select 2'
	    ;;
	    3)  echo 'You select 3'
	    ;;
	    4)  echo 'You select 4'
	    ;;
	    *)  echo 'You do not select a number between 1 to 4'
	    ;;
	esac

输入不同的内容，会有不同的结果，例如：

	Input a number between 1 to 4
	Your number is:3
	You select 3

## for 循环

for循环一般格式为：

	for 变量 in 列表
	do
	    command1
	    command2
	    ...
	    commandN
	done

列表是一组值（数字、字符串等）组成的序列，每个值通过空格分隔。每循环一次，就将列表中的下一个值赋给变量。

例如，顺序输出当前列表中的数字：

	for loop in 1 2 3 4 5
	do
	    echo "The value is: $loop"
	done

运行结果：

	The value is: 1
	The value is: 2
	The value is: 3
	The value is: 4
	The value is: 5

顺序输出字符串中的字符：

	for str in 'This is a string'
	do
	    echo $str
	done

运行结果：

	This is a string

不加引号：

	for str in This is a string
	do
	    echo $str
	done

运行结果：

	This
	is
	a
	string

显示主目录下以 .bash 开头的文件：

	for FILE in ~/.bash*
	do
	   echo $FILE
	done

运行结果：

	/home/panda/.bash_history
	/home/panda/.bash_logout
	/home/panda/.bash_profile
	/home/panda/.bashrc

## while 循环

while循环用于不断执行一系列命令，也用于从输入文件中读取数据；命令通常为测试条件。其格式为：

	while command
	do
	   Statement(s) to be executed if command is true
	done

命令执行完毕，控制返回循环顶部，从头开始直至测试条件为假。

以下是一个基本的while循环，测试条件是：如果COUNTER小于5，那么返回 true。COUNTER从0开始，每次循环处理时，COUNTER加1。运行上述脚本，返回数字1到5，然后终止。

	COUNTER=0
	while [ $COUNTER -lt 5 ]
	do
	    COUNTER='expr $COUNTER+1'
	    echo $COUNTER
	done

运行结果：

	1
	2
	3
	4
	5

while循环可用于读取键盘信息。下面的例子中，输入信息被设置为变量FILM，按<` Ctrl+D `>结束循环。

	echo 'type <CTRL-D> to terminate'
	echo -n 'enter your most liked film: '
	while read FILM
	do
	    echo "Yeah! great film the $FILM"
	done

运行结果：

	type <CTRL-D> to terminate
	enter your most liked film: Sound of Music
	Yeah! great film the Sound of Music

## until 循环

until 循环执行一系列命令直至条件为 true 时停止。until 循环与 while 循环在处理方式上刚好相反。一般while循环优于until循环，但在某些时候，也只是极少数情况下，until 循环更加有用。

until 循环格式为：

	until command
	do
	   Statement(s) to be executed until command is true
	done

command 一般为条件表达式，如果返回值为 false，则继续执行循环体内的语句，否则跳出循环。

例如，使用 until 命令输出 0 ~ 9 的数字：

	#!/bin/bash
	a=0
	until [ ! $a -lt 10 ]
	do
	   echo $a
	   a=`expr $a + 1`
	done

运行结果：

	0
	1
	2
	3
	4
	5
	6
	7
	8
	9

## 跳出循环

在循环过程中，有时候需要在未达到循环结束条件时强制跳出循环，像大多数编程语言一样，Shell也使用 break 和 continue 来跳出循环。

### break 命令

break命令允许跳出所有循环（终止执行后面的所有循环）。

下面的例子中，脚本进入死循环直至用户输入数字大于5。要跳出这个循环，返回到shell提示符下，就要使用break命令。

	#!/bin/bash
	while :
	do
	    echo -n "Input a number between 1 to 5: "
	    read aNum
	    case $aNum in
	        1|2|3|4|5) echo "Your number is $aNum!"
	        ;;
	        *) echo "You do not select a number between 1 to 5, game is over!"
	            break
	        ;;
	    esac
	done

在嵌套循环中，break 命令后面还可以跟一个整数，表示跳出第几层循环。例如：

	break n

表示跳出第 n 层循环。

下面是一个嵌套循环的例子，如果 var1 等于 2，并且 var2 等于 0，就跳出循环：

	#!/bin/bash
	for var1 in 1 2 3
	do
	   for var2 in 0 5
	   do
	      if [ $var1 -eq 2 -a $var2 -eq 0 ]
	      then
	         break 2
	      else
	         echo "$var1 $var2"
	      fi
	   done
	done

如上，break 2 表示直接跳出外层循环。运行结果：

	1 0
	1 5

### continue 命令

continue命令与break命令类似，只有一点差别，它不会跳出所有循环，仅仅跳出当前循环。

对上面的例子进行修改：

	#!/bin/bash
	while :
	do
	    echo -n "Input a number between 1 to 5: "
	    read aNum
	    case $aNum in
	        1|2|3|4|5) echo "Your number is $aNum!"
	        ;;
	        *) echo "You do not select a number between 1 to 5!"
	            continue
	            echo "Game is over!"
	        ;;
	    esac
	done

程序会一直运行。

同样，continue 后面也可以跟一个数字，表示跳出第几层循环。

再看一个 continue 的例子：

	#!/bin/bash
	NUMS="1 2 3 4 5 6 7"
	for NUM in $NUMS
	do
	   Q=`expr $NUM % 2`
	   if [ $Q -eq 0 ]
	   then
	      echo "Number is an even number!!"
	      continue
	   fi
	   echo "Found odd number"
	done

运行结果：

	Found odd number
	Number is an even number!!
	Found odd number
	Number is an even number!!
	Found odd number
	Number is an even number!!
	Found odd number

## 参考资料

- [Shell break和continue命令](http://c.biancheng.net/cpp/view/7010.html)
- [Shell until循环](http://c.biancheng.net/cpp/view/7009.html)
- [Shell while循环](http://c.biancheng.net/cpp/view/7008.html)
- [Shell for循环](http://c.biancheng.net/cpp/view/7007.html)