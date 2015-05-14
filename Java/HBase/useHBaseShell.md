# HBase Shell 命令

HBase 为用户提供了一个非常方便的使用方式, 我们称之为 “HBase Shell”。  
HBase Shell 提供了大多数的 HBase 命令, 通过 HBase Shell 用户可以方便地创建、删除及修改表, 还可以向表中添加数据、列出表中的相关信息等。

环境：

- Hbase-0.94.6-cdh4.3.0

## 准备测试数据

我们要创建及操作的表结构与数据是这样的：

> `HB_hyx_school`

<table>
	<tr>
		<th rowspan="2">row</th>
		<th rowspan="2">grade</th>
		<th colspan="3">courses</th>
	</tr>
	<tr>
		<th>chinese</th>
		<th>math</th>
		<th>history</th>
	</tr>
	<tr>
		<td>tom</td>
		<td>1</td>
		<td>95</td>
		<td>98</td>
		<td>80</td>
	</tr>
	<tr>
		<td>jack</td>
		<td>2</td>
		<td>90</td>
		<td>86</td>
		<td>88</td>
	</tr>
	<tr>
		<td>marry</td>
		<td>1</td>
		<td>100</td>
		<td>88</td>
		<td>96</td>
	</tr>
</table>

下面我们要进行的所有操作都是基于这张表的。

## HBase Shell 帮助

在装有 HBase 的机器上执行：

```shell
hbase shell
```

进入到 HBase 提供的命令交互页面：

	[panda@slave4 ~]$ hbase shell
	HBase Shell; enter 'help<RETURN>' for list of supported commands.
	Type "exit<RETURN>" to leave the HBase Shell
	Version 0.94.6-cdh4.3.0, rUnknown, Mon May 27 20:22:05 PDT 2013
	
	hbase(main):001:0> 

键入 `help` 指令，获取 HBase 提供的帮助：

	hbase(main):001:0> help
	HBase Shell, version 0.94.6-cdh4.3.0, rUnknown, Mon May 27 20:22:05 PDT 2013
	Type 'help "COMMAND"', (e.g. 'help "get"' -- the quotes are necessary) for help on a specific command.
	Commands are grouped. Type 'help "COMMAND_GROUP"', (e.g. 'help "general"') for help on a command group.
	
	COMMAND GROUPS:
	  Group name: general
	  Commands: status, table_help, version, whoami
	
	  Group name: ddl
	  Commands: alter, alter_async, alter_status, create, describe, disable, disable_all, drop, drop_all, enable, enable_all, exists, get_table, is_disabled, is_enabled, list, show_filters
	
	  Group name: dml
	  Commands: count, delete, deleteall, get, get_counter, incr, put, scan, truncate
	
	  Group name: tools
	  Commands: assign, balance_switch, balancer, close_region, compact, flush, hlog_roll, major_compact, move, split, unassign, zk_dump
	
	  Group name: replication
	  Commands: add_peer, disable_peer, enable_peer, list_peers, remove_peer, start_replication, stop_replication
	
	  Group name: snapshot
	  Commands: clone_snapshot, delete_snapshot, list_snapshots, restore_snapshot, snapshot
	
	  Group name: security
	  Commands: grant, revoke, user_permission
	
	SHELL USAGE:
	Quote all names in HBase Shell such as table and column names.  Commas delimit
	command parameters.  Type <RETURN> after entering a command to run it.
	Dictionaries of configuration used in the creation and alteration of tables are
	Ruby Hashes. They look like this:
	
	  {'key1' => 'value1', 'key2' => 'value2', ...}
	
	and are opened and closed with curley-braces.  Key/values are delimited by the
	'=>' character combination.  Usually keys are predefined constants such as
	NAME, VERSIONS, COMPRESSION, etc.  Constants do not need to be quoted.  Type
	'Object.constants' to see a (messy) list of all constants in the environment.
	
	If you are using binary keys or values and need to enter them in the shell, use
	double-quote'd hexadecimal representation. For example:
	
	  hbase> get 't1', "key\x03\x3f\xcd"
	  hbase> get 't1', "key\003\023\011"
	  hbase> put 't1', "test\xef\xff", 'f1:', "\x01\x33\x40"
	
	The HBase shell is the (J)Ruby IRB with the above HBase-specific commands added.
	For more on the HBase Shell, see http://hbase.apache.org/docs/current/book.html
	hbase(main):002:0> 

