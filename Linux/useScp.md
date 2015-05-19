# Scp 文件传输

Scp 是Linux中非常流行的机器间文件传输命令。Scp基于ssh登录，操作起来比较方便。

用法: 

	scp [-1246BCpqrv] [-c cipher] [-F ssh_config] [-i identity_file] [-l limit] [-o ssh_option] [-P port] [-S program] [[user@]host1:]file1 ... [[user@]host2:]file2
	# DESCRIPTION :scp copies files between hosts on a network.

上面的参数挺多的，我们用到的不多，就不深究其作用了。

Scp 可以对文件或目录进行机器间的相互传输，所以我们的需求很简单：

- 从本地复制到远程
- 从远程复制到本地
- 文件的复制
- 目录的复制

下面结合示例来体验 Scp 命令的使用。

## 从本地复制到远程

	scp [options] srcfile dstfile

### 复制文件

命令格式：

	scp local_file remote_username@remote_ip:remote_folder 
	scp local_file remote_username@remote_ip:remote_file 
	scp local_file remote_ip:remote_folder 
	scp local_file remote_ip:remote_file 

第1,2个指定了用户名，命令执行后需要输入用户密码，第1个仅指定了远程的目录，文件名字不变，相当于直接复制，第2个指定了文件名，相当于复制并改名。

第3,4个没有指定用户名，命令执行后需要输入用户名和密码。

示例：

	scp /home/panda/1.mp3 panda@192.168.10.80:/home/panda/music/
	scp /home/panda/1.mp3 panda@www.example.com:/home/panda/music/001.mp3
	scp /home/panda/1.mp3 192.168.10.80:/home/panda/music/ 
	scp /home/panda/1.mp3 www.example.com:/home/panda/music/001.mp3

### 复制目录

命令格式：

	scp -r local_folder remote_username@remote_ip:remote_folder 
	scp -r local_folder remote_ip:remote_folder

示例：

	scp -r /home/panda/music/ panda@192.168.10.80:/home/panda/download/

## 从远程复制到本地

	scp [options] srcfile dstfile

命令格式是一样的，只需要调换后两个参数的位置即可，很容易理解的。

### 复制文件

命令格式：

	scp remote_username@remote_ip:remote_file local_folder  
	scp remote_username@remote_ip:remote_file local_file 
	scp remote_ip:remote_file local_folder 
	scp remote_ip:remote_file local_file 

第1,2个指定了用户名，命令执行后需要输入用户密码，第1个仅指定了本地的目录，文件名字不变，相当于直接复制，第2个指定了文件名，相当于复制并改名。

第3,4个没有指定用户名，命令执行后需要输入用户名和密码。

示例：

	scp panda@192.168.10.80:/home/panda/music/1.mp3 /home/panda/
	scp panda@www.example.com:/home/panda/music/001.mp3 /home/panda/1.mp3
	scp 192.168.10.80:/home/panda/music/001.mp3 /home/panda/1.mp3 
	scp www.example.com:/home/panda/music/001.mp3 /home/panda/1.mp3

### 复制目录

命令格式：

	scp -r remote_username@remote_ip:remote_folder local_folder
	scp -r remote_ip:remote_folder local_folder

示例：

	scp -r panda@192.168.10.80:/home/panda/download/music/ /home/panda/

## 注意事项

### 特殊端口

如果远程服务器防火墙有特殊限制，scp便要走特殊端口，这是经常遇到的情况。

	scp -P 2212 local_file username@remote_ip:remote_folder

### 用户权限

使用scp要注意所使用的用户是否具有可读取或写入远程服务器相应文件的权限。

## 参考资料

- [scp 命令](http://www.cnblogs.com/hitwtx/archive/2011/11/16/2251254.html)
- [利用scp 远程上传下载文件/文件夹](http://www.cnblogs.com/no7dw/archive/2012/07/07/2580307.html)


