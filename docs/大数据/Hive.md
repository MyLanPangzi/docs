# Hive
[TOC]



## 概述

Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张表，并提供类SQL查询功能。<br/>
本质是：将HQL转化成MapReduce程序

![](image\SQL—Mapreduce.png)

1. Hive处理的数据存储在HDFS
2. Hive分析数据底层的实现是MapReduce
3. 执行程序运行在Yarn上

优点：

1. 操作接口采用类SQL语法，提供快速开发的能力（简单、容易上手）。
2. 避免了去写MapReduce，减少开发人员的学习成本。
3. Hive的执行延迟比较高，因此Hive常用于数据分析，对实时性要求不高的场合。
4. Hive优势在于处理大数据，对于处理小数据没有优势，因为Hive的执行延迟比较高。
5. Hive支持用户自定义函数，用户可以根据自己的需求来实现自己的函数。

缺点：
1. Hive的HQL表达能力有限
     1. 迭代式算法无法表达
     2. 数据挖掘方面不擅长
2. Hive的效率比较低
    1. Hive自动生成的MapReduce作业，通常情况下不够智能化
    2. Hive调优比较困难，粒度较粗

## 架构

1. 用户接口：Client
    1. CLI（hive shell）、JDBC/ODBC(java访问hive)、WEBUI（浏览器访问hive）
2. 元数据：Metastore
    1. 元数据包括：表名、表所属的数据库（默认是default）、表的拥有者、列/分区字段、表的类型（是否是外部表）、表的数据所在目录等；<br/>默认存储在自带的derby数据库中，推荐使用MySQL存储Metastore
3. Hadoop 使用HDFS进行存储，使用MapReduce进行计算。
4. 驱动器：Driver
    1. 解析器（SQL Parser）：将SQL字符串转换成抽象语法树AST，这一步一般都用第三方工具库完成，比如antlr；对AST进行语法分析，比如表是否存在、字段是否存在、SQL语义是否有误。
    2. 编译器（Physical Plan）：将AST编译生成逻辑执行计划。
    3. 优化器（Query Optimizer）：对逻辑执行计划进行优化。
    4. 执行器（Execution）：把逻辑执行计划转换成可以运行的物理计划。对于Hive来说，就是MR/Spark。

![](image\Hive架构.png)

## 数据类型

![](image\datatypes.png)

### 类型转换

1. 隐式类型转换规则如下
     1. 任何整数类型都可以隐式地转换为一个范围更广的类型，如TINYINT可以转换成INT，INT可以转换成BIGINT。
     2. 所有整数类型、FLOAT和STRING类型都可以隐式地转换成DOUBLE。
     3. TINYINT、SMALLINT、INT都可以转换为FLOAT。
     4. BOOLEAN类型不可以转换为任何其它的类型。
2. 可以使用CAST操作显示进行数据类型转换
    * 例如CAST('1' AS INT)将把字符串'1' 转换成整数1；如果强制类型转换失败，如执行CAST('X' AS INT)，表达式返回空值 NULL。

## DDL

### 数据库

```hiveql
create database db_hive;
create database if not exists db_hive;
create database db_hive2 location '/db_hive2.db';
show databases;
show databases like 'db_hive*';
desc database db_hive;
desc database extended db_hive;
use db_hive;
-- 数据库的其他元数据信息都是不可更改的，包括数据库名和数据库所在的目录位置。
alter database db_hive set dbproperties('createtime'='20170830');
drop database db_hive2;
drop database if exists db_hive2;
drop database db_hive cascade;
```

### 表

**Hive创建内部表时，会将数据移动到数据仓库指向的路径；若创建外部表，仅记录数据所在的路径，不对数据的位置做任何改变。<br/>**
**在删除表的时候，内部表的元数据和数据会被一起删除，而外部表只删除元数据，不删除数据。**

