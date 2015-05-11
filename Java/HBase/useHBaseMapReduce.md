# HBase MapReduce 操作

> 目录
> 
> - 读文件到HBase中
> - 读文件到多个HBase中
> - 从HBase中读取数据到文件
> - 参考资料

环境：

- Hadoop 2.0.0-cdh4.3.0
- Hbase-0.94.6-cdh4.3.0
- JDK 1.6

## 概述

HBase – Hadoop Database，是一个高可靠性、高性能、面向列、可伸缩的分布式存储系统，在HBase系统上运行批处理运算，最方便和实用的模型依然是MapReduce。

![](../../static/img/20150511143351.png)

HBase Table和Region的关系，比较类似HDFS File和Block的关系，HBase提供了配套的 `TableInputFormat` 和 `TableOutputFormat` API，可以方便的将HBase Table作为Hadoop MapReduce的Source和Sink，对于MapReduce Job应用开发人员来说，基本不需要关注HBase系统自身的细节。

## 读文件到HBase中

使用MapReduce读取HDFS中文件写入到HBase中。

### 准备测试数据

建立 course.txt 文件，内容如下( `\t` ( `tab` )分隔)：

	tom	math	98
	tom	chinese	95
	tom	history	80
	jack	math	86
	jack	history	88
	jack	chinese	90
	marry	math	88
	marry	history	96
	marry	chinese	100

放置到 HDFS 中，这里是：`/user/panda/input/course.txt`

### HBase表设计

建表：

- 表名：`HB_hyx_school`  
- 列族：`courses`

最终想要得到的结果为：

<table>
<tr>
<th>row</th><th>courses</th>
</tr>
<tr>
<td>tom</td><td>math:98</td>
</tr>
<tr>
<td>tom</td><td>chinese:95</td>
</tr>
<tr>
<td>tom</td><td>history:80</td>
</tr>
<tr>
<td>jack</td><td>math:86</td>
</tr>
<tr>
<td>jack</td><td>history:88</td>
</tr>
<tr>
<td>jack</td><td>chinese:90</td>
</tr>
<tr>
<td>marry</td><td>math:88</td>
</tr>
<tr>
<td>marry</td><td>history:96</td>
</tr>
<tr>
<td>marry</td><td>chinese:100</td>
</tr>
</table>

### MapReduce 实现

代码示例：

	import java.io.IOException;
	import java.security.PrivilegedExceptionAction;
	import java.util.Date;
	
	import org.apache.hadoop.conf.Configuration;
	import org.apache.hadoop.fs.Path;
	import org.apache.hadoop.hbase.client.Put;
	import org.apache.hadoop.hbase.mapreduce.TableMapReduceUtil;
	import org.apache.hadoop.hbase.mapreduce.TableReducer;
	import org.apache.hadoop.hbase.util.Bytes;
	import org.apache.hadoop.io.LongWritable;
	import org.apache.hadoop.io.NullWritable;
	import org.apache.hadoop.io.Text;
	import org.apache.hadoop.mapreduce.Job;
	import org.apache.hadoop.mapreduce.Mapper;
	import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
	import org.apache.hadoop.security.UserGroupInformation;
	
	import com.dragon.core.utils.DateUtils;
	import com.dragon.main.common.utils.HadoopUtils;
	
	/**
	 * @author panda
	 */
	public class ReadHDFSToHBase {
	
		public static class MapOne extends Mapper<LongWritable, Text, Text, Text> {
	
			@Override
			protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
				String[] vals = value.toString().split("\t");
				if (vals.length == 3) {
					String name = vals[0];
					String course = vals[1];
					String score = vals[2];
					context.write(new Text(name), new Text(course + "\t" + score));
				}
			};
		}
	
		public static class ReduceOne extends TableReducer<Text, Text, NullWritable> {
	
			@Override
			protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
				String row = key.toString();
				String family = "courses";
				byte[] familyB = Bytes.toBytes(family);
				Put put = new Put(Bytes.toBytes(row));
				for (Text text : values) {
					String[] vals = text.toString().split("\t");
					if (vals.length == 2) {
						String qualifier = vals[0];
						String value = vals[1];
						byte[] qualifierB = Bytes.toBytes(qualifier);
						byte[] valueB = Bytes.toBytes(value);
						put.add(familyB, qualifierB, valueB);
					}
				}
				if (!put.isEmpty()) {
					context.write(NullWritable.get(), put);
				}
	
			};
	
		}
	
		public static void runJob() throws Exception {
			Configuration conf = HadoopUtils.getDragonHbaseConfiguration();
			String input = "/user/panda/input/course.txt";
			try {
				Job jobOne = new Job(conf, "Dragon ReadHDFSToHBase Job 1 - " + DateUtils.format2date(new Date(), "yyyy-MM-dd HH:mm:ss"));
				jobOne.setJarByClass(ReadHDFSToHBase.class);
				jobOne.setMapperClass(MapOne.class);
				jobOne.setReducerClass(ReduceOne.class);
				jobOne.setNumReduceTasks(8);
				jobOne.setMapOutputKeyClass(Text.class);
				jobOne.setMapOutputValueClass(Text.class);
				FileInputFormat.addInputPath(jobOne, new Path(input));
				TableMapReduceUtil.initTableReducerJob("HB_hyx_school", ReduceOne.class, jobOne);
				jobOne.waitForCompletion(true);
			} catch (Exception e) {
				System.out.println("出现异常了");
			} finally {
	
			}
	
		}
	
		public static void main(final String[] args) throws IOException, InterruptedException {
	
			UserGroupInformation ugi = UserGroupInformation.createRemoteUser("panda");
			ugi.doAs(new PrivilegedExceptionAction<Void>() {
				@Override
				public Void run() throws Exception {
					runJob();
					return null;
				}
			});
		}
	
	}

