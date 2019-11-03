---
title: CDH Hue入门
date: 2019-04-01 22:14:33
categories: BigData
tags:
 - HUE
 - cdh
---


[翻译自](https://www.cloudera.com/documentation/enterprise/5-13-x/topics/quickstart_vm_administrative_information.html)
# 欢迎及介绍
1. cdh全称是cloudera open source distribution including apache hadoop的全称。
2. hue登录用户名：cloudera 密码：cloudera
3. 怎么用cdh
    - 如何进行简单的数据挖掘和分析
    - 让你老板给你涨工资~
    - 某些点会用到cloudera manager，可能）导致有些功能无法正常运行。有些部件也会用到商业版本的许可才能正常使用。
4. 避免以上问题，
    1. 可以用express 版本（最少需要8G内存和2核心cpu）
    2. 用企业版的试用版，试用版有60天的体验期。（最少需要10G内存和2核心cpu）

# 入门 提取查询关系数据

之后的教程中，我们将通过呈现一个关于DataCo公司的案例。我们的任务就是帮助这公司深入了解并解决一些问题。

- 剧情1
```
王老板：吐沫星子漫天飞的谈谈大数据。。
小明：hadoop吧那就。
```
 <!-- more -->
- 剧情2
```
DataCo公司现在难题是：哪种产品消费者最喜欢买。当然一般想到的是查看一下关系数据库中的交易数据表，排序一下就知道了，有这么简单?
但是更有效，更深入分析，且适合更大规模的，就要用到cdh平台（hadoop技术栈了）
下面这个例子，我们用cdh做，让你感觉常用的关系型数据库那种方法没啥两样。让你用同等的时间下，还能出更多的BI类分析和其他报表，
```

首先我们需要一个工具（sqoop）把常用的RDBMS关系型数据库中的结构字段扔到HDFS中（当然是肯定保持同样的数据结构。）这样就类似一个从库，在hdfs上查询不会占用其他的查询压力。

我们用一个优化的文件格式化工具`avro`，或者用`empala`做到上面这些工作。

    sqoop import-all-tables \
        -m 1 \
        --connect jdbc:mysql://quickstart:3306/retail_db \
        --username=retail_dba \
        --password=cloudera \
        --compression-codec=snappy \
        --as-parquetfile \
        --warehouse-dir=/user/hive/warehouse \
        --hive-import
    
    
注意默认都会到default库，如果需要到指定库，需要增加`--hive-database=yourdbname \`

上面的`sqoop`命令做了很多工作，通过`mapreduce`任务，拉取mysql数据写入到`hdfs`（应该是用apache 的parquet列存储格式存储，该列式存储支持`hive` `impala` `pig`等多种查询引擎，而且适配多个计算框架，如mapreduce，`spark`等）。最终以指定（默认）表的方式体现对应mysql中的schema。

`parquet`是用来再`hadoop`平台相关的统一的数据格式。与传统的行模式不同，他是以列存储。主要是为了分析一些特殊指定的数据，可以通过变量来分析关系数据。parquet能更优的存储与检索。

现在我们直观的看一下刚刚插入的hive的具体目录

    [cloudera@quickstart ~]$ hadoop fs -ls /user/hive/warehouse
    Found 7 items
    drwxrwxrwx   - cloudera supergroup          0 2018-04-15 23:36 /user/hive/warehouse/categories
    drwxrwxrwx   - cloudera supergroup          0 2018-04-15 23:38 /user/hive/warehouse/customers
    drwxrwxrwx   - cloudera supergroup          0 2018-04-15 23:39 /user/hive/warehouse/departments
    drwxrwxrwx   - cloudera supergroup          0 2018-04-15 23:40 /user/hive/warehouse/order_items
    drwxrwxrwx   - cloudera supergroup          0 2018-04-15 23:41 /user/hive/warehouse/orders
    drwxrwxrwx   - cloudera supergroup          0 2018-04-15 23:42 /user/hive/warehouse/products
    drwxrwxrwx   - cloudera supergroup          0 2018-04-13 01:41 /user/hive/warehouse/xin.db

通过 `hadoop fs -ls` 可以到指定标识为d的目录内继续查看，比如

    [cloudera@quickstart ~]$ hadoop fs -ls /user/hive/warehouse/categories
    Found 3 items
    drwxr-xr-x   - cloudera supergroup          0 2018-04-15 23:28 /user/hive/warehouse/categories/.metadata
    drwxr-xr-x   - cloudera supergroup          0 2018-04-15 23:36 /user/hive/warehouse/categories/.signals
    -rw-r--r--   1 cloudera supergroup       1957 2018-04-15 23:36 /user/hive/warehouse/categories/3e30822b-f7e7-4a0c-bde3-e61f3e373a11.parquet

注意：parquet的文件数量指的是sqoop运行时，mappe任务的数量。因为我的是单节点，所以就是一个。
我们追进来看一下元数据

    [cloudera@quickstart ~]$ hadoop fs -ls /user/hive/warehouse/categories/.metadata
    Found 1 items
    drwxr-xr-x   - cloudera supergroup          0 2018-04-15 23:28 /user/hive/warehouse/categories/.metadata/schemas
    [cloudera@quickstart ~]$ hadoop fs -ls /user/hive/warehouse/categories/.metadata/schemas
    Found 1 items
    -rw-r--r--   1 cloudera supergroup        594 2018-04-15 23:28 /user/hive/warehouse/categories/.metadata/schemas/1.avsc
    [cloudera@quickstart ~]$ hadoop fs -cat  /user/hive/warehouse/categories/.metadata/schemas/1.avsc
    {
      "type" : "record",
      "name" : "categories",
      "doc" : "Sqoop import of categories",
      "fields" : [ {
        "name" : "category_id",
        "type" : [ "null", "int" ],
        "default" : null,
        "columnName" : "category_id",
        "sqlType" : "4"
      }, {
        "name" : "category_department_id",
        "type" : [ "null", "int" ],
        "default" : null,
        "columnName" : "category_department_id",
        "sqlType" : "4"
      }, {
        "name" : "category_name",
        "type" : [ "null", "string" ],
        "default" : null,
        "columnName" : "category_name",
        "sqlType" : "12"
      } ],
      "tableName" : "categories"

当然我们在`hue`中用 `show create table categories`来查看，会看到和上面对应的信息

    Show CREATE TABLE categories
    
    1	CREATE TABLE `categories`(
    2	  `category_id` int, 
    3	  `category_department_id` int, 
    4	  `category_name` string)
    5	ROW FORMAT SERDE 
    6	  'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe' 
    7	STORED AS INPUTFORMAT 
    8	  'org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat' 
    9	OUTPUTFORMAT 
    10	  'org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat'
    11	LOCATION
    12	  'hdfs://quickstart.cloudera:8020/user/hive/warehouse/categories'
    13	TBLPROPERTIES (
    14	  'COLUMN_STATS_ACCURATE'='false', 
    15	  'avro.schema.url'='hdfs://quickstart.cloudera:8020/user/hive/warehouse/categories/.metadata/schemas/1.avsc', 
    16	  'kite.compression.type'='snappy', 
    17	  'numFiles'='0', 
    18	  'numRows'='-1', 
    19	  'rawDataSize'='-1', 
    20	  'totalSize'='0', 

另外我们在hive命令行中可以看到其他关于表的formated信息

    hive> describe formatted customers;
    OK
    # col_name            	data_type           	comment             
    	 	 
    customer_id         	int                 	                    
    customer_fname      	string              	                    
    customer_lname      	string              	                    
    customer_email      	string              	                    
    customer_password   	string              	                    
    customer_street     	string              	                    
    customer_city       	string              	                    
    customer_state      	string              	                    
    customer_zipcode    	string              	                    
    	 	 
    # Detailed Table Information	 	 
    Database:           	default             	 
    Owner:              	null                	 
    CreateTime:         	Sun Apr 15 23:36:57 PDT 2018	 
    LastAccessTime:     	UNKNOWN             	 
    Protect Mode:       	None                	 
    Retention:          	0                   	 
    Location:           	hdfs://quickstart.cloudera:8020/user/hive/warehouse/customers	 
    Table Type:         	MANAGED_TABLE       	 
    Table Parameters:	 	 
    	COLUMN_STATS_ACCURATE	false               
    	avro.schema.url     	hdfs://quickstart.cloudera:8020/user/hive/warehouse/customers/.metadata/schemas/1.avsc
    	kite.compression.type	snappy              
    	numFiles            	0                   
    	numRows             	-1                  
    	rawDataSize         	-1                  
    	totalSize           	0                   
    	transient_lastDdlTime	1523860617          
    	 	 
    Storage Information	 	 
    SerDe Library:      	org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe	 
    InputFormat:        	org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat	 
    OutputFormat:       	org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat	 
    Compressed:         	No                  	 
    Num Buckets:        	-1                  	 
    Bucket Columns:     	[]                  	 
    Sort Columns:       	[]                  	 
    Time taken: 0.069 seconds, Fetched: 39 row(s)


当然创建表也可以利用外部已经存在的文件导入（`CREATE EXTERNAL TABLE`）的方式。以外部表导入的方式不会在hive的仓库中查看到（用hive或者impala都能实现），

比如下面的例子,我们在hive中执行以下操作

    CREATE EXTERNAL TABLE tb_test01(id INT,category_id INT, name string,price INT)
    ROW FORMAT DELIMITED
    FIELDS TERMINATED BY ','
    STORED AS TEXTFILE
    location '/user/hive/external/tb_external_table01';

`LOCATION`指的是warehouse的存放路径，不指定就到hive.metastore.warehouse.dir指定的路径下
(一般演示我习惯用TERMINATED ,分割)
load 数据到刚刚创建的tb_test01表中，准备数据如下
    
    [cloudera@quickstart tongbo]$ cat hadoop_external_test.txt 
    1,24551,Cleats,17
    2,22246,Men's Footwear,18
    3,21035,Women's Apparel,24
    4,19298,Indoor/Outdoor Games,46
    5,17325,Fishing,45
    6,15540,Water Sports,48
    7,13729,Camping & Hiking,43
    8,12487,Cardio Equipment,9
    9,10984,Shop By Sport,29
    10,2029,Electronics


`LOAD data local inpath '/home/tongbo/hadoop_external_test.txt' into table tb_test01;`

这样就实现了数据的导入。内部表和外部表的区别，简单概况如下：
Hive 创建内部表时，会将数据移动到数据仓库指向的路径（配置文件中配置）；
若创建外部表，仅记录数据所在的路径，不对数据的位置做任何改变。
在删除表的时候，内部表的元数据和数 据会被一起删除，而外部表只删除元数据，不删除数据。

下面主要说明如何用cdh中的hive（impala） web页面来进行查询操作。默认为8888端口，用户名和密码都是cloudera，mysql用户名为root，密码为cloudera

在hive创建及查询等更新过程中，impala不会自动拉取跟新的元数据（metadata）的改变
，所以第一件事情是更新metadata的时效，这样我们可以看到所有目前的表

    invalidate metadata;
    show tables;

当然可以通过内置的hdfs看到实际存储的文件记录。
现在关系型数据库中的数据已经到了hdfs里面，回过头来看dataco公司的问题。
下面的mysql展示了每个商品的总利润，并且取前十条

        SELECT
        	count(order_item_quantity) AS count,
        	c.category_name,
        	c.category_id
        FROM
        	order_items AS oi
        INNER JOIN products AS p ON p.product_id = oi.order_item_product_id
        INNER JOIN categories AS c ON p.product_category_id = c.category_id
        GROUP BY
        	c.category_name
        ORDER BY
        	count DESC
        LIMIT 10;
        +-------+----------------------+-------------+
        | count | category_name        | category_id |
        +-------+----------------------+-------------+
        | 24551 | Cleats               |          17 |
        | 22246 | Men's Footwear       |          18 |
        | 21035 | Women's Apparel      |          24 |
        | 19298 | Indoor/Outdoor Games |          46 |
        | 17325 | Fishing              |          45 |
        | 15540 | Water Sports         |          48 |
        | 13729 | Camping & Hiking     |          43 |
        | 12487 | Cardio Equipment     |           9 |
        | 10984 | Shop By Sport        |          29 |
        |  3156 | Electronics          |          13 |
        +-------+----------------------+-------------+
        10 rows in set (0.28 sec)
        
        mysql> 


下面是hive语法

        select count(order_item_quantity) as count ,c.category_name,c.category_id from order_items as oi inner join products as p on p.product_id=oi.order_item_product_id
        inner join categories as c on p.product_category_id=c.category_id group by c.category_name,c.category_id
         order by count desc limit 10


注意，hive语法中select后面不能有非聚合列，如果必须要有，需要在group by 上加上你要聚合的字段。在上述hive语法中就是加上 `group by c.category_name,c.category_id`

        
        1	24551	Cleats	17
        2	22246	Men's Footwear	18
        3	21035	Women's Apparel	24
        4	19298	Indoor/Outdoor Games	46
        5	17325	Fishing	45
        6	15540	Water Sports	48
        7	13729	Camping & Hiking	43
        8	12487	Cardio Equipment	9
        9	10984	Shop By Sport	29
        10	2029	Electronics


再看下面一个复杂的sql

        SELECT
        	p.product_id,
        	p.product_name,
        	r.revenue
        FROM
        	products AS p
        INNER JOIN (
        	SELECT
        		oi.order_item_product_id,
        		sum(
        			cast(
        				oi.order_item_subtotal AS FLOAT
        			)
        		) AS revenue
        	FROM
        		order_items oi
        	INNER JOIN orders AS o ON oi.order_item_order_id = o.order_id
        	WHERE
        		o.order_status <> 'CANCELED'
        	AND o.order_status <> 'SUSPECTED_FARUD'
        	GROUP BY
        		order_item_product_id
        ) AS r ON p.product_id = r.order_item_product_id
        ORDER BY
        	r.revenue DESC
        LIMIT 10

(备注：SUSPECTED_FARUD 涉嫌欺诈)

结果如下：（记住这个结果，下面会用到）
        
        	p.product_id 	p.product_name 				r.revenue
        1	1004	Field & Stream Sportsman 16 Gun Fire Safe	6795260.4066467285
        2	365	Perfect Fitness Perfect Rip Deck	4335357.441116333
        3	957	Diamondback Women's Serene Classic Comfort Bi	4038330.9078979492
        4	191	Nike Men's Free 5.0+ Running Shoe	3586941.2666854858
        5	502	Nike Men's Dri-FIT Victory Golf Polo	3082050
        6	1073	Pelican Sunstream 100 Kayak	3033648.3933258057
        7	403	Nike Men's CJ Elite 2 TD Football Cleat	2831052.3296356201
        8	1014	O'Brien Men's Neoprene Life Vest	2830867.1741104126
        9	627	Under Armour Girls' Toddler Spine Surge Runni	1242929.2107200623
        10	565	adidas Youth Germany Black/Red Away Match Soc	65940


我用`impala`和`hive`分别执行上述语句。发现impala比hive快15倍左右。同时证明了我们用sqoop导入的数据结构（这里指metadata），适用于`hive`和`impala`两种引擎。
hive非常的灵活，是把sql的查询语法转换成mapreduce任务。而impala更适合交互接口分析，我们下面会再次hive在etl中的使用。


总结一下，我们完成了用sqoop把数据导入到hdfs中，然后把他转换为格式化为avro行式存储。（可以在深入了解avro和parquet的区别）
经过以上过程，已经可以用hive或者impala查询数据。我们要更多了了解的是hadoop与传统架构相比，有更多的扩展和灵活性。

- 剧情三
    ```
    领导：（无所谓）的说，你只是展示了你的数据，而且你这些数据我也知道。并没有什么卵用（额外的价值）
    你：也是一脸淡定的无所谓，然后撸起袖子准备干一下。。
    ```

练习2
把结构化数据和非结构化数据结合起来
作为基础运营，你现在有一点疑问：网站内浏览最多的商品就是卖的最多的吗？如果不是，导致原因是什么?
hadoop可以存储结构化和半结构化的数据，而不必向关系型数据库那样，增加一个字段将同步所有的数据列。尤其是适用于web log日志这样的文件形式。我们查看一下最原始的访问站点的日志
为了演示方便，我们批量导入180000条数据的access log。
先在目录下创建一个目录，然后通过hadoop mv命令复制到warehouse下
先看一下当前目录

        cloudera@quickstart ~]$ hadoop fs -ls  /user/hive/warehouse
        Found 7 items
        drwxrwxrwx   - cloudera supergroup          0 2018-04-15 23:36 /user/hive/warehouse/categories
        drwxrwxrwx   - cloudera supergroup          0 2018-04-15 23:38 /user/hive/warehouse/customers
        drwxrwxrwx   - cloudera supergroup          0 2018-04-15 23:39 /user/hive/warehouse/departments
        drwxrwxrwx   - cloudera supergroup          0 2018-04-15 23:40 /user/hive/warehouse/order_items
        drwxrwxrwx   - cloudera supergroup          0 2018-04-15 23:41 /user/hive/warehouse/orders
        drwxrwxrwx   - cloudera supergroup          0 2018-04-15 23:42 /user/hive/warehouse/products
        drwxrwxrwx   - cloudera supergroup          0 2018-04-13 01:41 /user/hive/warehouse/xin.db
        [cloudera@quickstart ~]$ 
        
        [cloudera@quickstart ~]$ sudo -u hdfs hadoop fs -mkdir /user/hive/warehouse/origin_access_logs

看一下准备好的日志文件：

        [cloudera@quickstart ~]$ cd /opt/examples/log_files/
        [cloudera@quickstart log_files]$ ls
        access.log.2
        [cloudera@quickstart log_files]$ du -f access.log.2 
        du: invalid option -- 'f'
        Try `du --help' for more information.
        [cloudera@quickstart log_files]$ du -h access.log.2 
        38M	access.log.2
        [cloudera@quickstart log_files]$ 

执行复制

        [cloudera@quickstart log_files]$ sudo -u hdfs hadoop fs -copyFromLocal /opt/examples/log_files/access.log.2  /user/hive/warehouse/origin_access_logs
        
        由于两次拼写错误，把之前的删除。。
        [cloudera@quickstart log_files]$ hadoop fs -rm -f /user/hive/warehouse/origin__access_logs
        Deleted /user/hive/warehouse/origin__access_logs
        [cloudera@quickstart log_files]$ hadoop fs -rm -f /user/hive/warehouse/original_access_logs
        Deleted /user/hive/warehouse/original_access_logs
        [cloudera@quickstart log_files]$ 

验证一下上面的操作

        [cloudera@quickstart ~]$ hadoop fs -ls /user/hive/warehouse/origin_access_logs
        Found 1 items
        -rw-r--r--   1 hdfs supergroup   39593868 2018-04-17 23:41 /user/hive/warehouse/origin_access_logs/access.log.2
        [cloudera@quickstart ~]$ 

现在我们可以创建一个表，然后用hive或者更腻害的impala来查询。我们需要以下两步：
1. 利用hive强大灵活的serdes ，解析日志，到自定义的hive表中的各个字段中。（通过（反）序列化到自定义的文件字段中）
2. 转移数据到中间表，以便不需要再次（反）序列化
数据放入到表中之后，就可以通过cli或者hue查询啦。
下面用`hue`创建表，并且导入。
先贴一下单一一行格式，参考regex的写法

        144.72.77.159 - - [14/Jun/2014:17:16:22 -0400] "GET /department/fan%20shop/category/fishing/product/Field%20&%20Stream%20Sportsman%2016%20Gun%20Fire%20Safe HTTP/1.1" 200 1206 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:30.0) Gecko/20100101 Firefox/30.0"
        
        
        CREATE external TABLE intermediate_access_logs (
        	ip string,
        	date string,
        	method string,
        	url string,
        	http_version string,
        	code1 string,
        	code2 string,
        	dash string,
        	user_agent string
        ) ROW format serde 'org.apache.hadoop.hive.contrib.serde2.RegexSerDe' WITH serdeproperties (
        	'input.regex' = '([^ ]*) - - \\[([^\\]]*)\\] "([^\ ]*) ([^\ ]*) ([^\ ]*)" (\\d*) (\\d*) "([^"]*)" "([^"]*)"',
        	'output.format.string' = "%1$$s %2$$s %3$$s %4$$s %5$$s %6$$s %7$$s %8$$s %9$$s"
        ) LOCATION '/user/hive/warehouse/origin_access_logs'