`help` 命令给出了基本的用法及教你如何获取更多的帮助信息。

HBase有很多命令，`help`不会直接把所有命令的使用方法直接列出（那太长了），所以HBase将其分为若干组，就是上面显示的：

	COMMAN GROUPS:
		Group name: general
		...

有如下组及命令（从上面的帮助信息中摘出来的）：

<table>
	<tr>
		<th>Group Name</th>
		<th>Command</th>
		<th>Desc</th>
	</tr>
	<tr>
		<td rowspan="4">general</td>
		<td>status</td>
		<td>返回HBase集群的状态</td>
	</tr>
	<tr>
		<td>table_help</td>
		<td></td>
	</tr>
	<tr>
		<td>version</td>
		<td>返回HBase版本信息</td>
	</tr>
	<tr>
		<td>whoami</td>
		<td>显示使用哪个用户</td>
	</tr>
	<tr>
		<td rowspan="17">ddl</td>
		<td>alter</td>
		<td>修改列族模式</td>
	</tr>
	<tr>
		<td>alter_async</td>
		<td></td>
	</tr>
	<tr>
		<td>alter_status</td>
		<td></td>
	</tr>
	<tr>
		<td>create</td>
		<td>创建表</td>
	</tr>
	<tr>
		<td>describe</td>
		<td>显示表相关的详细信息</td>
	</tr>
	<tr>
		<td>disable</td>
		<td>使表无效</td>
	</tr>
	<tr>
		<td>disable_all</td>
		<td>`disable_all 't.*'` 使所有t开头的表无效（`t.*`为正则表达式）</td>
	</tr>
	<tr>
		<td>drop</td>
		<td>删除表</td>
	</tr>
	<tr>
		<td>drop_all</td>
		<td>删除所有（匹配）表</td>
	</tr>
	<tr>
		<td>enable</td>
		<td>使表有效</td>
	</tr>
	<tr>
		<td>enable_all</td>
		<td>使所有（匹配）表有效</td>
	</tr>
	<tr>
		<td>exists</td>
		<td>测试表是否存在</td>
	</tr>
	<tr>
		<td>get_table</td>
		<td>返回一个表对象</td>
	</tr>
	<tr>
		<td>is_disabled</td>
		<td>是否是无效的</td>
	</tr>
	<tr>
		<td>is_enabled</td>
		<td>是否是有效</td>
	</tr>
	<tr>
		<td>list</td>
		<td>列出所有表</td>
	</tr>
	<tr>
		<td>show_filters</td>
		<td></td>
	</tr>
	<tr>
		<td rowspan="9">dml</td>
		<td>count</td>
		<td>统计表中行的数量</td>
	</tr>
	<tr>
		<td>delete</td>
		<td>删除指定对象值（可以为表，行，列对应的值）</td>
	</tr>
	<tr>
		<td>deleteall</td>
		<td>删除指定行的所有元素</td>
	</tr>
	<tr>
		<td>get</td>
		<td>获取行或单元的值</td>
	</tr>
	<tr>
		<td>get_counter</td>
		<td></td>
	</tr>
	<tr>
		<td>incr</td>
		<td>增加指定表，行，列的值</td>
	</tr>
	<tr>
		<td>put</td>
		<td>向指定的表单元添加值</td>
	</tr>
	<tr>
		<td>scan</td>
		<td>扫描表获取对应的值</td>
	</tr>
	<tr>
		<td>truncate</td>
		<td>重新创建指定表</td>
	</tr>
	<tr>
		<td rowspan="12">tools</td>
		<td>assign</td>
		<td></td>
	</tr>
	<tr>
		<td>balance_switch</td>
		<td></td>
	</tr>
	<tr>
		<td>balancer</td>
		<td></td>
	</tr>
	<tr>
		<td>close_region</td>
		<td></td>
	</tr>
	<tr>
		<td>compact</td>
		<td></td>
	</tr>
	<tr>
		<td>flush</td>
		<td></td>
	</tr>
	<tr>
		<td>hlog_roll</td>
		<td></td>
	</tr>
	<tr>
		<td>major_compact</td>
		<td></td>
	</tr>
	<tr>
		<td>move</td>
		<td></td>
	</tr>
	<tr>
		<td>split</td>
		<td></td>
	</tr>
	<tr>
		<td>unassign</td>
		<td></td>
	</tr>
	<tr>
		<td>zk_dump</td>
		<td></td>
	</tr>
	<tr>
		<td rowspan="7">replication</td>
		<td>add_peer</td>
		<td></td>
	</tr>
	<tr>
		<td>disable_peer</td>
		<td></td>
	</tr>
	<tr>
		<td>enable_peer</td>
		<td></td>
	</tr>
	<tr>
		<td>list_peers</td>
		<td></td>
	</tr>
	<tr>
		<td>remove_peer</td>
		<td></td>
	</tr>
	<tr>
		<td>start_replication</td>
		<td></td>
	</tr>
	<tr>
		<td>stop_replication</td>
		<td></td>
	</tr>
	<tr>
		<td rowspan="5">snapshot</td>
		<td>clone_snapshot</td>
		<td></td>
	</tr>
	<tr>
		<td>delete_snapshot</td>
		<td></td>
	</tr>
	<tr>
		<td>list_snapshots</td>
		<td></td>
	</tr>
	<tr>
		<td>restore_snapshot</td>
		<td></td>
	</tr>
	<tr>
		<td>snapshot</td>
		<td></td>
	</tr>
	<tr>
		<td rowspan="3">security</td>
		<td>grant</td>
		<td></td>
	</tr>
	<tr>
		<td>revoke</td>
		<td></td>
	</tr>
	<tr>
		<td>user_permission</td>
		<td></td>
	</tr>
	<tr>
		<td rowspan="2">其他</td>
		<td>exit</td>
		<td>退出HBase shell</td>
	</tr>
	<tr>
		<td>quit</td>
		<td>退出HBase shell</td>
	</tr>