运行结果：

![](../../static/img/20150511160136.png)

**注** 此代码非通用，但核心内容是可以借鉴的，下面分解介绍。

MapOne 读入文件，无需过多介绍，输出的 Key 为 name 是根据我们的业务需求决定的，因为我们的每一行就是一个人嘛。

ReduceOne 是本例的重点之一，我们采用继承 `TableReducer` 抽象类来向 HBase 中写数据：

	@InterfaceAudience.Public
	@InterfaceStability.Stable
	public abstract class TableReducer<KEYIN,VALUEIN,KEYOUT>
	extends org.apache.hadoop.mapreduce.Reducer<KEYIN,VALUEIN,KEYOUT,Mutation>

对于使用 `TableReducer` 来说，KEYOUT 没啥用，KEYOUT 一般指表名，而我们的表名已经在JOB中进行定义了，所以此处较随意，有兴趣的话，可以试试此处不为null的情况。

向 HBase 插数据当然是输出 Put 对象喽，一个 Put 实例代表一行。

	Put put = new Put(Bytes.toBytes(row));

输出嘛，将 Put 实例 write 出去。

	context.write(NullWritable.get(), put);

Job 部分核心部分其实就一行：

	TableMapReduceUtil.initTableReducerJob("HB_hyx_school", ReduceOne.class, jobOne);

指定输出到哪个表，使用哪个 Reducer。

至此，我们完成了从HDFS读取文件输出到HBase的操作。

## 读文件到多个HBase中

还是上面的数据，我们改变需求了，我要为每个学科建一个表。

### HBase表设计

数学表：

- 表名：`HB_hyx_math`
- 列族：`score`

语文表：

- 表名：`HB_hyx_chinese`
- 列族：`score`

历史表：

- 表名：`HB_hyx_history`
- 列族：`score`

我们要获得的结果你应该可以猜到了。

### MapReduce 实现