创建完之后可以用上面讲到的命令在hive命令行执行。
 `describe formatted intermediate_access_logs;`

`serde`这个关键词（应该是序列化，或者格式化），一般这样使用：
用户在建表的时候可以自定义 SerDe 或者使用自带的 SerDe。
如果没有指定 ROW FORMAT 或者 ROW FORMAT DELIMITED，将会使用自带的 SerDe。
在建表的时候，用户还需要为表指定列，用户在指定表的列的同时也会指定自定义的 SerDe，
Hive 通过 SerDe 确定表的具体的列的数据

再说`location`这个关键词
`EXTERNAL` 关键字可以让用户创建一个外部表，在建表的同时指定一个指向实际数据的路径（LOCATION）
上面那句是) `LOCATION '/user/hive/warehouse/origin_access_logs'`

另外插一句，上面说的内部表和外部表问题。
首先建立一个演示外部表的目录（新建一个）
`hadoop fs -mkdir -p /user/hive_external_table/`
然后把原始日志放入到这个目录
`sudo -u hdfs hadoop fs -copyFromLocal /opt/examples/log_files/access.log.2  /user/hive_external_table/`
验证以上的结果

        [cloudera@quickstart ~]$ hadoop fs -ls  /user/hive_external_table/
        Found 1 items
        -rw-r--r--   1 hdfs supergroup   39593868 2018-04-18 04:14 /user/hive_external_table/access.log.2
        [cloudera@quickstart ~]$ 
        
        
        drop table intermediate_access_logs
        
        CREATE external TABLE intermediate_access_logs (
        	ip string,
        	date string,
        	method string,
        	url string,
        	http_version string,
        	code1 string,
        	code2 string,
        	dash string,
        	user_agent string
        ) ROW format serde 'org.apache.hadoop.hive.contrib.serde2.RegexSerDe' WITH serdeproperties (
        	'input.regex' = '([^ ]*) - - \\[([^\\]]*)\\] "([^\ ]*) ([^\ ]*) ([^\ ]*)" (\\d*) (\\d*) "([^"]*)" "([^"]*)"',
        	'output.format.string' = "%1$$s %2$$s %3$$s %4$$s %5$$s %6$$s %7$$s %8$$s %9$$s"
        ) LOCATION ' /user/hive_external_table/'


