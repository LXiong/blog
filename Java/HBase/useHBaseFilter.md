# 客户端请求过滤器

Get 和 Scan 实例都可以用 `setFilter(Filter filter)` 配置。  
过滤器可能会搞混，因为有很多类型的过滤器，最好通过理解过滤器功能组来了解他们。

环境：

- HBase版本：HBase 0.94.6-cdh4.3.0
- Eclipse Indigo
- hbase-0.94.6-cdh4.3.0-security.jar
- 过滤器所属包：org.apache.hadoop.hbase.filter

## CompareOp 字节(Binary)比较类型

下面有用到比较类型的均来自下面的七种之一。

- `EQUAL` : 等于
- `GREATER` : 大于
- `GREATER_OR_EQUAL` : 大于等于
- `LESS` : 小于
- `LESS_OR_EQUAL` : 小于等于
- `NOT_EQUAL` : 不等于
- `NO_OP` : 没有操作，默认比较

例：

	// 小于等于"rowId"的行
	Filter filter = new RowFilter(CompareOp.LESS_OR_EQUAL, new BinaryComparator(Bytes.toBytes("rowId")));

## BitwiseOp 位(bit)比较类型

位比较类型，与，或，异或

- `AND` : 与
- `OR` : 或
- `XOR` : 异或

例：暂时不知道怎么使 ^_^ 。

## 比较器

HBase提供了多个不同用途的比较器。

- `BinaryComparator` : 最普通的比较器，对字节使用指定比较类型进行比较。上面的例子使用的就是它。
- `BinaryPrefixComparator` : 前缀比较器，匹配前缀。只能使用`EQUAL`和`NOT_EQUAL`。

		// 匹配rowkey是"log_"开头的行
		Filter filter = new RowFilter(CompareOp.EQUAL, new BinaryPrefixComparator(Bytes.toBytes("log_")));

- `BitComparator` : 应该跟BitwiseOp一起使用，暂时不会使。
- `NullComparator` : 判断为空或者不为空。具体用法不详。只能使用`EQUAL`和`NOT_EQUAL`。

	在使用`NullComparator`需要注意的是，HBase对空的定义:  
	row1的col1列没有值，在HBase中，不为空  
	row2的col1列不存在，在HBase中，表示空

		Filter filter = new RowFilter(CompareOp.EQUAL, new NullComparator());

- `RegexStringComparator` : 根据一个正则表达式去匹配。只能使用`EQUAL`和`NOT_EQUAL`。

		// 正则匹配末尾为"_33"的行，正则基本都使用"EQUAL"
		Filter filter = new RowFilter(CompareOp.EQUAL, new RegexStringComparator(".*_33"));
	正则默认是Java的正则样式 

- `SubstringComparator` : 判断是否包含指定内容。只能使用`EQUAL`和`NOT_EQUAL`。

		// 匹配包含"abc"的行，也是"EQUAL"
		Filter filter = new RowFilter(CompareOp.EQUAL, new SubstringComparator("abc"));

## 结构(Structural)过滤器

结构过滤器可以包含其他过滤器

### FilterList

FilterList 代表一个过滤器列表，过滤器间具有 `FilterList.Operator.MUST_PASS_ALL` 或 `FilterList.Operator.MUST_PASS_ONE` 关系。

下面示例展示两个过滤器的'或'关系（检查同一属性的'my value' 或'my other value' ）.

	Scan scan = new Scan();
	FilterList list = new FilterList(FilterList.Operator.MUST_PASS_ONE);
	SingleColumnValueFilter filter1 = new SingleColumnValueFilter(Bytes.toBytes("family"), Bytes.toBytes("qualifier"), CompareOp.EQUAL, Bytes.toBytes("my value"));
	list.addFilter(filter1);
	SingleColumnValueFilter filter2 = new SingleColumnValueFilter(Bytes.toBytes("family"), Bytes.toBytes("qualifier"), CompareOp.EQUAL, Bytes.toBytes("my other value"));
	list.addFilter(filter2);
	scan.setFilter(list);

## 列值过滤器

用于比较列的值的过滤器。

### ValueFilter

ValueFilter 对所有列的值进行比较。

包含 ".4" 的所有行。

	Scan scan = new Scan();
	Filter f = new ValueFilter(CompareOp.EQUAL, new SubstringComparator(".4") );
	scan.setFilter(f);