```hiveql
-- CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name 
-- [(col_name data_type [COMMENT col_comment], ...)] 
-- [COMMENT table_comment] 
-- [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)] 
-- [CLUSTERED BY (col_name, col_name, ...) 
-- [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS] 
-- [ROW FORMAT row_format] 
-- [STORED AS file_format] 
-- [LOCATION hdfs_path]
alter table student2 set tblproperties('EXTERNAL'='TRUE');
alter table student2 set tblproperties('EXTERNAL'='FALSE');
```

#### 分区表

* 分区表实际上就是对应一个HDFS文件系统上的独立的文件夹，该文件夹下是该分区所有的数据文件。<br/>
* Hive中的分区就是分目录，把一个大的数据集根据业务需要分割成小的数据集。<br/>
* 在查询时通过WHERE子句中的表达式选择查询所需要的指定的分区，这样的查询效率会提高很多。<br/>

```hiveql
create table dept_partition(
deptno int, dname string, loc string
)
partitioned by (month string)
row format delimited fields terminated by '\t';
load data local inpath '/opt/module/datas/dept.txt'
 into table default.dept_partition partition(month='201709');
select * from dept_partition where month='201709';
alter table dept_partition add partition(month='201706') ;
alter table dept_partition add partition(month='201705') partition(month='201704');
alter table dept_partition drop partition (month='201704');
alter table dept_partition drop partition (month='201705'), partition (month='201706');
show partitions dept_partition;
```

#### 把数据直接上传到分区目录上，让分区表和数据产生关联的三种方式

```hiveql
-- 方式一：上传数据后修复
msck repair table dept_partition2;
-- 方式二：上传数据后添加分区
alter table dept_partition2 add partition(month='201709', day='11');
-- 方式三：创建文件夹后load数据到分区
load data local inpath '/opt/module/datas/dept.txt' into table
 dept_partition2 partition(month='201709',day='10');
```

## DML

### 导入

* Load
* Insert
* As Select
* Location
* Import

```hiveql
create table student(id string, name string) row format delimited fields terminated by '\t';
load data local inpath '/opt/module/datas/student.txt' into table default.student;
dfs -put /opt/module/datas/student.txt /user/atguigu/hive;
load data inpath '/user/atguigu/hive/student.txt' into table default.student;

insert into table  student partition(month='201709') values(1,'wangwu');
insert overwrite table student partition(month='201708')
             select id, name from student where month='201709';
from student
              insert overwrite table student partition(month='201707')
              select id, name where month='201709'
              insert overwrite table student partition(month='201706')
              select id, name where month='201709';
create table if not exists student3
as select id, name from student;
create table if not exists student5(
              id int, name string
              )
              row format delimited fields terminated by '\t'
              location '/user/hive/warehouse/student5';
 dfs -put /opt/module/datas/student.txt /user/hive/warehouse/student5;
select * from student5;
import table student2 partition(month='201709') from
 '/user/hive/warehouse/export/student';
```

### 导出

* Insert
* hdfs dfs get
* hive -e/-f
* hive -e/-f
* Export
* Sqoop

```hiveql
insert overwrite local directory '/opt/module/datas/export/student'
            select * from student;
insert overwrite local directory '/opt/module/datas/export/student1'
           ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'             select * from student;
insert overwrite directory '/user/atguigu/student2'
             ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' 
             select * from student;
dfs -get /user/hive/warehouse/student/month=201709/000000_0 /opt/module/datas/export/student3.txt;
-- bin/hive -e 'select * from default.student;' > /opt/module/datas/export/student4.txt;
 export table default.student to '/user/hive/warehouse/export/student';
```

## 查询

### 基础查询

省略。。。

### 排序

#### Sort By 

Reducer内排序

#### Distribute By

MR分区，结合sort by使用。

#### Cluster By

当distribute by和sort by字段相同时，可以使用cluster by方式。<br/>
cluster by除了具有distribute by的功能外还兼具sort by的功能。但是排序只能是升序排序，不能指定排序规则为ASC或者DESC。

### 常用函数

* nvl
* CASE  WHEN  THEN  ELSE  END 
* CONCAT CONCAT_WS  
* COLLECT_SET  
* LATERAL VIEW EXPLODE(col)    用于和split, explode等UDTF一起使用，它能够将一列数据拆成多行数据，在此基础上可以对拆分后的数据进行聚合。