</table>

想获取某个组的命令帮助，或者想获取某个命令的介绍，在 help 后面跟上组名或命令名。

	help "dml"
	help "scan"
	# 双引号是必要的

例：

	hbase(main):008:0> help "get"
	Get row or cell contents; pass table name, row, and optionally
	a dictionary of column(s), timestamp, timerange and versions. Examples:
	
	  hbase> get 't1', 'r1'
	  hbase> get 't1', 'r1', {TIMERANGE => [ts1, ts2]}
	  hbase> get 't1', 'r1', {COLUMN => 'c1'}
	  hbase> get 't1', 'r1', {COLUMN => ['c1', 'c2', 'c3']}
	  hbase> get 't1', 'r1', {COLUMN => 'c1', TIMESTAMP => ts1}
	  hbase> get 't1', 'r1', {COLUMN => 'c1', TIMERANGE => [ts1, ts2], VERSIONS => 4}
	  hbase> get 't1', 'r1', {COLUMN => 'c1', TIMESTAMP => ts1, VERSIONS => 4}
	  hbase> get 't1', 'r1', {FILTER => "ValueFilter(=, 'binary:abc')"}
	  hbase> get 't1', 'r1', 'c1'
	  hbase> get 't1', 'r1', 'c1', 'c2'
	  hbase> get 't1', 'r1', ['c1', 'c2']
	
	The same commands also can be run on a reference to a table (obtained via get_table or
	 create_table). Suppose you had a reference t to table 't1', the corresponding commands would be:
	
	  hbase> t.get 'r1'
	  hbase> t.get 'r1', {TIMERANGE => [ts1, ts2]}
	  hbase> t.get 'r1', {COLUMN => 'c1'}
	  hbase> t.get 'r1', {COLUMN => ['c1', 'c2', 'c3']}
	  hbase> t.get 'r1', {COLUMN => 'c1', TIMESTAMP => ts1}
	  hbase> t.get 'r1', {COLUMN => 'c1', TIMERANGE => [ts1, ts2], VERSIONS => 4}
	  hbase> t.get 'r1', {COLUMN => 'c1', TIMESTAMP => ts1, VERSIONS => 4}
	  hbase> t.get 'r1', 'c1'
	  hbase> t.get 'r1', 'c1', 'c2'
	  hbase> t.get 'r1', ['c1', 'c2']
	hbase(main):009:0> 

