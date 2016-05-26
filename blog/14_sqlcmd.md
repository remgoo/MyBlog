Title: 14_sqlcmd
Date: 2016-05-26 12:15:56
Author: peter
Postid: 14
Slug: sqlcmd
Nicename: sqlcmd

# 14_sqlcmd


#### 使用sqlcmd导入MS SQL数据库。可以用来从MSSQL高版本导出数据到低版本。

1.导出的数据库中产生script脚本时选择导出数据，和选择对应的数据库版本。

2.拷贝script.sql到目标电脑c:/scriptpath

1. 打开  cmd

2,cd c:/scriptpath

3.sqlcmd -i  script.sql  -s 127.0.0.1