### 窗口函数

#### 相关函数说明
* OVER()：指定分析函数工作的数据窗口大小，这个数据窗口大小可能会随着行的变而变化
* CURRENT ROW：当前行
* n PRECEDING：往前n行数据
* n FOLLOWING：往后n行数据
* UNBOUNDED：起点，UNBOUNDED PRECEDING 表示从前面的起点， UNBOUNDED FOLLOWING表示到后面的终点
* LAG(col,n)：往前第n行数据
* LEAD(col,n)：往后第n行数据
* NTILE(n)：把有序分区中的行分发到指定数据的组中，各个组有编号，编号从1开始，对于每一行，NTILE返回此行所属的组的编号。注意：n必须为int类型。

### Rank

* RANK() 排序相同时会重复，总数不会变
* DENSE_RANK() 排序相同时会重复，总数会减少
* ROW_NUMBER() 会根据顺序计算

## 函数

1. 系统函数
2. 自定义函数
    1. UDF
    1. UDTF
    1. UDAF

## 压缩

#### 开启Map输出阶段压缩

```hiveql
set hive.exec.compress.intermediate=true;
set mapreduce.map.output.compress=true;
set mapreduce.map.output.compress.codec=org.apache.hadoop.io.compress.SnappyCodec; 
```
#### 开启Reduce输出阶段压缩

```hiveql
set hive.exec.compress.output=true;;
set mapreduce.output.fileoutputformat.compress=true;
set mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.SnappyCodec;
set mapreduce.output.fileoutputformat.compress.type=BLOCK;
```

#### 文件存储格式

Hive支持的存储数据的格式主要有：TEXTFILE 、SEQUENCEFILE、ORC、PARQUET。

##### 列式存储和行式存储

1. 行存储的特点
    * 查询满足条件的一整行数据的时候，列存储则需要去每个聚集的字段找到对应的每个列的值，行存储只需要找到其中一个值，其余的值都在相邻地方，所以此时行存储查询的速度更快。
2. 列存储的特点
    * 因为每个字段的数据聚集存储，在查询只需要少数几个字段的时候，能大大减少读取的数据量；每个字段的数据类型一定是相同的，列式存储可以针对性的设计更好的设计压缩算法。
3. TEXTFILE和SEQUENCEFILE的存储格式都是基于行存储的；
4. ORC和PARQUET是基于列式存储的。

##### TextFile格式

默认格式，数据不做压缩，磁盘开销大，数据解析开销大。可结合Gzip、Bzip2使用，但使用Gzip这种方式，hive不会对数据进行切分，从而无法对数据进行并行操作。

##### Orc格式

Orc (Optimized Row Columnar)是Hive 0.11版里引入的新的存储格式。<br/>
如图所示可以看到每个Orc文件由1个或多个stripe组成，每个stripe250MB大小，这个Stripe实际相当于RowGroup概念，不过大小由4MB->250MB，这样应该能提升顺序读的吞吐率。<br/>
每个Stripe里有三部分组成，分别是Index Data，Row Data，Stripe Footer：

1. Index Data：一个轻量级的index，默认是每隔1W行做一个索引。这里做的索引应该只是记录某行的各字段在Row Data中的offset。
2. Row Data：存的是具体的数据，先取部分行，然后对这些行按列进行存储。对每个列进行了编码，分成多个Stream来存储。
3. Stripe Footer：存的是各个Stream的类型，长度等信息。

每个文件有一个File Footer，这里面存的是每个Stripe的行数，每个Column的数据类型信息等；<br/>
每个文件的尾部是一个PostScript，这里面记录了整个文件的压缩类型以及FileFooter的长度信息等。<br/>
在读取文件时，会seek到文件尾部读PostScript，从里面解析到File Footer长度，再读FileFooter，从里面解析到各个Stripe信息，再读各个Stripe，即从后往前读。

![img](image\orc.jpg)

##### Parquet格式