代码如下：

	import java.io.IOException;
	import java.security.PrivilegedExceptionAction;
	import java.util.Date;
	
	import org.apache.hadoop.conf.Configuration;
	import org.apache.hadoop.fs.Path;
	import org.apache.hadoop.hbase.client.Put;
	import org.apache.hadoop.hbase.io.ImmutableBytesWritable;
	import org.apache.hadoop.hbase.mapreduce.MultiTableOutputFormat;
	import org.apache.hadoop.hbase.mapreduce.TableMapReduceUtil;
	import org.apache.hadoop.hbase.util.Bytes;
	import org.apache.hadoop.io.LongWritable;
	import org.apache.hadoop.io.Text;
	import org.apache.hadoop.mapreduce.Job;
	import org.apache.hadoop.mapreduce.Mapper;
	import org.apache.hadoop.mapreduce.Reducer;
	import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
	import org.apache.hadoop.security.UserGroupInformation;
	
	import com.dragon.core.utils.DateUtils;
	import com.dragon.main.common.utils.HadoopUtils;
	
	/**
	 * @author panda
	 */
	public class ReadHDFSToMultipleHBase {
	
		public static class MapOne extends Mapper<LongWritable, Text, Text, Text> {
	
			@Override
			protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
				String[] vals = value.toString().split("\t");
				if (vals.length == 3) {
					String name = vals[0];
					String course = vals[1];
					String score = vals[2];
					context.write(new Text(name + "\t" + course), new Text(score));
				}
			};
		}
	
		public static class ReduceOne extends Reducer<Text, Text, ImmutableBytesWritable, Put> {
	
			@Override
			protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
				String[] arr = key.toString().split("\t");
				if (arr.length == 2) {
					String row = arr[0];
					String course = arr[1];
					String tableName = "HB_hyx_" + course;
					ImmutableBytesWritable table = new ImmutableBytesWritable(Bytes.toBytes(tableName));
					byte[] familyB = Bytes.toBytes("score");
					Put put = new Put(Bytes.toBytes(row));
					for (Text text : values) {
						byte[] valueB = Bytes.toBytes(text.toString());
						// 没有列了，qualifier填null即可
						put.add(familyB, null, valueB);
						if (!put.isEmpty()) {
							// 将数据填到指定的表中
							context.write(table, put);
						}
					}
				}
	
			};
	
		}
	
		public static void runJob() throws Exception {
			Configuration conf = HadoopUtils.getDragonHbaseConfiguration();
			String input = "/user/panda/input/course.txt";
			try {
				Job jobOne = new Job(conf, "Dragon ReadHDFSToMultipleHBase Job 1 - " + DateUtils.format2date(new Date(), "yyyy-MM-dd HH:mm:ss"));
				jobOne.setJarByClass(ReadHDFSToMultipleHBase.class);
				jobOne.setMapperClass(MapOne.class);
				jobOne.setReducerClass(ReduceOne.class);
				jobOne.setNumReduceTasks(1);
				jobOne.setMapOutputKeyClass(Text.class);
				jobOne.setMapOutputValueClass(Text.class);
				jobOne.setOutputKeyClass(ImmutableBytesWritable.class);
				jobOne.setOutputValueClass(Put.class);
				FileInputFormat.addInputPath(jobOne, new Path(input));
				jobOne.setOutputFormatClass(MultiTableOutputFormat.class);
				TableMapReduceUtil.addDependencyJars(jobOne);
		        TableMapReduceUtil.addDependencyJars(jobOne.getConfiguration());
				jobOne.waitForCompletion(true);
			} catch (Exception e) {
				System.out.println("出现异常了");
			} finally {
	
			}
	
		}
	
		public static void main(final String[] args) throws IOException, InterruptedException {
	
			UserGroupInformation ugi = UserGroupInformation.createRemoteUser("panda");
			ugi.doAs(new PrivilegedExceptionAction<Void>() {
				@Override
				public Void run() throws Exception {
					runJob();
					return null;
				}
			});
		}
	
	}

运行结果：

![](../../static/img/20150511180605.png)

代码仅供逻辑参考，现对核心内容进行分解。

MapOne 也不多说了，根据业务需求灵活变动即可。

ReduceOne 变动了，这里使用了最原始的 Reducer ，TableReducer 的父类，你要问：为什么不使用 TableReducer 了呢？嗯，这里完全可以用 TableReducer 做如下改动即可：

