# Aira2 下载工具

Aria2是一个命令行下运行、多协议、多来源下载工具（HTTP/HTTPS、FTP、BitTorrent、Metalink），内建 XML-RPC 用户界面。

轻量，平均4-9MB内存使用量，BitTorrent下载速度2.8MiB/s时CPU占用约6%。

全面的BitTorrent特性支持，包括 DHT, PEX, Encryption, Magnet URI, Web-Seeding，选择下载，本地资源探测。 Mtalink 支持。

环境：

- Centos 6.5
- Aria2 1.16.4

## 相关网站

- [资源下载站点](http://sourceforge.net/projects/aria2/files/)
- [Aria2 Github](https://github.com/tatsuhiro-t/aria2)
- [Aria2 rpm 下载点](http://pkgs.repoforge.org/aria2/)

## 安装 Aria2

Aria2 支持 Mac , Linux , Android 系统。

### 源码编译安装

到资源点，下载相关的源码包，执行编译安装。

```shell
tar -zvxf aria2-**.tar.gz
cd aria2-**
./configure
sudo make
sudo make install
```

我在 Centos 6.5 上测试最新版 `1.8.10` 未通过，因为 6.5 中的 C++ 库比较旧了，而新版本的 Aria2 使用的 C++ 版本库较新，所以导致编译失败，在 Ubuntu 中编译通过，在 Centos 6.5 上编译 1.6 左右版本的测试通过。

### Ubuntu 安装 Aria2

除了编译安装外，Ubuntu 用户可以：

```shell
sudo apt-get install aria2
```

### Centos 安装 Aria2

Centos用户可以选择直接下载rpm包进行安装（注意系统的位数）：

> CentOS 6.x 32 位下安装

```shell
wget -c http://pkgs.repoforge.org/aria2/aria2-1.16.4-1.el6.rf.i686.rpm
rpm -ivh aria2-1.16.4-1.el6.rf.i686.rpm
```

> CentOS 6.x 64 位下安装

```shell
wget -c http://pkgs.repoforge.org/aria2/aria2-1.16.4-1.el6.rf.x86_64.rpm
rpm -ivh aria2-1.16.4-1.el6.rf.x86_64.rpm
```

在安装过程有可能会出现缺少 **libnettle.so.4** 的错误提示。

因此需要先到 [http://pkgs.repoforge.org/nettle/](http://pkgs.repoforge.org/nettle/) 去下载安装 nettle 即可。

> CentOS 6.x 32 位下安装 nettle

```shell
wget -c http://pkgs.repoforge.org/nettle/nettle-2.2-1.el6.rf.i686.rpm
wget -c http://pkgs.repoforge.org/nettle/nettle-devel-2.2-1.el6.rf.i686.rpm
rpm -ivh nettle-2.2-1.el6.rf.i686.rpm
rpm -ivh nettle-devel-2.2-1.el6.rf.i686.rpm
```

> CentOS 6.x 64 位下安装 nettle

```shell
wget -c http://pkgs.repoforge.org/nettle/nettle-2.2-1.el6.rf.x86_64.rpm
wget -c http://pkgs.repoforge.org/nettle/nettle-devel-2.2-1.el6.rf.x86_64.rpm
rpm -ivh nettle-2.2-1.el6.rf.x86_64.rpm
rpm -ivh nettle-devel-2.2-1.el6.rf.x86_64.rpm
```

通过 rpm 安装的一般都不会是最新版的。官网目前的最新版本已经到 1.18.10 ， Centos 6.5 系统只能编译安装 1.6 版本左右的 Aria2.

安装完成后，还可以安装一个 Web UI 工具，可视化管理下载过程。

围观网址：

[https://github.com/ziahamza/webui-aria2](https://github.com/ziahamza/webui-aria2)

我没有安装过，只是介绍有这么个东东。

## Aria2c 的使用

Aria2 安装完成后，我们要使用的命令其实是 `aria2c`，输入：

```shell
aria2c -h
```

查看此命令的帮助信息，介绍的挺详细的。

由于命令较多，你可以在 `aria2c -h` 后面加上命令标记，以查看特定的命令：

```shell
aria2c -h#basic
```

查看所有可选的标记：

```shell
aria2c -h#help
```
所有可选的标记为：

- basic
- advanced
- http
- https
- ftp
- metalink
- bittorrent
- cookie
- hook
- file
- rpc
- checksum
- experimental
- deprecated
- help
- all

### 命令格式

	Usage: aria2c [OPTIONS] [URI | MAGNET | TORRENT_FILE | METALINK_FILE]...

### 使用示例

由于功能太过强大，这里仅做入门使用介绍。

> 最简单的下载

```shell
aria2c http://example.com/a.rar
```

> 从多个源下载

```shell
aria2c http://example.com/a.rar ftp://a/f.iso
```

> BitTorrent下载

```shell
aria2c http://example.com/a.torrent
```

> BitTorrent Magnet URI

```shell
aria2c 'magnet:?xt=urn:btih:248D0A1CD08284299DE78D5C1ED359BB46717D8C'
```

> Metalink

```shell
aria2c http://example.com/a.metalink
```

> 文件中的URI

```shell
aria2c -i uri.txt
```

> 显示种子中包含哪些文件

```shell
aria2c -S bit.torrent
```

> 选择仅下载编号为 1,4,7 的文件

```shell
aria2c --select-file 1,4,7 bit.torrent
```

> 下载重命名

```shell
aria2c -o b.rar http://example.com/a.rar
```

> 下载https的文件

推荐加：`–check-certificate=false` 不检查ssl证书。

```shell
aria2c -s 10 –check-certificate=false https://www.google.com/a.zip
```

> 开多个连接去下载

```shell
aria2c -x 2 http://host/image.iso
```

对比

```shell
aria2c -s 2 http://host/image.iso
```

`-x` 与 `-s` 有一定关联，`-x` 指 `--max-connection-per-server` 每个Host最大连接数，默认为1，而 `-s` 指开多少线程，分多少块去下载这个文件，对相同主机的连接数被 `-x` 选项所限制。

```shell
aria2c -s 3 -x 2 http://host/a.rar
```

此例最大连接只能是2。

对于有多个URI的

```shell
aria2c -s 2 http://host/image.iso http://mirror1/image.iso http://mirror2/image.iso
```

上例有3个URI，连接数为2，只会用到URI中的2个，从前往后排，没用到的做备用，前面的不能用了，调用备用URI。

## aria2c help

贴出帮助文档方便查阅（未翻译）

	Options:
	 -v, --version 打印版本号并退出。
	
	                              标记： #basic
	
	 -h, --help[=TAG|KEYWORD]     Print usage and exit.
	                              The help messages are classified with tags. A tag
	                              starts with "#". For example, type "--help=#http"
	                              to get the usage for the options tagged with
	                              "#http". If non-tag word is given, print the usage
	                              for the options whose name includes that word.
	
	                              可能的取值： #basic, #advanced, #http, #https, #ftp, #metalink, #bittorrent, #cookie, #hook, #file, #rpc, #checksum, #experimental, #deprecated, #help, #all
	                              默认： #basic
	                              标记： #basic, #help
	
	 -l, --log=LOG 日志文件名称。如果指定'-'，
	                              日志将被写到标准输出（通常是显示器）。
	
	                              可能的取值： /path/to/file, -
	                              标记： #basic
	
	 -d, --dir=DIR 用于存储已下载文件的目录。
	
	                              可能的取值： /path/to/directory
	                              默认： /home/panda
	                              标记： #basic, #file
	
	 -o, --out=FILE 下载文件的文件名。当使用 -Z
	                              选项时，此选项被忽略。
	
	                              可能的取值： /path/to/file
	                              标记： #basic, #http, #ftp, #file
	
	 -s, --split=N 使用N个连接下载文件。如果提供了超过N个URL，使用前N个URL而剩下的URL作为备用。
	                    如果提供了少于N个URL，这些URL将会被重复使用来使这N个连接能够同时建立。
	                    对相同主机的连接数被--max-connection-per-server选项所限制。
	                    另见--min-split-size选项。
	
	                              可能的取值： 1-*
	                              默认： 5
	                              标记： #basic, #http, #ftp
	
	 --file-allocation=METHOD     Specify file allocation method.
	                              'none' doesn't pre-allocate file space. 'prealloc'
	                              pre-allocates file space before download begins.
	                              This may take some time depending on the size of
	                              the file.
	                              If you are using newer file systems such as ext4
	                              (with extents support), btrfs, xfs or NTFS
	                              (MinGW build only), 'falloc' is your best
	                              choice. It allocates large(few GiB) files
	                              almost instantly. Don't use 'falloc' with legacy
	                              file systems such as ext3 and FAT32 because it
	                              takes almost same time as 'prealloc' and it
	                              blocks aria2 entirely until allocation finishes.
	                              'falloc' may not be available if your system
	                              doesn't have posix_fallocate() function.
	                              'trunc' uses ftruncate() system call or
	                              platform-specific counterpart to truncate a file
	                              to a specified length.
	
	                              可能的取值： none, prealloc, trunc, falloc
	                              默认： prealloc
	                              标记： #basic, #file
	
	 -V, --check-integrity[=true|false] Check file integrity by validating piece
	                              hashes or a hash of entire file. This option has
	                              effect only in BitTorrent, Metalink downloads
	                              with checksums or HTTP(S)/FTP downloads with
	                              --checksum option. If piece hashes are provided,
	                              this option can detect damaged portions of a file
	                              and re-download them. If a hash of entire file is
	                              provided, hash check is only done when file has
	                              been already download. This is determined by file
	                              length. If hash check fails, file is
	                              re-downloaded from scratch. If both piece hashes
	                              and a hash of entire file are provided, only
	                              piece hashes are used.
	
	                              可能的取值： true, false
	                              默认： false
	                              标记： #basic, #metalink, #bittorrent, #file, #checksum
	
	 -c, --continue[=true|false] 继续下载一个仅部分完成的文件。
	                              使用这个选项来继续下载一个由浏览器或其他程序从开头单线程下载的文件。
	                              目前这个选项仅能用于http(s)/ftp下载。
	
	                              可能的取值： true, false
	                              默认： false
	                              标记： #basic, #http, #ftp
	
	 -i, --input-file=FILE 下载FILE中列出的地址。
	                              可以一次使用多个地址，在同一行里使用制表符分隔多个地址。使用"-"时从标准输入读取。
	                              另外，在每一行地址后可以指定选项。包含选项的行必须以至少一个空格开始，并且每行一个选项。
	                              在man手册中查看INPUT FILE章节。另见--deferred-input选项。
	
	                              可能的取值： /path/to/file, -
	                              标记： #basic
	
	 -j, --max-concurrent-downloads=N 设置对于独立的（http/ftp）地址、torrent和metalink的最大并行下载数量。
	                              另见--split选项。
	
	                              可能的取值： 1-*
	                              默认： 5
	                              标记： #basic
	
	 -Z, --force-sequential[=true|false] 继续从命令行获取链接，
	                              并以单独的会话下载每个链接，
	                              如同常见的命令行下载工具。
	
	                              可能的取值： true, false
	                              默认： false
	                              标记： #basic
	
	 -x, --max-connection-per-server=NUM The maximum number of connections to one
	                              server for each download.
	
	                              可能的取值： 1-16
	                              默认： 1
	                              标记： #basic, #http, #ftp
	
	 -k, --min-split-size=SIZE    aria2 does not split less than 2*SIZE byte range.
	                              For example, let's consider downloading 20MiB
	                              file. If SIZE is 10M, aria2 can split file into 2
	                              range [0-10MiB) and [10MiB-20MiB) and download it
	                              using 2 sources(if --split >= 2, of course).
	                              If SIZE is 15M, since 2*15M > 20MiB, aria2 does
	                              not split file and download it using 1 source.
	                              You can append K or M(1K = 1024, 1M = 1024K).
	
	                              可能的取值： 1048576-1073741824
	                              默认： 20M
	                              标记： #basic, #http, #ftp
	
	 --ftp-user=USER 设置FTP用户。此设置对所有链接有效。
	
	                              标记： #basic, #ftp
	
	 --ftp-passwd=PASSWD 设置FTP密码。此设置对所有链接有效。
	
	                              标记： #basic, #ftp
	
	 --http-user=USER 设置HTTP用户。此设置对所有链接有效。
	
	                              标记： #basic, #http
	
	 --http-passwd=PASSWD 设置HTTP密码。此设置对所有链接有效。
	
	                              标记： #basic, #http
	
	 --load-cookies=FILE 从FILE中载入Cookies，这些FILE通常使用Firefox3格式
	                              和Mozilla/Firefox(1.x/2.x)/Netscape格式。
	
	                              可能的取值： /path/to/file
	                              标记： #basic, #http, #cookie
	
	 -S, --show-files[=true|false] Print file listing of .torrent, .meta4 and
	                              .metalink file and exit. More detailed
	                              information will be listed in case of torrent
	                              file.
	
	                              可能的取值： true, false
	                              默认： false
	                              标记： #basic, #metalink, #bittorrent
	
	 --max-overall-upload-limit=SPEED Set max overall upload speed in bytes/sec.
	                              0 means unrestricted.
	                              You can append K or M(1K = 1024, 1M = 1024K).
	                              To limit the upload speed per torrent, use
	                              --max-upload-limit option.
	
	                              可能的取值： 0-*
	                              默认： 0
	                              标记： #basic, #bittorrent
	
	 -u, --max-upload-limit=SPEED 设置每个torrent上传速度最大值（b/s）。
	                              0意味着不限制。
	                              您可以附加K或M（1K=1024，1M=1024K）。
	                              要限制总体上传速度，使用
	                              --max-overall-upload-limit option.选项。
	
	                              可能的取值： 0-*
	                              默认： 0
	                              标记： #basic, #bittorrent
	
	 -T, --torrent-file=TORRENT_FILE 指定.torrent文件的路径。
	
	                              可能的取值： /path/to/file
	                              标记： #basic, #bittorrent
	
	 --listen-port=PORT... 为BT下载设置TCP端口。
	                              使用','可以指定多个端口，例如：
	                              "6881,6885"。您也可以使用'-'
	                              指定一个范围："6881-6999"。','
	                              和'-'可以一起使用。
	
	                              可能的取值： 1024-65535
	                              默认： 6881-6999
	                              标记： #basic, #bittorrent
	
	 --enable-dht[=true|false]    Enable IPv4 DHT functionality. It also enables
	                              UDP tracker support. If a private flag is set
	                              in a torrent, aria2 doesn't use DHT for that
	                              download even if ``true`` is given.
	
	                              可能的取值： true, false
	                              默认： true
	                              标记： #basic, #bittorrent
	
	 --dht-listen-port=PORT...    Set UDP listening port used by DHT(IPv4, IPv6)
	                              and UDP tracker. Multiple ports can be specified
	                              by using ',', for example: "6881,6885". You can
	                              also use '-' to specify a range: "6881-6999".
	                              ',' and '-' can be used together.
	
	                              可能的取值： 1024-65535
	                              默认： 6881-6999
	                              标记： #basic, #bittorrent
	
	 --enable-dht6[=true|false]   Enable IPv6 DHT functionality.
	                              Use --dht-listen-port option to specify port
	                              number to listen on. See also --dht-listen-addr6
	                              option.
	
	                              可能的取值： true, false
	                              默认： false
	                              标记： #basic, #bittorrent
	
	 --dht-listen-addr6=ADDR      Specify address to bind socket for IPv6 DHT. 
	                              It should be a global unicast IPv6 address of the
	                              host.
	
	                              标记： #basic, #bittorrent
	
	 -M, --metalink-file=METALINK_FILE The file path to the .meta4 and .metalink
	                              file. Reads input from stdin when '-' is
	                              specified.
	
	                              可能的取值： /path/to/file, -
	                              标记： #basic, #metalink
	
	URI, MAGNET, TORRENT_FILE, METALINK_FILE:
	 You can specify multiple HTTP(S)/FTP URIs. Unless you specify -Z option, all
	 URIs must point to the same file or downloading will fail.
	 You can also specify arbitrary number of BitTorrent Magnet URIs, torrent/
	 metalink files stored in a local drive. Please note that they are always
	 treated as a separate download.
	
	 You can specify both torrent file with -T option and URIs. By doing this,
	 download a file from both torrent swarm and HTTP/FTP server at the same time,
	 while the data from HTTP/FTP are uploaded to the torrent swarm. For single file
	 torrents, URI can be a complete URI pointing to the resource or if URI ends
	 with '/', 'name' in torrent file is added. For multi-file torrents, 'name' and
	 'path' in torrent are added to form a URI for each file.
	
	 Make sure that URI is quoted with single(') or double(") quotation if it
	 contains "&" or any characters that have special meaning in shell.
	
	About the number of connections
	 Since 1.10.0 release, aria2 uses 1 connection per host by default and has 20MiB
	 segment size restriction. So whatever value you specify using -s option, it
	 uses 1 connection per host. To make it behave like 1.9.x, use
	 --max-connection-per-server=4 --min-split-size=1M.
	
	Refer to man page for more information.


## 参考资料

- [Aria2 - Ubuntu中文](http://wiki.ubuntu.org.cn/Aria2)
- [Ubuntu下强大的下载工具Aria2c](http://cduym.iteye.com/blog/1716377)
- [CentOS下安装aria2教程](http://teddysun.com/381.html)
- [曾经的 Usage Sample（好心人收藏的）](http://ears.iteye.com/blog/1593816)
- [aria2c(1) 使用说明](https://github.com/tatsuhiro-t/aria2/blob/master/doc/manual-src/en/aria2c.rst)