# Shell 输入 输出

Shell 支持标准输入，输出及关联文件的输入输出。

## read 获取用户输入

read 命令接收标准输入的输入，或其它文件描述符的输入。得到输入后，read 命令将数据输入放入一个标准变量中。

### read 参数列表

read 命令有很多参数可选：

<table>
<tbody>
<tr><th>格 式</th><th>含 义</th></tr>
<tr><td>read answer</td><td>从标准输入读取一行并赋值给变量answer</td></tr>
<tr><td>read first middle last</td><td>从标准输入读取一行，直至遇到第一个换行符。把用户键入的第一个词存到变量first中，第二个词存到middle,把该行的剩余部分保存到变量last中</td></tr>
<tr><td>read [没其他参数]</td><td>标准输入读取一行并赋值给内置变量REPLY</td></tr>
<tr><td>read –a arrayname</td><td>读入一组词，依次赋值给数组arrayname</td></tr>
<tr><td>read –s</td><td>有时候需要脚本用户进行输入，但不希望输入的数据显示在监视器上</td></tr>
<tr><td>read –t</td><td>指定read命令等待输入的秒数。当计时器计时数满时，read命令返回一个非零退出状态</td></tr>
<tr><td>read –p desp</td><td>允许在read命令行中直接指定一个提示desp</td></tr>
</tbody>
</table>

当然还有其他的，熟悉这些基本就够了。

### read 实例

例子1：

	#!/bin/sh
	
	echo -n "enter your name:"   # -n用于允许用户在字符串后面立即输入数据，而不是在下一行输入
	read name
	echo "Hello ${name}."

运行结果：

	enter your name:tom
	Hello tom.

例子2：

	#!/bin/sh
	
	read -p "please enter your age:" age  # age与前面必须有空格
	echo "your age is $age"

运行结果：

	please enter your age:15
	your age is 15

例子3：

	#!/bin/sh
	
	read -p "enter a number:"
	factorial=1
	for (( count=1; count<=$REPLY; count++ ))
	do
	   factorial=$[ $factorial * $count ]
	done
	echo "the factorial of $REPLY is $factorial"

运行结果：(1 x 2 x 3 x 4 x 5 = 120)

	enter a number:5
	the factorial of 5 is 120

例子4：

	#!/bin/sh
	
	if read -t 5 -p "please enter your name:" name
	then
	    echo "hello $name ,welcome to my script"
	else
	  echo "sorry ,tow slow!"
	fi

运行结果：

	[panda@master ~]$ ./test.sh 
	please enter your name:tom
	hello tom ,welcome to my script

	[panda@master ~]$ ./test.sh  # 等待了5s
	please enter your name:sorry ,tow slow!

例子5：

	#!/bin/sh
	
	read -s -p "enter your password:" pass
	echo  "is your password really $pass?"

运行结果：

	[panda@master ~]$ ./test.sh 
	enter your password:is your password really hyxabc?

例子6：

	#!/bin/sh
	
	read first middle last
	echo "Hello $first"
	echo "Hello $middle"
	echo "Hello $last"

运行结果：

	[panda@master ~]$ ./test.sh 
	aa						# 输入一个
	Hello aa
	Hello 
	Hello 
	[panda@master ~]$ ./test.sh 
	aa bb cc				# 输入三个
	Hello aa
	Hello bb
	Hello cc
	[panda@master ~]$ ./test.sh 
	aa bb cc dd ee ff		# 输入六个
	Hello aa
	Hello bb
	Hello cc dd ee ff

例子7：

	#!/bin/sh
	
	read -a friends
	echo "Say hi to ${friends[1]}."

运行结果：

	tom jack rose
	Say hi to jack.

例子8：

从文件读取内容。

	#!/bin/sh
	
	count=0
	cat ~/abc.txt | while read line
	do
	  echo "line $count: $line"
	  count=$[$count + 1]
	done

abc.txt的内容：

	sss
	sdfe
	lll
	sss

运行结果：

	line 0: sss
	line 1: sdfe
	line 2: lll
	line 3: sss

## 标准输出

上面的 `read` 是读取标准输入，所谓的标准就是通过终端输入进去的内容，下面说一下输出（`echo`, `pritf`）。

### echo 命令

echo是Shell的一个内部指令，用于在屏幕上打印出指定的字符串。命令格式：

	echo [-ne][字符串]

- `-n` 不要在最后自动换行（默认是自动换行的）
- `-e` 打开反斜杠转义，支持输入转义字符。若字符串中出现以下字符，则特别加以处理，而不会将它当成一般文字输出：

		\a 发出警告声；
		\b 删除前一个字符；
		\c 最后不加上换行符号；
		\f 换行但光标仍旧停留在原来的位置；
		\n 换行且光标移至行首；
		\r 光标移至行首，但不换行；
		\t 插入tab；
		\v 与\f相同；
		\\ 插入\字符；
		\nnn 插入nnn（八进制）所代表的ASCII字符；
- `-E` 取消反斜杠转义 (默认)
- `-help` 显示帮助
- `-version` 显示版本信息

还有很多高级的用法，这里仅介绍基本用法。

### printf 命令

printf 命令用于格式化输出， 是echo命令的增强版。

注意：printf 由 POSIX 标准所定义，移植性要比 echo 好。

如同 echo 命令，printf 命令也可以输出简单的字符串：

	printf "Hello, Shell\n"

运行结果：

	Hello, Shell

printf 不像 echo 那样会自动换行，必须显式添加换行符(\n)。