### SingleColumnValueFilter

SingleColumnValueFilter 用于匹配指定列的值。

下面示例检查列值和字符串'my value' 相等

	Scan scan = new Scan();
	SingleColumnValueFilter filter = new SingleColumnValueFilter(Bytes.toBytes("family"), Bytes.toBytes("qualifier"), CompareOp.EQUAL, Bytes.toBytes("my value"));
	scan.setFilter(filter);

### SingleColumnValueExcludeFilter

SingleColumnValueExcludeFilter 与 SingleColumnValueFilter 相反，用法相同。匹配到的，不返回。

### DependentColumnFilter

DependentColumnFilter 稍微绕暂不介绍。

## 键值过滤器

键值区别于列值，列值指的是存储的 Value , 而键值指 family（列族名），qualifier（列名）。

### FamilyFilter

FamilyFilter 用于过滤列族。这个是基本不使用的，在 scan 中直接设置family(一般family都是已知的嘛)会优于使用过滤器做，如下面的例子。

	Scan scan = new Scan();
	scan.addFamily(Bytes.toBytes("family"));

使用 FamilyFilter. 指定比较类型和比较器。

	Scan scan = new Scan();
	Filter f = new FamilyFilter(CompareOp.EQUAL, new BinaryComparator(Bytes.toBytes("family")));
	scan.setFilter(f);

### QualifierFilter

基本同 FamilyFilter，用于过滤列族列，已知列的话，直接 addColumn 是最佳的选择。

	Scan scan = new Scan();
	scan.addColumn(Bytes.toBytes("family"), Bytes.toBytes("qualifier"));

使用 QualifierFilter. 指定比较类型和比较器。

	Scan scan = new Scan();
	Filter f = new QualifierFilter(CompareOp.EQUAL, new BinaryComparator(Bytes.toBytes("qualifier")));
	scan.setFilter(f);

### ColumnPrefixFilter

ColumnPrefixFilter 可基于列名(即Qualifier)前缀过滤。  

找出所有以 "abc" 开始的列。

	Scan scan = new Scan();
	byte[] prefix = Bytes.toBytes("abc");
	Filter f = new ColumnPrefixFilter(prefix);
	scan.setFilter(f);

### MultipleColumnPrefixFilter

看名字就知道可以指定多个前缀啦。

返回 "abc" 或 "xyz" 前缀的列。

	Scan scan = new Scan();
	byte[][] prefixes = new byte[][] {Bytes.toBytes("abc"), Bytes.toBytes("xyz")};
	Filter f = new MultipleColumnPrefixFilter(prefixes);
	scan.setFilter(f);

### ColumnRangeFilter

ColumnRangeFilter 可以进行高效列范围扫描。

你有 a - z 26个列，你想要 b - q 这些列。

	Scan scan = new Scan();
	byte[] startColumn = Bytes.toBytes("b");
	byte[] endColumn = Bytes.toBytes("q");
	Filter f = new ColumnRangeFilter(startColumn, true, endColumn, true);// 俩个true表示包含首尾
	scan.setFilter(f);

如果没有startColumn限制，让 `startColumn = null` 即可。

## RowKey 过滤器

对 RowKey 进行过滤的过滤器。

### RowFilter

通常认为行选择时Scan采用 startRow/stopRow 方法比较好。使用Get获取某一行也是挺好的。

	Scan scan = new Scan();
	scan.setStartRow(Bytes.toBytes("startRow"));
	scan.setStopRow(Bytes.toBytes("stopRow"));
	// startRow = stopRow 时等价于
	Get get = new Get(Bytes.toBytes("startRow"));
	Scan scan = new Scan(get);

使用 RowFilter.指定比较类型和比较器。

	Scan scan = new Scan();
	RowFilter f = new RowFilter(CompareOp.LESS_OR_EQUAL, new BinaryComparator(Bytes.toBytes("q")));// 小于等于q的行
	scan.setFilter(f);

### PrefixFilter

对RowKey前缀进行过滤。

	Scan scan = new Scan();
	Filter f = new PrefixFilter(Bytes.toBytes("log"));// log开头的行被选中
	scan.setFilter(f);

### InclusiveStopFilter

InclusiveStopFilter 设置结束行的过滤器，貌似没这个必要呀，我们已经有 `setStopRow` 了。

	Scan scan = new Scan();
	Filter f = new InclusiveStopFilter(Bytes.toBytes("stopRow"));
	scan.setFilter(f);