Parquet是面向分析型业务的列式存储格式，由Twitter和Cloudera合作开发，
2015年5月从Apache的孵化器里毕业成为Apache顶级项目。<br/>
Parquet文件是以二进制方式存储的，所以是不可以直接读取的，文件中包括该文件的数据和元数据，因此Parquet格式文件是自解析的。<br/>
通常情况下，在存储Parquet数据的时候会按照Block大小设置行组的大小，
由于一般情况下每一个Mapper任务处理数据的最小单位是一个Block，
这样可以把每一个行组由一个Mapper任务处理，增大任务执行并行度。 <br/>

![img](image\parquet.jpg)

上图展示了一个Parquet文件的内容，一个文件中可以存储多个行组，
文件的首位都是该文件的Magic Code，
用于校验它是否是一个Parquet文件，
Footer length记录了文件元数据的大小，
通过该值和文件长度可以计算出元数据的偏移量，
文件的元数据中包括每一个行组的元数据信息和该文件存储数据的Schema信息。
除了文件中每一个行组的元数据，每一页的开始都会存储该页的元数据，
在Parquet中，有三种类型的页：数据页、字典页和索引页。
数据页用于存储当前行组中该列的值，字典页存储该列值的编码字典，
每一个列块中最多包含一个字典页，索引页用来存储当前行组下该列的索引，目前Parquet中还不支持索引页。

**在实际的项目开发当中，hive表的数据存储格式一般选择：orc或parquet。压缩方式一般选择snappy，lzo。**

## 企业级调优

### Fetch抓取

Fetch抓取是指，Hive中对某些情况的查询可以不必使用MapReduce计算。
例如：SELECT * FROM employees;在这种情况下，Hive可以简单地读取employee对应的存储目录下的文件，
然后输出查询结果到控制台。<br/>
在hive-default.xml.template文件中hive.fetch.task.conversion默认是more，
老版本hive默认是minimal，该属性修改为more以后，在全局查找、字段查找、limit查找等都不走mapreduce。

### 本地模式

大多数的Hadoop Job是需要Hadoop提供的完整的可扩展性来处理大数据集的。<br/>
不过，有时Hive的输入数据量是非常小的。
在这种情况下，为查询触发执行任务消耗的时间可能会比实际job的执行时间要多的多。<br/>
对于大多数这种情况，Hive可以通过本地模式在单台机器上处理所有的任务。对于小数据集，执行时间可以明显被缩短。。<br/>
用户可以通过设置hive.exec.mode.local.auto的值为true，来让Hive在适当的时候自动启动这个优化。。<br/>

```hiveql
set hive.exec.mode.local.auto=true;  //开启本地mr
//设置local mr的最大输入数据量，当输入数据量小于这个值时采用local  mr的方式，默认为134217728，即128M
set hive.exec.mode.local.auto.inputbytes.max=50000000;
//设置local mr的最大输入文件个数，当输入文件个数小于这个值时采用local mr的方式，默认为4
set hive.exec.mode.local.auto.input.files.max=10;
```

### 表的优化

#### 大表Join大表

1. 空KEY过滤
2. 空key转换

####  MapJoin

如果不指定MapJoin或者不符合MapJoin的条件，那么Hive解析器会将Join操作转换成Common Join，
即：在Reduce阶段完成join。
容易发生数据倾斜。可以用MapJoin把小表全部加载到内存在map端进行join，避免reducer处理。

```hiveql
set hive.auto.convert.join = true; -- 默认为true
set hive.mapjoin.smalltable.filesize=25000000;
```

#### Group By

**采用GROUP by去重id**<br/>
默认情况下，Map阶段同一Key数据分发给一个reduce，当一个key数据过大时就倾斜了。
并不是所有的聚合操作都需要在Reduce端完成，很多聚合操作都可以先在Map端进行部分聚合，最后在Reduce端得出最终结果。

```hiveql
-- 是否在Map端进行聚合，默认为True
set hive.map.aggr = true;
-- 在Map端进行聚合操作的条目数目
set hive.groupby.mapaggr.checkinterval = 100000;
-- 有数据倾斜的时候进行负载均衡（默认是false）
set hive.groupby.skewindata = true;
```

