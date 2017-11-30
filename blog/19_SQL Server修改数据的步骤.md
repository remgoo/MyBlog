Title: 19_SQL Server修改数据的步骤
Date: 2016-06-02 11:26:19
Author: peter
Postid: 19
Slug: SQL Server
Nicename: SQL Server

# SQL Server修改数据的步骤


SQL Server修改数据的步骤
SQL Server对于数据的修改,会分为以下几个步骤顺序执行:
1.在SQL Server的缓冲区的日志中写入”Begin Tran”记录
2.在SQL Server的缓冲区的日志页写入要修改的信息
3.在SQL Server的缓冲区将要修改的数据写入数据页
4.在SQL Server的缓冲区的日志中写入”Commit”记录
5.将缓冲区的日志写入日志文件
6.发送确认信息(ACK)到客户端(SMSS,ODBC等）
 
可以看到,事务日志并不是一步步写入磁盘.而是首先写入缓冲区后，一次性写入日志到磁盘.这样既能在日志写入磁盘这块减少IO，还能保证日志LSN的顺序.
上面的步骤可以看出，即使事务已经到了Commit阶段，也仅仅只是把缓冲区的日志页写入日志，并没有把数据写入数据库.
那将要修改的数据页写入数据库是在何时发生的呢?
 
Lazy Writer和CheckPoint
上面提到，SQL Server修改数据的步骤中并没有包含将数据实际写入到磁盘的过程.实际上，将缓冲区内的页写入到磁盘是通过两个过程中的一个实现：
这两个过程分别为:
1.CheckPoint
2.Lazy Writer
任何在缓冲区被修改的页都会被标记为“脏”页。将这个脏页写入到数据磁盘就是CheckPoint或者Lazy Writer的工作.
> 当事务遇到Commit时，仅仅是将缓冲区的所有日志页写入磁盘中的log日志文件。
> 直到Lazy Writer或CheckPoint时，才真正将缓冲区的数据页写入磁盘data文件。

前面说过，日志文件中的LSN号是可以比较的，如果LSN2>LSN1,则说明LSN2的发生时间晚于LSN1的发生时间。CheckPoint或Lazy Writer通过将日志文件末尾的LSN号和缓冲区中数据文件的LSN进行对比，只有缓冲区内LSN号小于日志文件末尾的LSN号的数据才会被写入到磁盘中的数据库。因此确保了WAL（在数据写入到数据库之前，先写入日志)。


Lazy Writer和CheckPoint的区别
Lazy Writer和CheckPoint往往容易混淆。因为Lazy Writer和CheckPoint都是将缓冲区内的“脏”页写入到磁盘文件当中。但这也仅仅是他们唯一的相同点了。
Lazy Writer存在的目的是对缓冲区进行管理。当缓冲区达到某一临界值时，Lazy Writer会将缓冲区内的脏页存入磁盘文件中，而将未修改的页释放并回收资源。
而CheckPoint存在的意义是减少服务器的恢复时间(Recovery Time).CheckPoint就像他的名字指示的那样，是一个存档点.CheckPoint会定期发生.来将缓冲区内的“脏”页写入磁盘。但不像Lazy Writer,Checkpoint对SQL Server的内存管理毫无兴趣。所以CheckPoint也就意味着在这个点之前的所有修改都已经保存到了磁盘.这里要注意的是：CheckPoint会将所有缓冲区的脏页写入磁盘，不管脏页中的数据是否已经Commit。这意味着有可能已经写入磁盘的“脏页”会在之后回滚（RollBack).不过不用担心，如果数据回滚，SQL Server会将缓冲区内的页再次修改，并写入磁盘。


---