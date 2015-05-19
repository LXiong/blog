# 几种常见的 Shell

Shell是一种脚本语言，那么，就必须有解释器来执行这些脚本。

Unix/Linux上常见的Shell脚本解释器有bash、sh、csh、ksh等，习惯上把它们称作一种Shell。我们常说有多少种Shell，其实说的是Shell脚本解释器。

## bash

bash是Linux标准默认的shell，本教程也基于bash讲解。bash由Brian Fox和Chet Ramey共同完成，是BourneAgain Shell的缩写，内部命令一共有40个。

Linux使用它作为默认的shell是因为它有诸如以下的特色：

- 可以使用类似DOS下面的doskey的功能，用方向键查阅和快速输入并修改命令。
- 自动通过查找匹配的方式给出以某字符串开头的命令。
- 包含了自身的帮助功能，你只要在提示符下面键入help就可以得到相关的帮助。

## sh

sh 由Steve Bourne开发，是Bourne Shell的缩写，sh 是Unix 标准默认的shell。

## ash

ash shell 是由Kenneth Almquist编写的，Linux中占用系统资源最少的一个小shell，它只包含24个内部命令，因而使用起来很不方便。

## csh

csh 是Linux比较大的内核，它由以William Joy为代表的共计47位作者编成，共有52个内部命令。该shell其实是指向/bin/tcsh这样的一个shell，也就是说，csh其实就是tcsh。

## ksh

ksh 是Korn shell的缩写，由Eric Gisin编写，共有42条内部命令。该shell最大的优点是几乎和商业发行版的ksh完全兼容，这样就可以在不用花钱购买商业版本的情况下尝试商业版本的性能了。

**注意：**bash是 Bourne Again Shell 的缩写，是linux标准的默认shell ，它基于Bourne shell，吸收了C shell和Korn shell的一些特性。bash完全兼容sh，也就是说，用sh写的脚本可以不加修改的在bash中执行。