如果删除外部表，目录和文件都不会被删除，即使是指定和默认目录一样（比如创建外部表的时候指定 location 为`/user/hive/warehouse/products`）
如果是删除内部表，目录和文件都会被删除。即使是指定的为非默认目录，同样都会被删除（比如创建内部表时指定 location 为 `/user/hive_external_table`）
指定了目录之后，如果目录下有文件，将会自动加载所有

    CREATE EXTERNAL TABLE tokenized_access_logs (
        ip STRING,
        date STRING,
        method STRING,
        url STRING,
        http_version STRING,
        code1 STRING,
        code2 STRING,
        dash STRING,
        user_agent STRING)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
    LOCATION '/user/hive/warehouse/tokenized_access_logs';


跳出来，我们继续上面练习。

    ADD JAR /usr/lib/hive/lib/hive-contrib.jar;
    INSERT OVERWRITE TABLE tokenized_access_logs SELECT * FROM intermediate_access_logs;

最后的查询会调用mapreduce任务（和sqoop一样），可以并行的将数据转移到tokenized_access_logs表中。上面提到过，对于新加的表，我们如果用impala的话，必须重新获取。

    invalidate metadata;
    show  tables

我们可以看到刚刚创建的两个外部表。

    1	categories
    2	customers
    3	departments
    4	intermediate_access_logs
    5	order_items
    6	orders
    7	products
    8	tb_test01
    9	tokenized_access_logs