**备注**:写错 HBase Shell 命令时用键盘上的 “Delete” 进行删除, “Backspace” 不起作用。

## HBase Shell 实战

命令太多，我们拿我们最常用的来实践一下。

进入 HBase Shell 交互页面。

> 列出目前所有表

	list

> 创建 `HB_hyx_school` 表

	hbase(main):012:0> create 'HB_hyx_school', 'grade', 'courses'
	0 row(s) in 1.1000 seconds
	
	=> Hbase::Table - HB_hyx_school
	hbase(main):013:0> list
	TABLE                                                                                                   
	HB_hyx_school                                                                                                             
	1 row(s) in 0.0890 seconds
	
	hbase(main):014:0>

> 查看表结构

	hbase(main):014:0> describe 'HB_hyx_school'
	DESCRIPTION                                                                   ENABLED                                   
	 {NAME => 'HB_hyx_school', FAMILIES => [{NAME => 'courses', DATA_BLOCK_ENCODI true                                      
	 NG => 'NONE', BLOOMFILTER => 'NONE', REPLICATION_SCOPE => '0', VERSIONS => '                                           
	 3', COMPRESSION => 'NONE', MIN_VERSIONS => '0', TTL => '2147483647', KEEP_DE                                           
	 LETED_CELLS => 'false', BLOCKSIZE => '65536', IN_MEMORY => 'false', ENCODE_O                                           
	 N_DISK => 'true', BLOCKCACHE => 'true'}, {NAME => 'grade', DATA_BLOCK_ENCODI                                           
	 NG => 'NONE', BLOOMFILTER => 'NONE', REPLICATION_SCOPE => '0', VERSIONS => '                                           
	 3', COMPRESSION => 'NONE', MIN_VERSIONS => '0', TTL => '2147483647', KEEP_DE                                           
	 LETED_CELLS => 'false', BLOCKSIZE => '65536', IN_MEMORY => 'false', ENCODE_O                                           
	 N_DISK => 'true', BLOCKCACHE => 'true'}]}                                                                              
	1 row(s) in 0.0620 seconds
	
	hbase(main):015:0>

> 按测试数据结构插入数据

插入数据使用： `put 'table', 'row', 'family:qualifier', 'value'`

	put 'HB_hyx_school', 'tom', 'grade:', '1'
	put 'HB_hyx_school', 'jack', 'grade', '2'
	put 'HB_hyx_school', 'marry', 'grade', '2'
	put 'HB_hyx_school', 'tom', 'courses:chinese', '95'
	put 'HB_hyx_school', 'jack', 'courses:chinese', '90'
	put 'HB_hyx_school', 'marry', 'courses:chinese', '100'
	put 'HB_hyx_school', 'tom', 'courses:math', '98'
	put 'HB_hyx_school', 'jack', 'courses:math', '86'
	put 'HB_hyx_school', 'marry', 'courses:math', '88'
	put 'HB_hyx_school', 'tom', 'courses:history', '80'
	put 'HB_hyx_school', 'jack', 'courses:history', '88'
	put 'HB_hyx_school', 'marry', 'courses:history', '96'

只有列族没有列的可以加 `:` 也可以不加，上面的 `grade`