printf 命令的语法：

	printf  format-string  [arguments...]

format-string 为格式控制字符串，arguments 为参数列表。

看下面的例子：

	# format-string为双引号
	printf "%d %s\n" 1 "abc"
	1 abc
	# 单引号与双引号效果一样 
	printf '%d %s\n' 1 "abc" 
	1 abc
	# 没有引号也可以输出
	printf %s abcdef
	abcdef
	# 格式只指定了一个参数，但多出的参数仍然会按照该格式输出，format-string 被重用
	printf %s abc def
	abcdef
	printf "%s\n" abc def
	abc
	def
	printf "%s %s %s\n" a b c d e f g h i j
	a b c
	d e f
	g h i
	j
	# 如果没有 arguments，那么 %s 用NULL代替，%d 用 0 代替
	printf "%s and %d \n" 
	and 0
	# 如果以 %d 的格式来显示字符串，那么会有警告，提示无效的数字，此时默认置为 0
	printf "The first program always prints'%s,%d\n'" Hello Shell
	-bash: printf: Shell: invalid number
	The first program always prints 'Hello,0'

注意，根据POSIX标准，浮点格式%e、%E、%f、%g与%G是“不需要被支持”。这是因为awk支持浮点预算，且有它自己的printf语句。这样Shell程序中需要将浮点数值进行格式化的打印时，可使用小型的awk程序实现。然而，内建于bash、ksh93和zsh中的printf命令都支持浮点格式。

## 文件重定向

命令的输入输出不仅可以是显示器，还可以是文件。

### 输出重定向

命令输出重定向的语法为：

	command > file

这样，输出到显示器的内容就可以被重定向到文件。

	echo "hello world..." > abc.txt # 创建abc.txt文件，内容为hello world...如果存在abc.txt，abc.txt的内容将被替换。

	echo "hello world..." >> abc.txt # >> 表示追加到abc.txt中

### 输入重定向

和输出重定向一样，Unix 命令也可以从文件获取输入，语法为：

	command < file

这样，本来需要从键盘获取输入的命令会转移到文件读取内容。

统计 abc.txt 的行数

	wc -l < ~/abc.txt

### 重定向深入讲解

一般情况下，每个 Unix/Linux 命令运行时都会打开三个文件：

- 标准输入文件(stdin)：stdin的文件描述符为0，Unix程序默认从stdin读取数据。
- 标准输出文件(stdout)：stdout 的文件描述符为1，Unix程序默认向stdout输出数据。
- 标准错误文件(stderr)：stderr的文件描述符为2，Unix程序会向stderr流中写入错误信息。

默认情况下，`command > file` 将 stdout 重定向到 file，`command < file` 将stdin 重定向到 file。

如果希望 stderr 重定向到 file，可以这样写：

	command 2 > file
	command 2 >> file

2 表示标准错误文件(stderr)。

如果希望将 stdout 和 stderr 合并后重定向到 file，可以这样写：

	command > file 2>&1

或

	command >> file 2>&1

如果希望对 stdin 和 stdout 都重定向，可以这样写：

	command < file1 > file2

command 命令将 stdin 重定向到 file1，将 stdout 重定向到 file2.

### Here Document （嵌入文档）

	command << delimiter
	    document
	delimiter

它的作用是将两个 delimiter 之间的内容(document) 作为输入传递给 command。

注意：

- 结尾的delimiter 一定要顶格写，前面不能有任何字符，后面也不能有任何字符，包括空格和 tab 缩进。
- 开始的delimiter前后的空格会被忽略掉。

下面的例子，通过 `wc -l` 命令计算 document 的行数：

	wc -l << EOF
	    This is a simple lookup program
	    for good (and bad) restaurants
	    in Cape Town.
	EOF
	# 输出3
	3

### /dev/null 文件

如果希望执行某个命令，但又不希望在屏幕上显示输出结果，那么可以将输出重定向到 /dev/null：

	command > /dev/null

/dev/null 是一个特殊的文件，写入到它的内容都会被丢弃；如果尝试从该文件读取内容，那么什么也读不到。但是 /dev/null 文件非常有用，将命令的输出重定向到它，会起到”禁止输出“的效果。

如果希望屏蔽 stdout 和 stderr，可以这样写：

	command > /dev/null 2>&1

## 文件包含

一样，Shell 也可以包含外部脚本，将外部脚本的内容合并到当前脚本。

Shell 中包含脚本可以使用：

	. filename

或者

	source filename

两种方式的效果相同，简单起见，一般使用点号(.)，但是注意点号(.)和文件名中间有一空格。

例如，创建两个脚本，一个是被调用脚本 subscript.sh，内容如下：

	sayHi="Hi,jack..."

一个是主文件 test.sh，内容如下：

	#!/bin/bash
	. ./subscript.sh
	echo $sayHi

执行脚本：

	chmod 700 test.sh
	./test.sh
	Hi,jack...

**注意：被包含脚本不需要有执行权限。**



## 参考资料

- [Shell读取用户输入](http://blog.csdn.net/zilong00007/article/details/6681090)
- [linuxSHELL学习之获取用户输入 ](http://blog.itpub.net/23655288/viewspace-735056/)
- [Shell命令：echo介绍，echo如何输出带颜色的文本 ](http://blog.chinaunix.net/uid-27124799-id-3383327.html)
- [Shell输入输出重定向：Shell Here Document，/dev/null文件](http://c.biancheng.net/cpp/view/2738.html)
- [shell printf命令：格式化输出语句](http://c.biancheng.net/cpp/view/1499.html)