将

	public static class ReduceOne extends Reducer<Text, Text, ImmutableBytesWritable, Put> {

更改为：

	public static class ReduceOne extends TableReducer<Text, Text, ImmutableBytesWritable> {

即可。

多表输出的核心就是这个 ImmutableBytesWritable (KEYOUT)，它控制着数据往哪个表存，里面的内容不再赘述。

下面再谈一下Job。

Job中没有使用 TableMapReduceUtil 指定表了，而是变动了如下代码：

	// 指定了输出Key的类型
	jobOne.setOutputKeyClass(ImmutableBytesWritable.class);
	// 指定输出Value的类型
	jobOne.setOutputValueClass(Put.class);
	
	// 指定输出格式为多表输出（TableReducer默认的为TableOutputFormat）
	jobOne.setOutputFormatClass(MultiTableOutputFormat.class);
	// 添加HBase依赖环境（普通的Job是不会主动加载HBase依赖的）
	TableMapReduceUtil.addDependencyJars(jobOne);
	TableMapReduceUtil.addDependencyJars(jobOne.getConfiguration());

你应该有几个问题吧:

上面的例子为什么没有`setOutputKeyClass`,`setOutputValueClass`,`setOutputFormatClass`?

使用 `TableReducer` 不能实现多表输出吗？应该怎样设置？

请带着这些疑问自行测试（我不能告诉你我不知道呀）。

## 从HBase中读数据到文件

说完了写，该到读了。

读理所当然关系到 TableMapper , TableInputFormat了。

现在我们从 `HB_hyx_school` 中获取所有的学生姓名。

### MapReduce 实现

代码如下：

	import java.io.IOException;
	import java.security.PrivilegedExceptionAction;
	import java.util.Date;
	
	import org.apache.hadoop.conf.Configuration;
	import org.apache.hadoop.fs.FileSystem;
	import org.apache.hadoop.fs.Path;
	import org.apache.hadoop.hbase.KeyValue;
	import org.apache.hadoop.hbase.client.Put;
	import org.apache.hadoop.hbase.client.Result;
	import org.apache.hadoop.hbase.client.Scan;
	import org.apache.hadoop.hbase.io.ImmutableBytesWritable;
	import org.apache.hadoop.hbase.mapreduce.TableMapReduceUtil;
	import org.apache.hadoop.hbase.mapreduce.TableMapper;
	import org.apache.hadoop.hbase.util.Bytes;
	import org.apache.hadoop.io.NullWritable;
	import org.apache.hadoop.io.Text;
	import org.apache.hadoop.mapreduce.Job;
	import org.apache.hadoop.mapreduce.Reducer;
	import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
	import org.apache.hadoop.security.UserGroupInformation;
	
	import com.dragon.core.utils.DateUtils;
	import com.dragon.main.common.utils.HadoopUtils;
	
	/**
	 * @author panda
	 */
	public class ReadHBaseToHDFS {
	
		public static class MapOne extends TableMapper<ImmutableBytesWritable, Put> {
	
			@Override
			protected void map(ImmutableBytesWritable key, Result value, Context context) throws IOException, InterruptedException {
				context.write(key, resultToPut(key, value));
			}
			
		}
		
		/**
		 * 将result转化为put
		 * 
		 * @param key
		 * @param result
		 * @return
		 * @throws IOException
		 */
		private static Put resultToPut(ImmutableBytesWritable row, Result result) throws IOException {
	  		Put put = new Put(row.get());
	 		for (KeyValue kv : result.raw()) {
				put.add(kv);
			}
			return put;
	   	}
		
		public static class ReduceOne extends Reducer<ImmutableBytesWritable, Put, Text, NullWritable> {
	
			@Override
			protected void reduce(ImmutableBytesWritable key, Iterable<Put> values, Context context) throws IOException, InterruptedException {
				String row = Bytes.toString(key.get());
				context.write(new Text(row), NullWritable.get());
			}
			
		}
		
		public static void runJob() throws Exception {
			Configuration conf = HadoopUtils.getDragonHbaseConfiguration();
			FileSystem fs = FileSystem.get(conf);
			String output = "/user/panda/output/course";
			Path src = new Path(output);
			fs.delete(src, true);
			try {
				Scan scan = new Scan();
				Job jobOne = new Job(conf, "Dragon ReadHBaseToHDFS Job 1 - " + DateUtils.format2date(new Date(), "yyyy-MM-dd HH:mm:ss"));
				jobOne.setJarByClass(ReadHBaseToHDFS.class);
				TableMapReduceUtil.initTableMapperJob("HB_hyx_school", scan, MapOne.class, ImmutableBytesWritable.class, Put.class, jobOne);
				jobOne.setReducerClass(ReduceOne.class);
				jobOne.setNumReduceTasks(1);
				jobOne.setOutputKeyClass(Text.class);
				jobOne.setOutputValueClass(NullWritable.class);
				FileOutputFormat.setOutputPath(jobOne, new Path(output));
				jobOne.waitForCompletion(true);
				
			} catch (Exception e) {
				System.out.println("出现异常了");
			} finally {
	
			}
	
		}
	
		public static void main(final String[] args) throws IOException, InterruptedException {
	
			UserGroupInformation ugi = UserGroupInformation.createRemoteUser("panda");
			ugi.doAs(new PrivilegedExceptionAction<Void>() {
				@Override
				public Void run() throws Exception {
					runJob();
					return null;
				}
			});
		}
	
	}

输出数据：

	jack
	marry
	tom

MapOne 继承自 TableMapper

	@InterfaceAudience.Public
	@InterfaceStability.Stable
	public abstract class TableMapper<KEYOUT,VALUEOUT>
	extends org.apache.hadoop.mapreduce.Mapper<ImmutableBytesWritable,Result,KEYOUT,VALUEOUT>

从构造方法中可以得知，TableMapper 的 KEYOUT,VALUEOUT 对应 Mapper 的输出类型，而输入类型是确定的：ImmutableBytesWritable,Result.

ImmutableBytesWritable 对应 HBase 的 rowKey 我们这里就是 学生姓名 。

无需理会我的 `resultToPut`，本例中没实际用处。

ReduceOne 为普通的 Reducer，一看就懂了。

Job 阶段有以下几个点：

	Scan scan = new Scan();
	TableMapReduceUtil.initTableMapperJob("HB_hyx_school", scan, MapOne.class, ImmutableBytesWritable.class, Put.class, jobOne);

应该可以很容易看懂啥意思，不多说了。

当然，我们也可以不用 TableMapper , initTableMapperJob , 跟我们上面的输出到HBase类似，直接采用普通的 Mapper 配合 TableInputFormat 即可实现从 HBase 读取数据。

有兴趣的话，请自行测试，注意相应的输入输出及依赖等。

## 参考资料

- [HBase 官方文档 0.97](http://abloz.com/hbase/book.html#mapreduce)
- [hbase apidocs](http://hbase.apache.org/apidocs/org/apache/hadoop/hbase/mapreduce/package-summary.html)
- [mapreduce 只使用Mapper往多个hbase表中写数据](http://www.cnblogs.com/sixiweb/p/4044057.html)