> 查看插入的数据 `scan 'table'`

	hbase(main):042:0> scan 'HB_hyx_school'
	ROW                             COLUMN+CELL                                                                             
	 jack                           column=courses:chinese, timestamp=1431585353746, value=90                               
	 jack                           column=courses:history, timestamp=1431585423839, value=88                               
	 jack                           column=courses:math, timestamp=1431585392539, value=86                                  
	 jack                           column=grade:, timestamp=1431584740397, value=2                                         
	 marry                          column=courses:chinese, timestamp=1431585368244, value=100                              
	 marry                          column=courses:history, timestamp=1431585433923, value=96                               
	 marry                          column=courses:math, timestamp=1431585402958, value=88                                  
	 marry                          column=grade:, timestamp=1431584752027, value=2                                         
	 tom                            column=courses:chinese, timestamp=1431585342555, value=95                               
	 tom                            column=courses:history, timestamp=1431585414179, value=80                               
	 tom                            column=courses:math, timestamp=1431585380994, value=98                                  
	 tom                            column=grade:, timestamp=1431584707853, value=1                                         
	3 row(s) in 0.0210 seconds
	
	hbase(main):043:0>

发现 marry 的信息输入错了，应该是 1 年级的。

> 更改 marry 的信息，仍然是 `put` .

	put 'HB_hyx_school', 'marry', 'grade', '1'

> 查看 marry 的信息 `get` .

	# 查看年级
	hbase(main):047:0> get 'HB_hyx_school', 'marry', 'grade'
	COLUMN                             CELL                                                                                               
	 grade:                            timestamp=1431585597274, value=1                                                                   
	1 row(s) in 0.0080 seconds
	
	# 查看数学成绩
	hbase(main):048:0> get 'HB_hyx_school', 'marry', 'courses:math'
	COLUMN                             CELL                                                                                               
	 courses:math                      timestamp=1431585402958, value=88                                                                  
	1 row(s) in 0.0040 seconds

	# 查看英语和历史的成绩
	hbase(main):049:0> get 'HB_hyx_school', 'marry', 'courses:chinese', 'courses:history'
	COLUMN                             CELL                                                                                               
	 courses:chinese                   timestamp=1431585368244, value=100                                                                 
	 courses:history                   timestamp=1431585433923, value=96                                                                  
	2 row(s) in 0.0060 seconds

	# 查看所有信息
	hbase(main):050:0> get 'HB_hyx_school', 'marry'
	COLUMN                             CELL                                                                                               
	 courses:chinese                   timestamp=1431585368244, value=100                                                                 
	 courses:history                   timestamp=1431585433923, value=96                                                                  
	 courses:math                      timestamp=1431585402958, value=88                                                                  
	 grade:                            timestamp=1431585597274, value=1                                                                   
	4 row(s) in 0.0120 seconds

get 命令手册：

	hbase> get 't1', 'r1'
	hbase> get 't1', 'r1', {TIMERANGE => [ts1, ts2]}
	hbase> get 't1', 'r1', {COLUMN => 'c1'}
	hbase> get 't1', 'r1', {COLUMN => ['c1', 'c2', 'c3']}
	hbase> get 't1', 'r1', {COLUMN => 'c1', TIMESTAMP => ts1}
	hbase> get 't1', 'r1', {COLUMN => 'c1', TIMERANGE => [ts1, ts2], VERSIONS => 4}
	hbase> get 't1', 'r1', {COLUMN => 'c1', TIMESTAMP => ts1, VERSIONS => 4}
	hbase> get 't1', 'r1', {FILTER => "ValueFilter(=, 'binary:abc')"}
	hbase> get 't1', 'r1', 'c1'
	hbase> get 't1', 'r1', 'c1', 'c2'
	hbase> get 't1', 'r1', ['c1', 'c2']

> 查看两位同学的数学和历史成绩 `scan` .

	hbase(main):056:0> scan 'HB_hyx_school', {COLUMNS => ['courses:math', 'courses:history'], LIMIT => 2}
	ROW                                COLUMN+CELL                                                                                        
	 jack                              column=courses:history, timestamp=1431585423839, value=88                                          
	 jack                              column=courses:math, timestamp=1431585392539, value=86                                             
	 marry                             column=courses:history, timestamp=1431585433923, value=96                                          
	 marry                             column=courses:math, timestamp=1431585402958, value=88                                             
	2 row(s) in 0.0090 seconds