#### 行列过滤

列处理：在SELECT中，只拿需要的列，如果有，尽量使用分区过滤，少用SELECT *。<br/>
行处理：在分区剪裁中，当使用外关联时，如果将副表的过滤条件写在Where后面，那么就会先全表关联，之后再过滤 

#### 笛卡尔积

#### 动态分区调整

关系型数据库中，对分区表Insert数据时候，数据库自动会根据分区字段的值，
将数据插入到相应的分区中，Hive中也提供了类似的机制，即动态分区(Dynamic Partition)，
只不过，使用Hive的动态分区，需要进行相应的配置。

```hiveql
set hive.exec.dynamic.partition.mode=nonstrict;
set hive.exec.max.dynamic.partitions.pernode=100;
set hive.exec.max.created.files=100000;
set hive.error.on.empty.partition=false;
```

### 数据倾斜

1. 合理设置Map数
2. 小文件进行合并
3. 复杂文件增加Map数
4. 合理设置Reduce数
    1. 处理大数据量利用合适的reduce数；使单个reduce任务处理数据量大小要合适；

```hiveql
set mapreduce.input.fileinputformat.split.maxsize=100;
set hive.exec.reducers.bytes.per.reducer=256000000;
set hive.exec.reducers.max=1009;
set mapreduce.job.reduces = 15;
```

### 并行执行

Hive会将一个查询转化成一个或者多个阶段。<br/>
这样的阶段可以是MapReduce阶段、抽样阶段、合并阶段、limit阶段。或者Hive执行过程中可能需要的其他阶段。
默认情况下，Hive一次只会执行一个阶段。<br/>
不过，某个特定的job可能包含众多的阶段，而这些阶段可能并非完全互相依赖的，也就是说有些阶段是可以并行执行的，
这样可能使得整个job的执行时间缩短。<br/>
如果有更多的阶段可以并行执行，那么job可能就越快完成。
通过设置参数hive.exec.parallel值为true，就可以开启并发执行。不过，在共享集群中，需要注意下，
如果job中并行阶段增多，那么集群利用率就会增加。<br/>
当然，得是在系统资源比较空闲的时候才有优势，否则，没资源，并行也起不来。

```hiveql
set hive.exec.parallel=true;              //打开任务并行执行
set hive.exec.parallel.thread.number=16;  //同一个sql允许最大并行度，默认为8。
```

### 严格模式    

Hive提供了一个严格模式，可以防止用户执行那些可能意想不到的不好的影响的查询。
通过设置属性hive.mapred.mode值为默认是非严格模式nonstrict 。
开启严格模式需要修改hive.mapred.mode值为strict，开启严格模式可以禁止3种类型的查询。

```xml
<property>
    <name>hive.mapred.mode</name>
    <value>strict</value>
    <description>
      The mode in which the Hive operations are being performed. 
      In strict mode, some risky queries are not allowed to run. They include:
        Cartesian Product.
        No partition being picked up for a query.
        Comparing bigints and strings.
        Comparing bigints and doubles.
        Orderby without limit.
    </description>
</property>
```

1. 对于分区表，除非where语句中含有分区字段过滤条件来限制范围，否则不允许执行。
2. 对于使用了order by语句的查询，要求必须使用limit语句。
3. 限制笛卡尔积的查询。

### JVM重用

1. 小文件的场景或task特别多的场景，这类场景大多数执行时间都很短。
2. JVM重用可以使得JVM实例在同一个job中重新使用N次。

### 推测执行
### Explain

## Idea连接Hive

1. core-site.xml

   ```xml
       <property>
           <!-- atguigu 替换成登录用户名(root，其他用户）-->
           <name>hadoop.proxyuser.atguigu.hosts</name>
           <value>*</value>
       </property>
       <property>
           <name>hadoop.proxyuser.atguigu.groups</name>
           <value>*</value>
       </property>
   
   ```

2. driver![image-20200324100935485](image\idea-hive-driver.png)

3. idea![image-20200324101525433](image\idea-hive.png)