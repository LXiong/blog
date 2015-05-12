# Linux 多线程下载工具 Axel

> 目录
> 
> - [相关网站]()
> - [Axel 安装]()
>     - [源码安装]()
>     - [Ubuntu Axel]()
>     - [Centos Axel]()
> - [Axel 的使用]()
>     - [命令格式]()
>     - [参数详解]()
>     - [Basic 认证]()
>     - [配置文件]()
> - [参考资料]()

Axel 通过打开多个 HTTP/FTP 连接来将一个文件进行分段下载，从而达到加速下载的目的。对于下载大文件，该工具将特别有用。

测试环境：

- Centos 6.5 64

## 相关网站

- [Axel 官网](http://axel.alioth.debian.org/)
- [Axel 资源列表](https://alioth.debian.org/frs/?group_id=100070) 资源下载还需要账号密码，让人不爽
- [Axel rpm 包](http://pkgs.repoforge.org/axel/) 支持rpm安装的系统可以到这免费下载
- [Axel WebSVN](http://svn.debian.org/wsvn/axel)
- [Axel ViewSVN](http://svn.debian.org/viewsvn/axel)

## Axel 的安装

此软件非系统自带，需手动安装，安装很简单。

### 源码安装

下载源码包，`axel-**.tar.gz`

```shell
tar -zvxf axel-**.tar.gz
cd axel**
./configure
sudo make
sudo make install
```

`**` 为未定的文件部分，根据你下载的内容更改。

### Ubuntu Axel

```shell
sudo apt-get install axel
```

### Centos Axel

```shell
sudo yum install axel
```
使用rpm包安装：

```shell
wget http://pkgs.repoforge.org/axel/axel-2.4-1.el6.rf.x86_64.rpm
sudo rpm -ivh axel-2.4-1.el6.rf.x86_64.rpm
```

## Axel 的使用

使用也很简单，先看帮助吧。

```shell
axel -h
```

![](../static/img/20150512164049.png)

### 命令格式

	axel [options] url1 [url2] [url...]

Axel 可以带多个参数，并且可以同时下载多个文件（下载链接使用空格隔开）。

参数列表在上图中都展示出来了。

### 参数详解

> `-s x` 或 `--max-speed=x`

限制下载速度(bytes per second)，x为任意正整数，单位为字节。

```shell
axel --max-speed=2048 http://example.com/a.rar
# 等价于
axel -s 2048 http://example.com/a.rar
```

> `-n x` 或 `--num-connections=x`

限制最大连接数（即开几个线程下载）。

```shell
axel -n 3 http://example.com/a.rar
```

> `-o f` 或 `--output=f`

对下载的文件进行重命名。

```shell
axel http://example.com/a.rar -o test.rar
# a.rar 下载完将被保存为 test.rar
axel http://example.com/a.rar -o /home/panda/
# 将 a.rar 保存到 /home/panda 目录下
```

> `-S [x]` 或 `--search[=x]`

搜索镜像并从x server服务器下载。

此例子摘自网络

```shell
axel -S5 ftp://ftp.kernel.org/pub/linux/kernel/v2.6/linux-2.6.9.tar.gz
# -S5 : 表示通过文件搜索引擎 filesearching.com 来寻找镜像文件，以便增快下载速度。
```

> `-H x` 或 `--header=x`

添加 HTTP 请求头。

```shell
axel -H "Accept-Encoding:gzip, deflate, sdch" http://example.com/a.rar
```

> `-U x` 或 `--user-agent=x`

设置用户浏览器信息。

```shell
axel -U "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.135 Safari/537.36" http://example.com/a.rar
```

> `-N` 或 `--no-proxy`

不使用代理。

```shell
axel -N http://example.com/a.rar
```

> `-q` 或 `--quiet`

静默下载，即偷偷的下载，你看不到下载进度等信息。

```shell
axel -q http://example.com/a.rar
```

> `-v` 或 `--verbose`

更多状态信息

> `-V` 或 `--version`

显示版本信息

> `-a` 或 `--alternate`

以不刷屏的方式显示进度。进度条就一条。

类似：

	[69%] ..... ...... ..... ...      [262.8KB/s]

而不是：

	[  0%] .....            [262.8KB/s]
	[  6%] ..... ..         [262.8KB/s]

这样的多条进度线。

### Basic 认证

Axel 可以支持简单的Basic认证实现加密数据的下载。

```shell
axel http://user:password@example.com/a.rar
```

### 配置文件

Axel 的默认配置文件是 `/etc/axelrc`，里面有一些可配置参数。

```shell
############################################################################
##                                                                        ##
##   Example configuration file for Axel - A light download accelerator   ##
##                                                                        ##
############################################################################

# reconnect_delay sets the number of seconds before trying again to build
# a new connection to the server.
#
# reconnect_delay = 20

# You can set a maximum speed (bytes per second) here, Axel will try to be
# at most 5% faster or 5% slower.
#
# max_speed = 0

# You can set the maximum number of connections Axel will try to set up
# here. There's a value precompiled in the program too, setting this too
# high requires recompilation. PLEASE respect FTP server operators and other
# users: Don't set this one too high!!! 4 is enough in most cases.
#
# num_connections = 4

# If no data comes from a connection for this number of seconds, abort (and
# resume) the connection.
#
# connection_timeout = 45

# Set proxies. no_proxy is a comma-separated list of domains which are
# local, axel won't use any proxy for them. You don't have to specify full
# hostnames there.
#
# Note: If the HTTP_PROXY environment variable is set correctly already,
# you don't have to set it again here. The setting should be in the same
# format as the HTTP_PROXY var, like 'http://host.domain.com:8080/',
# although a string like 'host.domain.com:8080' should work too..
#
# Sometimes HTTP proxies support FTP downloads too, so the proxy you
# configure here will be used for FTP downloads too.
#
# If you have to use a SOCKS server for some reason, the program should
# work perfectly through socksify without even knowing about that server.
# I haven't tried it myself, so I would love to hear from you about the
# results.
#
# http_proxy =
# no_proxy =

# Keep CGI arguments in the local filename?
#
# strip_cgi_parameters = 1

# When downloading a HTTP directory/index page, (like http://localhost/~me/)
# what local filename do we have to store it in?
#
# default_filename = default

# Save state every x seconds. Set this to 0 to disable state saving during
# download. State files will always be saved when the program terminates
# (unless the download is finished, of course) so this is only useful to
# protect yourself against sudden system crashes.
#
# save_state_interval = 10

# Buffer size: Maximum amount of bytes to read from a connection. One single
# buffer is used for all the connections (no separate per-connection buffer).
# A smaller buffer might improve the accuracy of the speed limiter, but a
# larger buffer is a better choice for fast connections.
#
# buffer_size = 5120

# By default some status messages about the download are printed. You can
# disable this by setting this one to zero.
#
# verbose = 1

# FTP searcher
#
# search_timeout - Maximum time (seconds) to wait for a speed test
# search_threads - Maximum number of speed tests at once
# search_amount  - Number of URL's to request from filesearching.com
# search_top     - Number of different URL's to use for download
#
# search_timeout = 10
# search_threads = 3
# search_amount = 15
# search_top = 3

# If you have multiple interfaces to the Internet, you can make Axel use all
# of them by listing them here (interface name or IP address). If one of the
# interfaces is faster than the other(s), you can list it more than once and
# it will be used more often.
#
# This option is blank by default, which means Axel uses the first match in
# the routing table.
#
# interfaces =

# If you don't like the wget-alike interface, you can set this to 1 to get
# a scp-alike interface.
#
# alternate_output = 1
```

此配置文件这里就不翻译了，英语水平不行。

## 参考资料

- [Axel - Ubuntu中文](http://wiki.ubuntu.org.cn/Axel)
- [Axel使用技巧 Wow!Ubuntu](http://wowubuntu.com/axel.html)
- [Linux下的下载工具 axel](http://www.cnblogs.com/SunWentao/archive/2008/07/10/1239924.html)


