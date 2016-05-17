Title: mysql服务性能优化—my.cnf配置说明详解（16G内存）
Date: 2016-05-17 13:42:55
Author: peter
Postid: 6
Slug: tomcat
Nicename: tomcat

# mysql服务性能优化—my.cnf配置说明详解（16G内存）
url:https://www.linuxyw.com/a/shujuku/20130506/216.html

MySQL内存及虚拟内存优化设置 (2011-07-12 11:09:14)转载▼
标签： mysql 优化 	分类： 计算机学习
mysql 优化调试命令
 
1、
mysqld --verbose --help
这个命令生成所有mysqld选项和可配置变量的列表
2、
 
通过连接它并执行这个命令，可以看到实际上使用的变量的值：
mysql> SHOW VARIABLES;
还可以通过下面的语句看到运行服务器的统计和状态指标：
mysql>SHOW STATUS；
使用mysqladmin还可以获得系统变量和状态信息：
shell> mysqladmin variables
shell> mysqladmin extended-status
shell> mysqladmin flush-table 命令可以立即关闭所有不使用的表并将所有使用中的表标记为已经关闭，这样可以有效释放大多数使用中的内存。FLUSH TABLE在关闭所有表之前不返回结果。
 
swap -s检查可用交换区
 
mysql内存计算公式
 
mysql used mem = key_buffer_size + query_cache_size + tmp_table_size
+ innodb_buffer_pool_size + innodb_additional_mem_pool_size
+ innodb_log_buffer_size
+ max_connections * (
read_buffer_size + read_rnd_buffer_size
+ sort_buffer_size+ join_buffer_size
+ binlog_cache_size + thread_stack
)
 
在mysql 中输入如下命令，可自动计算自己的当前配置最大的内存消耗
 
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
SHOW VARIABLES LIKE 'innodb_additional_mem_pool_size';
SHOW VARIABLES LIKE 'innodb_log_buffer_size';
SHOW VARIABLES LIKE 'thread_stack';
SET @kilo_bytes = 1024;
SET @mega_bytes = @kilo_bytes * 1024;
SET @giga_bytes = @mega_bytes * 1024;
SET @innodb_buffer_pool_size = 2 * @giga_bytes;
SET @innodb_additional_mem_pool_size = 16 * @mega_bytes;
SET @innodb_log_buffer_size = 8 * @mega_bytes;
SET @thread_stack = 192 * @kilo_bytes;
SELECT
( @@key_buffer_size + @@query_cache_size + @@tmp_table_size
+ @innodb_buffer_pool_size + @innodb_additional_mem_pool_size
+ @innodb_log_buffer_size
+ @@max_connections * (
@@read_buffer_size + @@read_rnd_buffer_size + @@sort_buffer_size
+ @@join_buffer_size + @@binlog_cache_size + @thread_stack
) ) / @giga_bytes AS MAX_MEMORY_GB;
 
 
mysql关键参数设置
Mysqld 数据库的参数设置有两种类型，
 
一种是全局参数，影响服务器的全局操作；
另一种是会话级参数，只影响当前的客户端连接的相关操作。
 
服务器启动时，所有全局参数都初始化为默认值。可以在初始化文件或命令行中指定的选项来更改这些默认值。服务器启动后，通过连接服务器并执行 SET GLOBAL var_name 语句可以更改动态全局参数。要想更改全局参数，必须具有 SUPER 权限。全局参数的修改只对新的连接生效，已有的客户端连接并不会生效。
 
服务器还可以为每个客户端连接维护会话级参数，客户端连接时使用相应全局参数的当前值对客户端会话参数进行初始化。客户可以通过 SET SESSION var_name 语句来更改动态会话参数。设置会话级参数不需要特殊权限，但每个客户端可以只更改自己的会话级参数，不能更改其它客户的会话级参数。
不指定设置的参数类型时，默认设置的是会话级参数。
 

(1)、max_connections：
允许的同时客户的数量。增加该值增加 mysqld 要求的文件描述符的数量。这个数字应该增加，否则，你将经常看到 too many connections 错误。 默认数值是100，我把它改为1024 。

