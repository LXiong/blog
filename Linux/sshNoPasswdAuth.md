# SSH 无密码认证

配置SSH无密码认证，在很多场景下是非常方便的，尤其是在管理大型集群服务时，避免了繁琐的密码验证，在安全级别越高的服务器上，通常密码的设置更复杂，配置SSH，不仅可以用密钥保证节点间通信的安全性，同时也降低了频繁输入密码登陆的耗时，大大提高了管理效率。

提到SSH无密码认证，对于大数据时代，让人自然而然的想到了Hadoop，读完本篇文章，为你搭建Hadoop集群迈出了第一步。

环境：

- Centos 6.5 64 虚拟机两台

## SSH无密码认证的原理

Master作为客户端，要实现无密码公钥认证，连接到服务器Salve上时，需要在Master上生成一个密钥对，包括一个公钥和一个私钥，而后将公钥复制到所有的Salve上。当Master通过SSH链接到Salve上时，Salve会生成一个随机数并用Master的公钥对随机数进行加密，并发送给Master。Master收到加密数之后再用私钥解密，并将解密数回传给Salve，Salve确认解密数无误之后就允许Master进行连接了。这就是一个公钥认证过程，期间不需要手工输入密码，重要的过程是将Master上产生的公钥复制到Salve上。

## SSH无密码认证的几种关系

通常情况下，一个集群服务下至少有一个Master和若干个Slave，那么无密码登录通常指的是由Master到任意一个Slave的无验证的单向登录，意思就是只能从Master登录到Slave是不需要密码的，但是如果你想从Slave无验证登录到Master，或者你想在Slave与Slave之间进行无验证登录，这些都是不可行的，除非，你进行了密钥对的双向验证，才可以双向登录，我们在这里先不去议论相互之间登录有没有意义，可能某些情况下或许需要这些方式。 

下面将为大家介绍SSH双向无密码认证的配置。

## SSH无密码认证

<table>
<tr>
<td>结点名</td>
<td>IP地址</td>
</tr>
<tr>
<td>master</td>
<td>169.254.206.129</td>
</tr>
<tr>
<td>slave1</td>
<td>169.254.206.131</td>
</tr>
</table>

### Master 单机无密码认证

> 使用root账户登录或切换到root账户

	sudo su
	# or
	su root
	# or
	su

> 修改主机名

	vi /etc/sysconfig/network

更改 129 机器的主机名为 master

	NETWORKING=yes
	HOSTNAME=master

更改 131 机器的主机名为 slave1

	NETWORKING=yes
	HOSTNAME=slave1

> 修改hosts配置

	vi /etc/hosts

在两台机器的hosts文件下方增加如下配置

	169.254.206.129	master
	169.254.206.131	slave1

> 创建hadoop用户

	useradd hadoop
	passwd hadoop
	# 输入两次相同的密码

注意用户名，密码保持一致。

> 在master机器上登入hadoop用户

	su hadoop

> 生成密钥对，并把公钥文件写入授权文件，并赋值权限

	ssh-keygen -t rsa -P ''
	cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
	chmod 600 ~/.ssh/authorized_keys

> 切换到root用户（或者使用sudo权限也行，但是我们上面没有给hadoop用户赋予sudo权限），配置sshd，执行如下操作

	su root
	# 如果是从root用户切换过来的可以直接exit退出hadoop用户回到root
	# exit
	vi /etc/ssh/sshd_config

将原来的

	#RSAAuthentication yes
	#PubkeyAuthentication yes
	#AuthorizedKeysFile	.ssh/authorized_keys

更改为（取消注释）

	RSAAuthentication yes
	PubkeyAuthentication yes
	AuthorizedKeysFile	.ssh/authorized_keys
	
重启sshd服务

	service sshd restart

至此，我们本机的SSH已经配置完毕。

> 本机SSH无密码认证测试

	# 切换为hadoop用户
	su hadoop
	# 登录localhost，按提示输入yes即可，不需密码
	ssh localhost
	yes
	# 退出使用master登录
	exit
	ssh master
	yes
	# 退出使用ip登录
	exit
	ssh 169.254.206.129

无论使用localhost，还是IP地址，或者是主机名，我们都可以顺利的进行本机的无密码认证。 
	
### Master无密码认证登录Slave

上面在Master单机完成了无密码认证，其实放到Slave上是一样的。

> 将master的公钥复制到slave的hadoop用户上

	scp ~/.ssh/id_rsa.pub hadoop@169.254.206.131:~/
	yes
	# 输入hadoop用户的密码
	# 拷贝成功

> 去slave1机器上，使用hadoop用户登录

	su hadoop
	cd ~
	mkdir -p .ssh
	chmod 700 ~/.ssh
	cat ~/id_rsa.pub >> ~/.ssh/authorized_keys
	chmod 600 ~/.ssh/authorized_keys

> 切换为root用户进行跟master中类似的sshd配置

	su root
	vi /etc/ssh/sshd_config
	# 将配置文件中的那三行注释取消注释
	# 取消注释
	# 重启sshd服务
	service sshd restart

> 回到master机的hadoop用户上，进行测试

	su hadoop
	ssh slave1
	ssh 169.254.206.131

都可以无密码认证了。

至此我们已经实现了master到slave的单向无密码认证。

如果你想实现双向认证，非常简单，把我们的master和slave操作颠倒一下就OK了。

## 参考资料

- [CentOS6.4之图解SSH无验证双向登陆配置](http://www.aboutyun.com/blog-3779-85.html)