scan 命令手册（部分）：

	hbase> scan 't1', {COLUMNS => ['c1', 'c2'], LIMIT => 10, STARTROW => 'xyz'}
	hbase> scan 't1', {COLUMNS => 'c1', TIMERANGE => [1303668804, 1303668904]}
	hbase> scan 't1', {FILTER => "(PrefixFilter ('row2') AND (QualifierFilter (>=, 'binary:xyz'))) AND (TimestampsFilter ( 123, 456))"}
	hbase> scan 't1', {FILTER => org.apache.hadoop.hbase.filter.ColumnPaginationFilter.new(1, 0)}

**备注**：`COLUMN` 和 `COLUMNS` （`get`,`scan`中都可使用）现在好像不区分了，我测试的都可以使用，只是必须大写是已知的。

关于过滤器，有两种使用方式

- 通过过滤器的名称，例：`PrefixFilter('row2')`
- 使用过滤器的包名，例：`org.apache.hadoop.hbase.filter.ColumnPaginationFilter.new(1, 0)`

> 查看有多少行

	hbase(main):065:0> count 'HB_hyx_school'
	3 row(s) in 0.0180 seconds

> 删除数据 `delete` , `deleteall`

	# 删除jack的数学成绩
	delete 'HB_hyx_school', 'jack', 'courses:math'
	# 删除jack的历史成绩
	deleteall 'HB_hyx_school', 'jack', 'courses:history'
	# 删除jack的年级信息
	delete 'HB_hyx_school', 'jack', 'grade'
	# 删除marry的年级信息
	deleteall 'HB_hyx_school', 'marry', 'grade'

	# 删含有列的列族下面的操作是删不掉的
	delete[all] 'HB_hyx_school', 'tom', 'courses'

	# 删除一整行（deleteall）
	deleteall 'HB_hyx_school', 'jack'

> 查看表是否禁用

	hbase(main):092:0> is_disabled 'HB_hyx_school'
	false                                                                                                                                 
	0 row(s) in 0.0070 seconds

> 禁用表

	hbase(main):093:0> disable 'HB_hyx_school'
	0 row(s) in 2.1100 seconds
	
	hbase(main):094:0> is_disabled 'HB_hyx_school'
	true                                                                                                                                  
	0 row(s) in 0.0050 seconds

> 启用表

	hbase(main):095:0> enable 'HB_hyx_school'
	0 row(s) in 2.0980 seconds
	
	hbase(main):096:0> is_enabled 'HB_hyx_school'
	true                                                                                                                                  
	0 row(s) in 0.0020 seconds

> 重置表（清空数据重新建立表）

	hbase(main):097:0> truncate 'HB_hyx_school'
	Truncating 'HB_hyx_school' table (it may take a while):
	 - Disabling table...
	 - Dropping table...
	 - Creating table...
	0 row(s) in 4.3460 seconds

	hbase(main):098:0> scan 'HB_hyx_school'
	ROW                                COLUMN+CELL                                                                                        
	0 row(s) in 0.0090 seconds

> 更新表结构

更新表结构需要先禁用表

	# 添加info列族
	hbase(main):101:0> disable 'HB_hyx_school'
	0 row(s) in 2.0930 seconds
	
	hbase(main):102:0> alter 'HB_hyx_school', NAME => 'info'
	Updating all regions with the new schema...
	1/1 regions updated.
	Done.
	0 row(s) in 1.3170 seconds

	# 添加 hobby area 列族
	hbase(main):105:0> alter 'HB_hyx_school', {NAME => 'hobby'}, {NAME => 'area'}
	Updating all regions with the new schema...
	1/1 regions updated.
	Done.
	Updating all regions with the new schema...
	1/1 regions updated.
	Done.
	0 row(s) in 4.4810 seconds

	# 删除 area 列族
	hbase(main):107:0> alter 'HB_hyx_school', NAME => 'area', METHOD => 'delete'
	Updating all regions with the new schema...
	1/1 regions updated.
	Done.
	0 row(s) in 1.2610 seconds

其他高端修改，请看帮助手册。

> 删除表