(2)、record_buffer：
每个进行一个顺序扫描的线程为其扫描的每张表分配这个大小的一个缓冲区。如果你做很多顺序扫描，你可能想要增加该值。默认数值是131072(128k)，我把它改为16773120 (16m)

(3)、key_buffer_size：
为了最小化磁盘的 I/O ， MyISAM 存储引擎的表使用键高速缓存来缓存索引，这个键高速缓存的大小则通过 key-buffer-size 参数来设置。如果应用系统中使用的表以 MyISAM 存储引擎为主，则应该适当增加该参数的值，以便尽可能的缓存索引，提高访问的速度。

索引块是缓冲的并且被所有的线程共享。key_buffer_size是用于索引块的缓冲区大小，增加它可得到更好处理的索引(对所有读和多重写)，到你能负担得起那样多。如果你使它太大，系统将开始换页并且真的变慢了。默认数值是8388600(8m)，我的mysql主机有2gb内存，所以我把它改为 402649088(400mb)。
默认情况下，所有的索引都使用相同的键高速缓存，当访问的索引不在缓存中时，使用 LRU （ Least Recently Used 最近最少使用）算法来替换缓存中最近最少使用的索引块。为了进一步避免对键高速缓存的争用，从 MySQL5.1 开始，可以设置多个键高速缓存，并为不同的索引键指定使用的键高速缓存。下面的例子演示如何修改高速键缓存的值，如何设置多个键高速缓存，以及如何为不同的索引指定不同的缓存：
显示当前的参数大小，为16M：
mysql> show variables like 'key_buffer_size';
+-----------------+-------+
| Variable_name | Value |
+-----------------+-------+
| key_buffer_size | 16384 |
+-----------------+-------+
1 row in set (0.00 sec)
修改参数值到200M：
mysql> set global key_buffer_size=204800;
Query OK, 0 rows affected (0.00 sec)
mysql> show variables like 'key_buffer_size';
+-----------------+--------+
| Variable_name | Value |
+-----------------+--------+
| key_buffer_size | 204800 |
+-----------------+--------+
1 row in set (0.00 sec)
上面介绍的是默认的键缓存，下面介绍如何设置多个键缓存：
设置 hot_cache 的键缓存 100M ， cold_cache 的键缓存 100M ，另外还有 200M 的默认的键缓存。如果索引不指定键缓存，则会放在默认的键缓存中。
mysql> set global hot_cache.key_buffer_size=102400;
Query OK, 0 rows affected (0.00 sec)
mysql> set global cold_cache.key_buffer_size= 1024 00;
Query OK, 0 rows affected (0.01 sec)
mysql> show variables like 'key_buffer_size';
+-----------------+--------+
| Variable_name | Value |
+-----------------+--------+
| key_buffer_size | 204800 |
+-----------------+--------+
1 row in set (0.00 sec)
如果要显示设置的多键缓存的值，可以使用：
mysql> SELECT @@global.hot_cache.key_buffer_size;
+------------------------------------+
| @@global.hot_cache.key_buffer_size |
+------------------------------------+
| 102400 |
+------------------------------------+
1 row in set (0.03 sec)
mysql> SELECT @@global.cold_cache.key_buffer_size;
+-------------------------------------+
| @@global.cold_cache.key_buffer_size |
+-------------------------------------+
| 102400 |
+-------------------------------------+
1 row in set (0.00 sec)
指定不同的索引使用不同的键缓存：
mysql> CACHE INDEX test1 in hot_cache;
+------------+--------------------+----------+----------+
| Table | Op | Msg_type | Msg_text |
+------------+--------------------+----------+----------+
| test .test1 | assign_to_keycache | status | OK |
+------------+--------------------+----------+----------+
1 row in set (0.00 sec)
mysql> CACHE INDEX test2 in hot_cache;
+------------+--------------------+----------+----------+
| Table | Op | Msg_type | Msg_text |
+------------+--------------------+----------+----------+
| test .test2 | assign_to_keycache | status | OK |
+------------+--------------------+----------+----------+
1 row in set (0.00 sec)
通常在数据库刚刚启动的时候，需要等待数据库热起来，也就是等待数据被缓存到缓存区中，这段时间数据库会因为 buffer 的命中率低而导致应用的访问效率不高。使用键高速缓存的时候，可以通过命令将索引预加载到缓存区中，大大缩短了数据库预热的时间。具体的操作方式是：
mysql> LOAD INDEX INTO CACHE test1,test2 IGNORE LEAVES;
+------------+--------------+----------+----------+
| Table | Op | Msg_type | Msg_text |
+------------+--------------+----------+----------+
| test .test1 | preload_keys | status | OK |
| test .test2 | preload_keys | status | OK |
+------------+--------------+----------+----------+
2 rows in set (3.89 sec)
如果已经使用 CACHE INDEX 语句为索引分配了一个键高速缓冲，预加载可以将索引块放入该缓存，否则，索引块将被加载到默认的键高速缓冲。
 