还是为了验证一下：查询url中包括product的url的总量，按照倒序排

```
select count(*) as nums,url from tokenized_access_logs where url like '%\/product\/%' group by url order by nums desc
```

摘抄结果如下：（这里面要是有商品id可能会与下面的对比更明显些，nginx可以用cookie实现）

        nums    url
     1    1926    /department/apparel/category/cleats/product/Perfect%20Fitness%20Perfect%20Rip%20Deck
     2    1793    /department/apparel/category/featured%20shops/product/adidas%20Kids'%20RG%20III%20Mid%20Football%20Cleat
     3    1780    /department/golf/category/women's%20apparel/product/Nike%20Men's%20Dri-FIT%20Victory%20Golf%20Polo
     4    1757    /department/apparel/category/men's%20footwear/product/Nike%20Men's%20CJ%20Elite%202%20TD%20Football%20Cleat
     5    1104    /department/fan%20shop/category/water%20sports/product/Pelican%20Sunstream%20100%20Kayak
     6    1084    /department/fan%20shop/category/indoor/outdoor%20games/product/O'Brien%20Men's%20Neoprene%20Life%20Vest
     7    1059    /department/fan%20shop/category/camping%20&%20hiking/product/Diamondback%20Women's%20Serene%20Classic%20Comfort%20Bi
     8    1028    /department/fan%20shop/category/fishing/product/Field%20&%20Stream%20Sportsman%2016%20Gun%20Fire%20Safe
     9    1004    /department/footwear/category/cardio%20equipment/product/Nike%20Men's%20Free%205.0+%20Running%20Shoe
     10    939    /department/footwear/category/fitness%20accessories/product/Under%20Armour%20Hustle%20Storm%20Medium%20Duffle%20Bag`


对数据很敏感的人会联想到上面我们的一个结果，是统计商品id，商品名字，和贡献收入的，再贴一下

    1	1004	Field & Stream Sportsman 16 Gun Fire Safe	6795260.4066467285
    2	365	Perfect Fitness Perfect Rip Deck	4335357.441116333 
    3	957	Diamondback Women's Serene Classic Comfort Bi	4038330.9078979492
    4	191	Nike Men's Free 5.0+ Running Shoe	3586941.2666854858
    5	502	Nike Men's Dri-FIT Victory Golf Polo	3082050
    6	1073	Pelican Sunstream 100 Kayak	3033648.3933258057
    7	403	Nike Men's CJ Elite 2 TD Football Cleat	2831052.3296356201
    8	1014	O'Brien Men's Neoprene Life Vest	2830867.1741104126
    9	627	Under Armour Girls' Toddler Spine Surge Runni	1242929.2107200623
    10	565	adidas Youth Germany Black/Red Away Match Soc	65940


通过简单对比，发现`/department/apparel/category/featured%20shops/product/adidas%20Kids'%20RG%20III%20Mid%20Football%20Cleat`这个url访问的数量占据第二。