## Utility 过滤器

工具性质的过滤器。

### KeyOnlyFilter

KeyOnlyFilter 返回每个键值对的key，value置为空（默认）或为值的长度。

	// 默认 lenAsVal = false
	Filter filter = new KeyOnlyFilter();
	// 设置 lenAsVal = false
	Filter filter = new KeyOnlyFilter(false);
	// 设置 lenAsVal = true,返回的 value = value.length
	Filter filter = new KeyOnlyFilter(true);

	Scan scan = new Scan();
    scan.setFilter(filter);


### FirstKeyOnlyFilter

FirstKeyOnlyFilter 与 KeyOnlyFilter 的区别是只返回一行的第一个Key-Value，value为实际的value。

	Filter filter = new FirstKeyOnlyFilter();

	Scan scan = new Scan();
    scan.setFilter(filter);

### TimestampsFilter

TimestampsFilter 返回值的时间戳是在我们此过滤器设置的列表中的值。

	Scan scan = new Scan();
	List<Long> ts = new ArrayList<Long>(); 
	// 没有我这样的时间戳，时间戳是13位的long型 
    ts.add(new Long(5));
    ts.add(new Long(10));  
    ts.add(new Long(15));
    Filter f = new TimestampsFilter(ts);
	scan.setFilter(f);

### ColumnCountGetFilter

ColumnCountGetFilter 在Scan中无用，是为了测试Get获取的列时临时的。返回一行中的n列。不做探究。

### ColumnPaginationFilter

ColumnPaginationFilter 以 ColumnCountGetFilter 为基础，实现一行的列的分页展示，暂不做探究。

### RandomRowFilter

RandomRowFilter 随机选取若干行返回。

	// chance取值为0到1.0 1返回全部，0.1返回随机的10%
	Filter filter = new RandomRowFilter(0.1);

### SkipFilter

SkipFilter 只作用在值过滤器上，对于行的和键值的过滤不适用，他是一个装饰器，以实际例子来理解它的作用。

每一行有多个列值，对于每一行，有全是value的，有部分是value的，有全不是value的。

没有使用 SkipFilter
	
	// 返回包含value值的列，包括全是value的和部分是value的
	Filter f1 = new ValueFilter(CompareOp.EQUAL, new BinaryComparator(Bytes.toBytes("value")));

使用 SkipFilter

	// 只返回全是value值的列，对于没有value和部分value的跳过
	Filter f1 = new ValueFilter(CompareOp.EQUAL, new BinaryComparator(Bytes.toBytes("value")));
	Filter skipFilter = new SkipFilter(f1);

### WhileMatchFilter

WhileMatchFilter 也是装饰器，当匹配上了继续。

### PageFilter

PageFilter 页过滤器可以根据主键有序返回固定数量的记录，这需要客户端在遍历的时候记住页开始的地方，配合scan的startRow一起使用。

	Scan scan = new Scan();
	scan.setStartRow(Bytes.toBytes("startRow"));
	PageFilter pageFilter =new PageFilter(500);
	scan.setFilter(pageFilter);

startRow是包含在查询中的。所以在处理实际的分页时，需要做必要的调整。

方案一：

1. 获取page1，得到最后一行endRow
2. 使用endRow作为page2的startRow肯定是错误的，按如下处理

		byte[] tmp = new byte[]{ 0x00 };
		byte[] startRow = Bytes.add(endRow, tmp);

	举例说明：  
	endRow = sds  
	startRow = sds0  
	HBase是按字典序排列的，则sds0在sds的后面紧挨着，所以这样查出的结果就是我们想要的了。

方案二：

1. 查询的时候多查一个，即 `pageSize = pageSize + 1;`
2. 展示的时候展示前pageSize个，把最后一个作为下一个page的startRow.

## 参考资料

- [HBase 官方文档0.97](http://abloz.com/hbase/book.html#client.filter.cv.scvf)
- [HBase API](http://hbase.apache.org/apidocs/index.html)
- [hbase权威指南阅读随手笔记二之过滤器](http://blog.csdn.net/saint1126/article/details/8257941)
- [Hbase 学习笔记（二）: 高级模块](http://ygydaiaq-gmail-com.iteye.com/blog/1716844)
- [HBase开发实践](http://blog.csdn.net/kuyuyingzi/article/details/43950373)