4)、back_log：
要求 mysql 能有的连接数量。当主要mysql线程在一个很短时间内得到非常多的连接请求，这就起作用，然后主线程花些时间(尽管很短)检查连接并且启动一个新线程。
back_log 值指出在mysql暂时停止回答新请求之前的短时间内多少个请求可以被存在堆栈中。只有如果期望在一个短时间内有很多连接，你需要增加它，换句话说，这值对到来的tcp/ip连接的侦听队列的大小。你的操作系统在这个队列大小上有它自己的限制。试图设定back_log高于你的操作系统的限制将是无效的。
当你观察你的主机进程列表，发现大量 264084 | unauthenticated user | xxx.xxx.xxx.xxx | null | connect | null | login | null 的待连接进程时，就要加大 back_log 的值了。默认数值是50，我把它改为500。

(5)、interactive_timeout：
服务器在关闭它前在一个交互连接上等待行动的秒数。一个交互的客户被定义为对 mysql_real_connect()使用 client_interactive 选项的客户。 默认数值是28800，我把它改为7200。


(6)、sort_buffer：
每个需要进行排序的线程分配该大小的一个缓冲区。增加这值加速order by或group by操作。默认数值是2097144(2m)，我把它改为 16777208 (16m)。

(7)、table_cache：
为所有线程打开表的数量。增加该值能增加mysqld要求的文件描述符的数量。mysql对每个唯一打开的表需要2个文件描述符。默认数值是64，我把它改为512。

(8)、thread_cache_size：
可以复用的保存在中的线程的数量。如果有，新的线程从缓存中取得，当断开连接的时候如果有空间，客户的线置在缓存中。如果有很多新的线程，为了提高性能可以这个变量值。通过比较 connections 和 threads_created 状态的变量，可以看到这个变量的作用。我把它设置为 80。

(9)mysql的搜索功能
用mysql进行搜索，目的是能不分大小写，又能用中文进行搜索
只需起动mysqld时指定 --default-character-set=gb2312

(10)、wait_timeout：
服务器在关闭它之前在一个连接上等待行动的秒数。 默认数值是28800，我把它改为7200。
 
(11)、innodb_thread_concurrency：
你的服务器CPU有几个就设置为几,默认为8。
 
(12)、query_cache_size  与 query_cache_limit
QueryCache 之后所带来的负面影响：
a) Query 语句的hash 运算以及hash 查找资源消耗。当我们使用Query Cache 之后，每条SELECT
类型的Query 在到达MySQL 之后，都需要进行一个hash 运算然后查找是否存在该Query 的
Cache，虽然这个hash 运算的算法可能已经非常高效了，hash 查找的过程也已经足够的优化
了，对于一条Query 来说消耗的资源确实是非常非常的少，但是当我们每秒都有上千甚至几千
条Query 的时候，我们就不能对产生的CPU 的消耗完全忽视了。
b) Query Cache 的失效问题。如果我们的表变更比较频繁，则会造成Query Cache 的失效率非常
高。这里的表变更不仅仅指表中数据的变更，还包括结构或者索引等的任何变更。也就是说我
们每次缓存到Query Cache 中的Cache 数据可能在刚存入后很快就会因为表中的数据被改变而被
清除，然后新的相同Query 进来之后无法使用到之前的Cache。
c) Query Cache 中缓存的是Result Set ，而不是数据页，也就是说，存在同一条记录被Cache 多
次的可能性存在。从而造成内存资源的过渡消耗。当然，可能有人会说我们可以限定Query
Cache 的大小啊。是的，我们确实可以限定Query Cache 的大小，但是这样，Query Cache 就很
容易造成因为内存不足而被换出，造成命中率的下降。
 
