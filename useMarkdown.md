# Markdown 语法

Markdown的语法简洁明了、学习容易，而且功能强大，用它写博客既快速又美观，值得一用。

## 概述

### 宗旨

Markdown 的目标是实现「易读易写」。  
可读性，无论如何，都是最重要的。一份使用 Markdown 格式撰写的文件应该可以直接以纯文本发布，并且看起来不会像是由许多标签或是格式指令所构成。

### 兼容 HTML

Markdown 语法的目标是：成为一种适用于网络的书写语言。  
Markdown 不是想要取代 HTML，甚至也没有要和它相近，它的语法种类很少，只对应 HTML 标记的一小部分。  
不在 Markdown 涵盖范围之内的标签，都可以直接在文档里面用 HTML 撰写。不需要额外标注这是 HTML 或是 Markdown，只要直接加标签就可以了。  
要制约的只有一些 HTML 区块元素，比如 `<div>`、`<table>`、`<pre>`、`<p>` 等标签，必须在前后加上空行与其它内容区隔开，还要求它们的开始标签与结尾标签不能用制表符或空格来缩进。Markdown 的生成器有足够智能，不会在 HTML 区块标签外加上不必要的 `<p>` 标签。

例：在 Markdown 文件里加上一段 HTML 表格：

    <table>
        <tr>
            <td>Foo</td><td>Foo2</td><td>Foo3</td>
        </tr>
    </table>

效果如下：

<table>
    <tr>
        <td>Foo</td><td>Foo2</td><td>Foo3</td>
    </tr>
</table>

请注意，在 HTML 区块标签间的 Markdown 格式语法将不会被处理。比如，你在 HTML 区块内使用 Markdown 样式的`*强调*`会没有效果。  
HTML 的区段（行内）标签如 `<span>`、`<cite>`、`<del>` 可以在 Markdown 的段落、列表或是标题里随意使用。依照个人习惯，甚至可以不用 Markdown 格式，而直接采用 HTML 标签来格式化。举例说明：如果比较喜欢 HTML 的 `<a>` 或 `<img>` 标签，可以直接使用这些标签，而不用 Markdown 提供的链接或是图像标签语法。

和处在 HTML 区块标签间不同，Markdown 语法在 HTML 区段（行内）标签间是有效的。

### 特殊字符自动转换

在 HTML 文件中，有一些字符有时可能需要特殊处理，如 `<` 和 `&` 。 `<` 符号用于起始标签，`&` 符号则用于标记 HTML 实体，如果你只是想要显示这些字符的原型，使用实体的形式，像是 `&lt;` 和 `&amp;`。

例：

	AT&T
	AT&amp;T

效果：

AT&T  
AT&amp;T

例：

	< &lt; &gt; >

效果：

< &lt; &gt; >

`>` 是不能直接作为一行的开头的，它有特殊用途。

`&` 字符出现在网址中的情况，这里列出下面的几种情况。

例：

	http://images.google.com/images?num=30&q=larry+bird
	http://images.google.com/images?num=30&amp;q=larry+bird
	[test1](http://images.google.com/images?num=30&q=larry+bird)
	[test2](http://images.google.com/images?num=30&amp;q=larry+bird)
	<a href="http://images.google.com/images?num=30&q=larry+bird">test3</a>
	<a href="http://images.google.com/images?num=30&amp;q=larry+bird">test4</a>

效果：

http://images.google.com/images?num=30&q=larry+bird  
http://images.google.com/images?num=30&amp;q=larry+bird  
[test1](http://images.google.com/images?num=30&q=larry+bird)  
[test2](http://images.google.com/images?num=30&amp;q=larry+bird)  
<a href="http://images.google.com/images?num=30&q=larry+bird">test3</a>  
<a href="http://images.google.com/images?num=30&amp;q=larry+bird">test4</a>

Markdown 让你可以自然地书写字符，需要转换的由它来处理好了。如果你使用的 `&` 字符是 HTML 字符实体的一部分，它会保留原状，否则它会被转换成 `&amp`;。

所以你如果要在文档中插入一个版权符号 `©`，你可以这样写：

	&copy;

Markdown 会保留它不动。而若你写：

	AT&T

Markdown 就会将它转为：

	AT&amp;T

类似的状况也会发生在 `<` 符号上，因为 Markdown 允许 [兼容 HTML] ，如果你是把 `<` 符号作为 HTML 标签的定界符使用，那 Markdown 也不会对它做任何转换，但是如果你写：

    4 < 5

Markdown 将会把它转换为：

    4 &lt; 5

不过需要注意的是，code 范围内，不论是行内还是区块， `<` 和 `&` 两个符号都一定会被转换成 HTML 实体，这项特性让你可以很容易地用 Markdown 写 HTML code （和 HTML 相对而言， HTML 语法中，你要把所有的 `<` 和 `&` 都转换为 HTML 实体，才能在 HTML 文件里面写出 HTML code。）

## 区块元素

### 段落和换行

一个 Markdown 段落是由一个或多个连续的文本行组成，它的前后要有一个以上的空行（空行的定义是显示上看起来像是空的，便会被视为空行。比方说，若某一行只包含空格和制表符，则该行也会被视为空行）。普通段落不该用空格或制表符来缩进。  
「由一个或多个连续的文本行组成」这句话其实暗示了 Markdown 允许段落内的强迫换行（插入换行符）。  
如果你*确实*想要依赖 Markdown 来插入 `<br />` 标签的话，在插入处先按入两个以上的空格然后回车。

例：换行和插入空行

	我是一行[两个以上空格，回车]
	我是另一行

效果：

我是一行  
我是另一行

	我是一行[回车]
	[回车]
	我是另一行

效果：

我是一行

我是另一行

### 标题

Markdown 支持两种标题的语法。

利用多个 `=` （最高阶标题）做底线和多个 `-` （第二阶标题）做底线。

例：

	This is an H1
	=============
	This ia an H2
	-------------

效果：
This is an H1
=============
This ia an H2
-------------

在行首插入 1 到 6 个 `#` ，对应到标题 1 到 6 阶

例：

	# 这是H1
	## 这是H2
	### 这是H3
	#### 这是H4
	##### 这是H5
	###### 这是H6

效果：

# 这是H1
## 这是H2
### 这是H3
#### 这是H4
##### 这是H5
###### 这是H6

### 区块引用

### 列表

### 代码区块

### 分割线

## 区段（行间）元素

### 链接

### 强调

### 代码

### 图片

## 其他

### 反斜杠

### 自动链接

## 参考资料