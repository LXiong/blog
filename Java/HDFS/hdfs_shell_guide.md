# HDFS Shell 命令手册

HDFS文件系统提供了一套基于命令行的命令操作，非常方便。

## 概述

调用文件系统(FS)Shell命令使用

	hadoop fs <args>
	# 或
	hdfs dfs <args>

所有的的FS Shell命令使用URI路径作为参数。URI格式是`scheme://authority/path`。对HDFS文件系统，scheme是 `hdfs`，对本地文件系统，scheme是 `file`。其中scheme和authority参数都是可选的，如果未加指定，就会使用配置中指定的默认scheme。

一个HDFS文件或目录比如 `/parent/child` 可以表示成 `hdfs://namenode:namenodeport/parent/child` ，或者更简单的 `/parent/child`（假设你配置文件中的默认值是 `namenode:namenodeport` ）。

## 命令使用

HDFS 中提供了对文件操作的各种命令。

### appendToFile

用法: 

	hdfs dfs -appendToFile <localsrc> ... <dst>

将一个或多个文件添加到FS中. 也可以通过输入将数据添加到FS中.

示例:

	hdfs dfs -appendToFile localfile /user/hadoop/hadoopfile
	hdfs dfs -appendToFile localfile1 localfile2 /user/hadoop/hadoopfile
	hdfs dfs -appendToFile localfile hdfs://nn.示例.com/hadoop/hadoopfile
	# Reads the input from stdin.
	hdfs dfs -appendToFile - hdfs://nn.示例.com/hadoop/hadoopfile 

返回值:

成功返回 0 失败返回 1.

### cat

用法: 

	hdfs dfs -cat URI [URI ...]

读取文件内容并显示。

示例:

	hdfs dfs -cat hdfs://nn1.示例.com/file1 hdfs://nn2.示例.com/file2
	hdfs dfs -cat file:///file3 /user/hadoop/file4

返回值:

成功返回 0 失败返回 -1.

### chgrp

用法: 

	hdfs dfs -chgrp [-R] GROUP URI [URI ...]

改变文件所属组。

The user must be the owner of files, or else a super-user.

参数：

- `-R` ： 将使改变在目录结构下递归进行。

### chmod

用法: 

	hdfs dfs -chmod [-R] <MODE[,MODE]... | OCTALMODE> URI [URI ...]

改变文件的权限。

The user must be the owner of the file, or else a super-user. 

参数

- `-R` ： 将使改变在目录结构下递归进行。

### chown

用法: 

	hdfs dfs -chown [-R] [OWNER][:[GROUP]] URI [URI ]

改版文件的所有者。 

The user must be a super-user. 

参数:

- `-R` ： 将使改变在目录结构下递归进行。

### copyFromLocal

用法: 

	hdfs dfs -copyFromLocal <localsrc> URI

和`put`类似，除了限定源路径是一个本地文件外。

参数:

- `-f` ： 将会覆盖已经存在的目标路径。

### copyToLocal

用法: 

	hdfs dfs -copyToLocal [-ignorecrc] [-crc] URI <localdst>

和`get`类似，除了限定目标路径是一个本地文件外。

### count

用法: 

	hdfs dfs -count [-q] <paths>

Count the number of directories, files and bytes under the paths that match the specified file pattern. 

The output columns with `-count` are: `DIR_COUNT`, `FILE_COUNT`, `CONTENT_SIZE`, `FILE_NAME`

The output columns with `-count -q` are: `QUOTA`, `REMAINING_QUATA`, `SPACE_QUOTA`, `REMAINING_SPACE_QUOTA`, `DIR_COUNT`, `FILE_COUNT`, `CONTENT_SIZE`, `FILE_NAME`

示例:

	hdfs dfs -count hdfs://nn1.示例.com/file1 hdfs://nn2.示例.com/file2
	hdfs dfs -count -q hdfs://nn1.示例.com/file1