QueryCache 的正确使用：
虽然Query Cache 的使用会存在一些负面影响，但是我们也应该相信其存在是必定有一定价值。我
们完全不用因为Query Cache 的上面三个负面影响就完全失去对Query Cache 的信心。只要我们理解了
Query Cache 的实现原理，那么我们就完全可以通过一定的手段在使用Query Cache 的时候扬长避短，重
发发挥其优势，并有效的避开其劣势。
首先，我们需要根据Query Cache 失效机制来判断哪些表适合使用Query 哪些表不适合。由于Query
Cache 的失效主要是因为Query 所依赖的Table 的数据发生了变化，造成Query 的Result Set 可能已经
有所改变而造成相关的Query Cache 全部失效，那么我们就应该避免在查询变化频繁的Table 的Query 上
使用，而应该在那些查询变化频率较小的Table 的Query 上面使用。MySQL 中针对Query Cache 有两个专
用的SQL Hint（提示）：SQL_NO_CACHE 和SQL_CACHE，分别代表强制不使用Query Cache 和强制使用
Query Cache。我们完全可以利用这两个SQL Hint，让MySQL 知道我们希望哪些SQL 使用Query Cache 而
哪些SQL 就不要使用了。这样不仅可以让变化频繁Table 的Query 浪费Query Cache 的内存，同时还可以
减少Query Cache 的检测量。
其次，对于那些变化非常小，大部分时候都是静态的数据，我们可以添加SQL_CACHE 的SQL Hint，
强制MySQL 使用Query Cache，从而提高该表的查询性能。
最后，有些SQL 的Result Set 很大，如果使用Query Cache 很容易造成Cache 内存的不足，或者将
之前一些老的Cache 冲刷出去。对于这一类Query 我们有两种方法可以解决，一是使用SQL_NO_CACHE 参
数来强制他不使用Query Cache 而每次都直接从实际数据中去查找， 另一种方法是通过设定
“query_cache_limit”参数值来控制Query Cache 中所Cache 的最大Result Set ，系统默认为
1M（1048576）。当某个Query 的Result Set 大于“query_cache_limit”所设定的值的时候，Query
Cache 是不会Cache 这个Query 的。

(13)、innodb_buffer_pool_size
innodb_buffer_pool_size 定义了 InnoDB 存储引擎的表数据和索引数据的最大内存缓冲区大小。和 MyISAM 存储引擎不同， MyISAM 的 key_buffer_size 只能缓存索引键，而 innodb_buffer_pool_size 却可以缓存数据块和索引键。适当的增加这个参数的大小，可以有效的减少 InnoDB 类型的表的磁盘 I/O 。在一个以 InnoDB 为主的专用数据库服务器上，可以考虑把该参数设置为物理内存大小的 60%-80%
 
InnoDB占用的内存，除innodb_buffer_pool_size用于存储页面缓存数据外，另外正常情况下还有大约8%的开销，主要用在每个缓存页帧的描述、adaptive hash等数据结构，如果不是安全关闭，启动时还要恢复的话，还要另开大约12%的内存用于恢复，两者相加就有差不多21%的开销。

这样，12G的innodb_buffer_pool_size，最多的时候InnoDB就可能占用到14.5G(12G X 21%)的内存，再加上操作系统用的几百M，近千个线程堆栈，就差不多16G了。
 
MAX_QUERIES_PER_HOUR 用来限制用户每小时运行的查询数量：
mysql> grant all on dbname。* to db@localhost identified by “123456” with max_connections_per_hour 5;
（db用户在dbname的数据库上控制用户每小时打开新连接的数量为5个）
 
MAX_USER_CONNECTIONS 限制有多少用户连接MYSQL服务器：
mysql> grant all on dbname。* to db@localhost identified by “123456” with max_user_connections 2;
（db用户在dbname的数据库账户一次可以同时连接的最大连接数为2个）