这里就会发现一些问题。

实践证明，如果米没有一个大数据的结构化的分析工具。统计出以上时间可能会花费很多时间。不排除自己搭建的平台的容错兼容分布式等维护问题带来的数据损失。


你帮老板发现了这个问题，老板很高兴，要给你资金支持。你准备大干一把了！

实践三：市场部门要优化市场策略，想通过一些数据的交叉分析（关联性）把单独浏览量少的商品卖出去更多，或者再次统计一下倒数10的商品。

快速的大数据分析，那就是用到apache的spark了。我们可以构建一个spark任务，直观展示商品之间的关联。

通过以下命令执行

    [cloudera@quickstart ~]$ spark-shell --master yarn-client
    Setting default log level to "WARN".
    To adjust logging level use sc.setLogLevel(newLevel).
    SLF4J: Class path contains multiple SLF4J bindings.
    SLF4J: Found binding in [jar:file:/usr/lib/zookeeper/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
    SLF4J: Found binding in [jar:file:/usr/lib/flume-ng/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
    SLF4J: Found binding in [jar:file:/usr/lib/parquet/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
    SLF4J: Found binding in [jar:file:/usr/lib/avro/avro-tools-1.7.6-cdh5.12.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
    SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
    SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
    Welcome to
          ____              __
         / __/__  ___ _____/ /__
        _\ \/ _ \/ _ `/ __/  '_/
       /___/ .__/\_,_/_/ /_/\_\   version 1.6.0
          /_/
    Using Scala version 2.10.5 (Java HotSpot(TM) 64-Bit Server VM, Java 1.7.0_67)
    Type in expressions to have them evaluated.
    Type :help for more information.
    18/04/23 19:59:57 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
    18/04/23 19:59:59 WARN shortcircuit.DomainSocketFactory: The short-circuit local reads feature cannot be used because libhadoop cannot be loaded.
    Spark context available as sc (master = yarn-client, app id = application_1524536469679_0001).
    SQL context available as sqlContext.

下一步，我们首先引入我们需要的类

    import org.apache.hadoop.mapreduce.Job
    import org.apache.hadoop.mapreduce.lib.input.FileInputFormat
    import org.apache.avro.generic.GenericRecord
    import parquet.hadoop.ParquetInputFormat
    import parquet.avro.AvroReadSupport
    import org.apache.spark.rdd.RDD


rdd是spark的核心，一个rdd可以理解为一个可以被分区的只读数据集（当然是分布式的）。一个rdd内有很多分区，分区内又有大量的数据记录。
rdd的操作最终还是落到内存或者硬盘上。

下面我们创建一个rdd，提供给order_items和products表使用

    def rddFromParquetHdfsFile(path: String): RDD[GenericRecord] = {
        val job = new Job()
        FileInputFormat.setInputPaths(job, path)
        ParquetInputFormat.setReadSupportClass(job,
            classOf[AvroReadSupport[GenericRecord]])
        return sc.newAPIHadoopRDD(job.getConfiguration,
            classOf[ParquetInputFormat[GenericRecord]],
            classOf[Void],
            classOf[GenericRecord]).map(x => x._2)
    }

    val warehouse = "hdfs://quickstart/user/hive/warehouse/"
    val order_items = rddFromParquetHdfsFile(warehouse + "order_items");
    val products = rddFromParquetHdfsFile(warehouse + "products");

下一步，我们从order_items表和products表提取出我们想要的数据，以一个列表的形式存在，包含name和quantity，以order排序。

    val orders = order_items.map { x => (
        x.get("order_item_product_id"),
        (x.get("order_item_order_id"), x.get("order_item_quantity")))
    }.join(
      products.map { x => (
        x.get("product_id"),
        (x.get("product_name")))
      }
    ).map(x => (
        scala.Int.unbox(x._2._1._1), // order_id
        (
            scala.Int.unbox(x._2._1._2), // quantity
            x._2._2.toString // product_name
        )
    )).groupByKey()


最后，我们衡量（tally）计算出订单中所有商品的组合次数，比如发现啤酒和纸尿裤这两个关联性特别高。
按顺序排列拿到前10

    val cooccurrences = orders.map(order =>
      (
        order._1,
        order._2.toList.combinations(2).map(order_pair =>
            (
                if (order_pair(0)._2 < order_pair(1)._2)
                    (order_pair(0)._2, order_pair(1)._2)
                else
                    (order_pair(1)._2, order_pair(0)._2),
                order_pair(0)._1 * order_pair(1)._1
            )
        )
      )
    )
    val combos = cooccurrences.flatMap(x => x._2).reduceByKey((a, b) => a + b)
    val mostCommon = combos.map(x => (x._2, x._1)).sortByKey(false).take(10)

最后打印结果

    println(mostCommon.deep.mkString("\n"))

    exit


完整的代码如下：

    // First we're going to import the classes we need
    import org.apache.hadoop.mapreduce.Job
    import org.apache.hadoop.mapreduce.lib.input.FileInputFormat
    import org.apache.avro.generic.GenericRecord
    import parquet.hadoop.ParquetInputFormat
    import parquet.avro.AvroReadSupport
    import org.apache.spark.rdd.RDD

    // Then we create RDD's for 2 of the files we imported from MySQL with Sqoop
    // RDD's are Spark's data structures for working with distributed datasets

    def rddFromParquetHdfsFile(path: String): RDD[GenericRecord] = {
        val job = new Job()
        FileInputFormat.setInputPaths(job, path)
        ParquetInputFormat.setReadSupportClass(job,
            classOf[AvroReadSupport[GenericRecord]])
        return sc.newAPIHadoopRDD(job.getConfiguration,
            classOf[ParquetInputFormat[GenericRecord]],
            classOf[Void],
            classOf[GenericRecord]).map(x => x._2)
    }

    val warehouse = "hdfs://quickstart/user/hive/warehouse/"
    val order_items = rddFromParquetHdfsFile(warehouse + "order_items");
    val products = rddFromParquetHdfsFile(warehouse + "products");

    // Next, we extract the fields from order_items and products that we care about
    // and get a list of every product, its name and quantity, grouped by order

    val orders = order_items.map { x => (
        x.get("order_item_product_id"),
        (x.get("order_item_order_id"), x.get("order_item_quantity")))
    }.join(
      products.map { x => (
        x.get("product_id"),
        (x.get("product_name")))
      }
    ).map(x => (
        scala.Int.unbox(x._2._1._1), // order_id
        (
            scala.Int.unbox(x._2._1._2), // quantity
            x._2._2.toString // product_name
        )
    )).groupByKey()

    // Finally, we tally how many times each combination of products appears
    // together in an order, then we sort them and take the 10 most common

    val cooccurrences = orders.map(order =>
      (
        order._1,
        order._2.toList.combinations(2).map(order_pair =>
            (
                if (order_pair(0)._2 < order_pair(1)._2)
                    (order_pair(0)._2, order_pair(1)._2)
                else
                    (order_pair(1)._2, order_pair(0)._2),
                order_pair(0)._1 * order_pair(1)._1
            )
        )
      )
    )
    val combos = cooccurrences.flatMap(x => x._2).reduceByKey((a, b) => a + b)
    val mostCommon = combos.map(x => (x._2, x._1)).sortByKey(false).take(10)

    // We print our results, 1 per line, and exit the Spark shell

    println(mostCommon.deep.mkString("\n"))

    exit


结果如下：

    scala> println(mostCommon.deep.mkString("\n"))
    (67876,(Nike Men's Dri-FIT Victory Golf Polo,Perfect Fitness Perfect Rip Deck))
    (62924,(O'Brien Men's Neoprene Life Vest,Perfect Fitness Perfect Rip Deck))
    (54399,(Nike Men's Dri-FIT Victory Golf Polo,O'Brien Men's Neoprene Life Vest))
    (39656,(Nike Men's Free 5.0+ Running Shoe,Perfect Fitness Perfect Rip Deck))
    (39314,(Perfect Fitness Perfect Rip Deck,Perfect Fitness Perfect Rip Deck))
    (35092,(Perfect Fitness Perfect Rip Deck,Under Armour Girls' Toddler Spine Surge Runni))
    (33750,(Nike Men's Dri-FIT Victory Golf Polo,Nike Men's Free 5.0+ Running Shoe))
    (33406,(Nike Men's Free 5.0+ Running Shoe,O'Brien Men's Neoprene Life Vest))
    (29835,(Nike Men's Dri-FIT Victory Golf Polo,Nike Men's Dri-FIT Victory Golf Polo))
    (29342,(Nike Men's Dri-FIT Victory Golf Polo,Under Armour Girls' Toddler Spine Surge Run



简单的说，map就是通过提取过滤指定的字段，进行方法的invoke map。reduce是join && group by。

如果没有spark这种分析工具，统计这些数据是很话费时间并且很困难。然后用scala几行代码。你就会分析出来订单中n多商品的相互关联性。
并且花费很少时间。

翻篇儿：

领导找你：数据有问题，赶紧过来看，怎么干的事情！

你:刚得瑟几天，怎么出大事了，赶紧去看看what happened

现在我们讲一讲实时的日志同步，并且以多维度去筛选。用到的是apache的flume 和apache的solr。钻取（drill down）和探取（exploration）

solr以类sql形式组织数据。每条数据也是叫document（文档或者集合），每个文档包含字段（类似于mysql的schema），
solr的数据很灵活，而且可以全文索引中某个字段。solr也是把数据分布式放在各个分片上。并且在查询的时候可以自动均衡，提高响应速度。

solr就不说了，现在都是elk了。。

完



