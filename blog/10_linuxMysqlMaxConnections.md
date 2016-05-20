Title: linuxMysqlMaxConnections
Date: 2016-05-17 13:42:55
Author: peter
Postid: 10
Slug: mysql
Nicename: mysql

# linuxMysqlMaxConnections

MySQL服务器的连接数并不是要达到最大的100%为好，还是要具体问题具体分析，下面就对MySQL服务器最大连接数的合理设置进行了详尽的分析，供您参考。
我们经常会遇见“MySQL: ERROR 1040: Too many connections”的情况，通常，mysql的最大连接数默认是100, 最大可以达到16384。
一种是访问量确实很高，MySQL服务器抗不住，这个时候就要考虑增加从服务器分散读压力，另外一种情况是MySQL配置文件中max_connections值过小：
mysql> show variables like 'max_connections';
+-----------------+-------+
| Variable_name | Value |
+-----------------+-------+
| max_connections | 256 |
+-----------------+-------+
这台MySQL服务器最大连接数是256，然后查询一下服务器响应的最大连接数：
mysql> show global status like 'Max_used_connections';
MySQL服务器过去的最大连接数是245，没有达到服务器连接数上限256，应该没有出现1040错误，比较理想的设置是：
Max_used_connections / max_connections * 100% ≈ 85%
最大连接数占上限连接数的85%左右，如果发现比例在10%以下，MySQL服务器连接上线就设置得过高了

 在Windows下常用的有两种方式修改最大连接数。
*第一种：命令行修改。
>mysql -uuser -ppassword(命令行登录MySQL)
    mysql>show variables like 'max_connections';(查可以看当前的最大连接数)
    msyql>set global max_connections=1000;(设置最大连接数为1000，可以再次查看是否设置成功)
    mysql>exit(推出)
    这种方式有个问题，就是设置的最大连接数只在mysql当前服务进程有效，一旦mysql重启，又会恢复到初始状态。因为mysql启动后的初始化工作是从其配置文件中读取数据的，而这种方式没有对其配置文件做更改。

*第二种：修改配置文件。
   这种方式说来很简单，只要修改MySQL配置文件my.ini 或 my.cnf的参数max_connections，将其改为max_connections=1000，然后重启MySQL即可。但是有一点最难的就是my.ini这个文件在哪找。通常有两种可能，一个是在安装目录下（这是比较理想的情况），另一种是在数据文件的目录下，安装的时候如果没有人为改变目录的话，一般就在C:/ProgramData/MySQL往下的目录下。