MAX_UPDATES_PER_HOUR 用来限制用户每小时的修改数据库数据的数量：
mysql> grant all on dbname。* to db@localhost identified by “123456” with max_updates_per_hour 5;
（db用户在dbname的数据库上控制用户每小时修改更新数据库的次数为5次）
MAX_USER_CONNECTIONS 用来限制用户每小时的修改数据库数据的数量：
mysql> grant all on dbname。* to db@localhost identified by “123456”
With MAX_QUERIES_PER_HOUR 20 ;指mysql单个用户的最大连接数
（db用户在dbname的数据库上控制用户每小时的连接数为20个）
 
调优举例
针对my.cnf文件进行优化：
[mysqld]
skip-locking（取消文件系统的外部锁）
skip-name-resolve（不进行域名反解析,注意由此带来的权限/授权问题）
key_buffer_size = 256M（分配给MyISAM索引缓存的内存总数）对于内存在4GB左右的服务器该参数可设置为256M或384M。
  注意：该参数值设置的过大反而会是服务器整体效率降低！
  max_allowed_packet = 4M（允许最大的包大小）
  thread_stack = 256K（每个线程的大小）
  table_cache = 128K（缓存可重用的线程数）
  back_log = 384（临时停止响应新请求前在短时间内可以堆起多少请求，如果你需要在短时间内允许大量连接，可以增加该数值）
  sort_buffer_size = 2M(分配给每个线程中处理排序)
  read_buffer_size = 2M（读取的索引缓冲区大小）
  join_buffer_size = 2M（分配给每个线程中处理扫描表连接及索引的内存）
  myisam_sort_buffer_size = 64M（myisam引擎排序缓冲区的大小）
  table_cache = 512（缓存数据表的数量，避免重复打开表的开销）
  thread_cache_size = 64（缓存可重用线程数，见笑创建新线程的开销）
  query_cache_size = 64M（控制分配给查询缓存的内存总量）
  tmp_table_size = 256M(指定mysql缓存的内存大小)
  max_connections = 768（最大连接数）指mysql整个的最大连接数
max_connect_errors = 10000(最大连接错误数据)
  wait_timeout = 10（超时时间，可以避免攻击）
  thread_concurrency = 8（根据cpu数量来设置）
  skip-bdb 禁用不必要的引擎
  skip-networking（关闭mysql tcp/ip连接方式）
  Log-slow-queries = /var/log/mysqlslowqueries.log
  long_query_time = 4（设定慢查询的时间）
  skip-host-cache(提高mysql速度的)
  open_files_limit = 4096(打开文件数）
interactive_timeout = 10(服务器在关闭它前在一个交互连接上等待行动的秒数)
max_user_connections = 500（最大用户连接数）
 
key_buffer_size                 默认为218       调到128最佳
query_cache_size  
tmp_table_size                  默认为16M        调到64-256最挂

innodb_thread_concurrency=8       你的服务器CPU有几个就设置为几,默认为8
 
table_cache=1024 物理内存越大,设置就越大.默认为2402,调到512-1024最佳
innodb_additional_mem_pool_size=8M   默认为2M
innodb_flush_log_at_trx_commit=0 等到innodb_log_buffer_size列队满后再统一储存,默认为1
innodb_log_buffer_size=4M          默认为1M

read_buffer_size=4M                   默认为64K
read_rnd_buffer_size   随机读 缓存区  默认为256K
sort_buffer_size=32M                   默认为256K
max_connections=1024                 默认为1210
thread_cache_size=120             默认为60
 
性能测试
1、mysql 自带测试工具
shell> perl -MCPAN -e shell
cpan> install DBI
cpan> install DBD::mysql
shell> cd sql-bench
shell> perl run-all-tests --server=server_name
server_name是一个支持的服务器。要获得所有选项和支持的服务器，调用命令：
shell> perl run-all-tests --help
2、mysqlreport
 http://hackmysql.com/mysqlreport
 
参考文档
http://dev.mysql.com/doc/refman/5.1/zh/optimization.html
 
http://hackmysql.com/tools
 
http://www.imysql.cn/
 