返回值:

成功返回 0 失败返回 -1.

### cp

用法: 

	hdfs dfs -cp [-f] URI [URI ...] <dest>

复制一个或多个文件到指定路径，如果是多个文件，目标路径必须是目录。

参数:

- `-f` ： 将会覆盖已经存在的目标路径。

示例:

	hdfs dfs -cp /user/hadoop/file1 /user/hadoop/file2
	hdfs dfs -cp /user/hadoop/file1 /user/hadoop/file2 /user/hadoop/dir

返回值:

成功返回 0 失败返回 -1.

### du

用法: 

	hdfs dfs -du [-s] [-h] URI [URI ...]

Displays sizes of files and directories contained in the given directory or the length of a file in case its just a file.

显示目录中所有文件的大小，或者当只指定一个文件时，显示此文件的大小。

参数:

- `-s` 参数 will result in an aggregate summary of file lengths being displayed, rather than the individual files.
- `-h` 参数 will format file sizes in a "human-readable" fashion (e.g 64.0m instead of 67108864)

示例:

	hdfs dfs -du /user/hadoop/dir1 /user/hadoop/file1 hdfs://nn.示例.com/user/hadoop/dir1

返回值: 

成功返回 0 失败返回 -1.

### dus

用法: 

	hdfs dfs -dus <args>

Displays a summary of file lengths. This is an alternate form of `hdfs dfs -du -s`.

### expunge

用法: 

	hdfs dfs -expunge

Empty the Trash. 清空回收站。

### get

用法: 

	hdfs dfs -get [-ignorecrc] [-crc] <src> <localdst>

Copy files to the local file system. Files that fail the CRC check may be copied with the `-ignorecrc` 参数. Files and CRCs may be copied using the `-crc` 参数.

复制文件到本地文件系统。可用`-ignorecrc`选项复制CRC校验失败的文件。使用`-crc`选项复制文件以及CRC信息。

示例:

	hdfs dfs -get /user/hadoop/file localfile
	hdfs dfs -get hdfs://nn.示例.com/user/hadoop/file localfile

返回值:

成功返回 0 失败返回 -1.

### getfacl

用法: 

	hdfs dfs -getfacl [-R] <path>

Displays the Access Control Lists (ACLs) of files and directories. If a directory has a default ACL, then getfacl also displays the default ACL.

参数:

- `-R` : List the ACLs of all files and directories recursively.
- `path` : File or directory to list.

示例:

	hdfs dfs -getfacl /file
	hdfs dfs -getfacl -R /dir

返回值:

成功返回 0 失败返回非 0.

### getfattr

用法: 

	hdfs dfs -getfattr [-R] -n name | -d [-e en] <path>

Displays the extended attribute names and values (if any) for a file or directory.

参数:

- `-R` : Recursively list the attributes for all files and directories.
- `-n name` : Dump the named extended attribute value.
- `-d` : Dump all extended attribute values associated with pathname.
- `-e encoding` : Encode values after retrieving them. Valid encodings are "text", "hex", and "base64". Values encoded as text strings are enclosed in double quotes ("), and values encoded as hexadecimal and base64 are prefixed with 0x and 0s, respectively.
- `path` : The file or directory.

示例:

	hdfs dfs -getfattr -d /file
	hdfs dfs -getfattr -R -n user.myAttr /dir

返回值:

成功返回 0 失败返回非 0.

### getmerge

用法：

	hadoop fs -getmerge <src> <localdst> [addnl]

接受一个源目录和一个目标文件作为输入，并且将源目录中所有的文件连接成本地目标文件。`addnl`是可选的，用于指定在每个文件结尾添加一个换行符。

### ls

用法: 

	hdfs dfs -ls <args>

如果是文件，则按照如下格式返回文件信息：

	permissions number_of_replicas userid groupid filesize modification_date modification_time filename

如果是目录，则返回它直接子文件的一个列表，就像在Unix中一样。目录返回列表的信息如下：

	permissions userid groupid modification_date modification_time dirname

示例:

	hdfs dfs -ls /user/hadoop/file1

返回值:

成功返回 0 失败返回 -1.

### lsr

用法: 

	hdfs dfs -lsr <args>

ls命令的递归版本。

**Note:** This command is **deprecated**. Instead use `hdfs dfs -ls -R`

### mkdir

用法: 

	hdfs dfs -mkdir [-p] <paths>

创建目录。

参数:

- `-p` ： 类似 Unix `mkdir -p`, 创建父目录。

示例:

	hdfs dfs -mkdir /user/hadoop/dir1 /user/hadoop/dir2
	hdfs dfs -mkdir hdfs://nn1.示例.com/user/hadoop/dir hdfs://nn2.示例.com/user/hadoop/dir

返回值:

成功返回 0 失败返回 -1.

### moveFromLocal

用法: 

	hdfs dfs -moveFromLocal <localsrc> <dst>

Similar to `put` command, except that the source localsrc is **deleted** after it's copied.

### moveToLocal

用法: 

	hdfs dfs -moveToLocal [-crc] <src> <dst>

Displays a "Not implemented yet" message.

### mv

用法: 

	hdfs dfs -mv URI [URI ...] <dest>

将文件从源路径移动到目标路径。这个命令允许有多个源路径，此时目标路径必须是一个目录。不允许在不同的文件系统间移动文件。 

示例:

	hdfs dfs -mv /user/hadoop/file1 /user/hadoop/file2
	hdfs dfs -mv hdfs://nn.示例.com/file1 hdfs://nn.示例.com/file2 hdfs://nn.示例.com/file3 hdfs://nn.示例.com/dir1

返回值:

成功返回 0 失败返回 -1.

### put

用法: 

	hdfs dfs -put <localsrc> ... <dst>

从本地文件系统中复制单个或多个源路径到目标文件系统。也支持从标准输入中读取输入写入目标文件系统。

示例:

	hdfs dfs -put localfile /user/hadoop/hadoopfile
	hdfs dfs -put localfile1 localfile2 /user/hadoop/hadoopdir
	hdfs dfs -put localfile hdfs://nn.示例.com/hadoop/hadoopfile
	# Reads the input from stdin.
	hdfs dfs -put - hdfs://nn.示例.com/hadoop/hadoopfile 

返回值:

成功返回 0 失败返回 -1.

### rm

用法: 

	hdfs dfs -rm [-f] [-r|-R] [-skipTrash] URI [URI ...]

删除文件.

参数:

- `-f` 参数 will not display a diagnostic message or modify the exit status to reflect an error if the file does not exist.
- `-R` 参数 deletes the directory and any content under it recursively.
- `-r` 参数 is equivalent to `-R`.
- `-skipTrash` 参数 will bypass trash, if enabled, and delete the specified file(s) immediately. This can be useful when it is necessary to delete files from an over-quota directory.

可以配合Unix的 `rm` 命令进行记忆。

示例:

	hdfs dfs -rm hdfs://nn.示例.com/file /user/hadoop/emptydir

返回值:

成功返回 0 失败返回 -1.

### rmr

用法: 

	hdfs dfs -rmr [-skipTrash] URI [URI ...]

递归删除文件及目录。

**Note:** This command is **deprecated**. Instead use `hdfs dfs -rm -r`

### setfacl

用法: 

	hdfs dfs -setfacl [-R] [-b|-k -m|-x <acl_spec> <path>]|[--set <acl_spec> <path>]

Sets Access Control Lists (ACLs) of files and directories.

参数:

- `-b` : Remove all but the base ACL entries. The entries for user, group and others are retained for compatibility with permission bits.
- `-k` : Remove the default ACL.
- `-R` : Apply operations to all files and directories recursively.
- `-m` : Modify ACL. New entries are added to the ACL, and existing entries are retained.
- `-x` : Remove specified ACL entries. Other ACL entries are retained.
- `--set` : Fully replace the ACL, discarding all existing entries. The `acl_spec` must include entries for user, group, and others for compatibility with permission bits.
- `acl_spec` : Comma separated list of ACL entries.
- `path` : File or directory to modify.

示例:

	hdfs dfs -setfacl -m user:hadoop:rw- /file
	hdfs dfs -setfacl -x user:hadoop /file
	hdfs dfs -setfacl -b /file
	hdfs dfs -setfacl -k /dir
	hdfs dfs -setfacl --set user::rw-,user:hadoop:rw-,group::r--,other::r-- /file
	hdfs dfs -setfacl -R -m user:hadoop:r-x /dir
	hdfs dfs -setfacl -m default:user:hadoop:r-x /dir

返回值:

成功返回 0 失败返回非 0.

### setfattr

用法: 

	hdfs dfs -setfattr -n name [-v value] | -x name <path>

Sets an extended attribute name and value for a file or directory.

参数:

- `-b` : Remove all but the base ACL entries. The entries for user, group and others are retained for compatibility with permission bits.
- `-n name` : The extended attribute name.
- `-v value` : The extended attribute value. There are three different encoding methods for the value. If the argument is enclosed in double quotes, then the value is the string inside the quotes. If the argument is prefixed with 0x or 0X, then it is taken as a hexadecimal number. If the argument begins with 0s or 0S, then it is taken as a base64 encoding.
- `-x name` : Remove the extended attribute.
- `path` : The file or directory.

示例:

	hdfs dfs -setfattr -n user.myAttr -v myValue /file
	hdfs dfs -setfattr -n user.noValue /file
	hdfs dfs -setfattr -x user.myAttr /file

返回值:

成功返回 0 失败返回非 0.

### setrep

用法: 

	hdfs dfs -setrep [-R] [-w] <numReplicas> <path>

改变一个文件的副本系数。`-R`选项用于递归改变目录下所有文件的副本系数。

参数:

- `-w` ： 等待复制完成，需要等待很长时间哟。
- `-R` ： 递归设计吧。

示例:

	hdfs dfs -setrep -w 3 /user/hadoop/dir1

返回值:

成功返回 0 失败返回 -1.

### stat

用法: 

	hdfs dfs -stat URI [URI ...]

指定路径的统计信息。

示例:

	hdfs dfs -stat path

返回值: 

成功返回 0 失败返回 -1.

### tail

用法: 

	hdfs dfs -tail [-f] URI

将文件尾部1K字节的内容输出到stdout。

参数:

- `-f` 跟在Unix中类似，文件新追加内容了，也会显示出来。

示例:

	hdfs dfs -tail pathname

返回值: 

成功返回 0 失败返回 -1.

### test

用法：

	hadoop fs -test -[ezd] URI

参数：
- `-e` 检查文件是否存在。如果存在则返回0。
- `-z` 检查文件是否是0字节。如果是则返回0。 
- `-d` 如果路径是个目录，则返回1，否则返回0。

示例：

	hadoop fs -test -e filename

### text

用法：

	hadoop fs -text <src> 

将源文件输出为文本格式。允许的格式是zip和TextRecordInputStream。

### touchz

用法：

	hadoop fs -touchz URI [URI …] 

创建一个0字节的空文件。

示例：

	hadoop -touchz pathname

返回值:

成功返回0，失败返回-1。

## 参考资料

- [Apache Hadoop 2.4.1 FileSystemShell](http://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html)
- [Apache Hadoop 2.6.0 FileSystemShell](http://hadoop.apache.org/docs/r2.6.0/hadoop-project-dist/hadoop-common/FileSystemShell.html)
- [Hadoop Shell命令](http://hadoop.apache.org/docs/r1.0.4/cn/hdfs_shell.html)