测试完毕，删掉表，删表也需要表是禁用状态。

	hbase(main):109:0> drop 'HB_hyx_school'
	0 row(s) in 1.1120 seconds
	
	hbase(main):110:0> list
	TABLE                                                                                                                   
	0 row(s) in 0.0510 seconds
	
	hbase(main):111:0>

基础操作到此讲述完毕，遇到任何需求记得 `help "command"` 寻求帮助。

## HBase Shell 脚本

进入 HBase Shell 交互界面虽然可以操作，但是毕竟太过效率低下，那就试试 shell 脚本批量执行吧。

> `test.hbaseshell` 内容

	vim test.hbaseshell

	# 输入如下内容
	list
	create 'HB_hyx_school', 'grade', 'courses'
	put 'HB_hyx_school', 'tom', 'grade:', '1'
	put 'HB_hyx_school', 'jack', 'grade', '2'
	put 'HB_hyx_school', 'marry', 'grade', '2'
	put 'HB_hyx_school', 'tom', 'courses:chinese', '95'
	put 'HB_hyx_school', 'jack', 'courses:chinese', '90'
	put 'HB_hyx_school', 'marry', 'courses:chinese', '100'
	put 'HB_hyx_school', 'tom', 'courses:math', '98'
	put 'HB_hyx_school', 'jack', 'courses:math', '86'
	put 'HB_hyx_school', 'marry', 'courses:math', '88'
	put 'HB_hyx_school', 'tom', 'courses:history', '80'
	put 'HB_hyx_school', 'jack', 'courses:history', '88'
	put 'HB_hyx_school', 'marry', 'courses:history', '96'

保存，执行下面的命令：

```shell
hbase shell test.hbaseshell
```

	[panda@slave4 ~]$ hbase shell test.hbaseshell 
	15/05/14 16:09:22 WARN conf.Configuration: hadoop.native.lib is deprecated. Instead, use io.native.lib.available
	TABLE                                                                                                                                                                                                                                      
	HB_dmp_action2                                                                                                                                                                                                                             
	HB_dmp_data                                                                                                                                                                                                                                
	HB_dmp_data2                                                                                                                                                                                                                               
	HB_dmp_search                                                                                                                                                                                                                              
	HB_dmp_search2                                                                                                                                                                                                                             
	HB_dmp_user                                                                                                                                                                                                                                
	HB_hyx_chinese                                                                                                                                                                                                                             
	HB_hyx_history                                                                                                                                                                                                                             
	HB_hyx_math                                                                                                                                                                                                                                
	HB_hyx_test                                                                                                                                                                                                                                
	HB_jiuyu_data                                                                                                                                                                                                                              
	HB_log_data                                                                                                                                                                                                                                
	HB_radius_data                                                                                                                                                                                                                             
	HB_test                                                                                                                                                                                                                                    
	HB_ua_data                                                                                                                                                                                                                                 
	users                                                                                                                                                                                                                                      
	16 row(s) in 0.5200 seconds
	
	0 row(s) in 1.1110 seconds
	
	0 row(s) in 0.0710 seconds
	
	0 row(s) in 0.0050 seconds
	
	0 row(s) in 0.0070 seconds
	
	0 row(s) in 0.0030 seconds
	
	0 row(s) in 0.0030 seconds
	
	0 row(s) in 0.0030 seconds
	
	0 row(s) in 0.0030 seconds
	
	0 row(s) in 0.0030 seconds
	
	0 row(s) in 0.0020 seconds
	
	0 row(s) in 0.0020 seconds
	
	0 row(s) in 0.0030 seconds
	
	0 row(s) in 0.0030 seconds
	
	HBase Shell; enter 'help<RETURN>' for list of supported commands.
	Type "exit<RETURN>" to leave the HBase Shell
	Version 0.94.6-cdh4.3.0, rUnknown, Mon May 27 20:22:05 PDT 2013
	
	hbase(main):001:0> 

我们的数据瞬间就回来了，挺好。

## 参考资料

- [Hbase shell详情](http://www.cnblogs.com/linjiqin/archive/2013/03/08/2949339.html)
- [hbase shell基础和常用命令详解](http://www.jb51.net/article/31172.htm)




