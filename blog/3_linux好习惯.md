Title: linux好习惯
Date: 2012-02-14 21:41:57
Author: peter
Postid: 3
Slug: linux
Nicename: linux


git remote add upstream https://github.scm.corp.ebay.com/montage/frontend-ui-workspace

好习惯 1 使用一个命令来定义目录树
好习惯 1 的示例：使用一个命令来定义目录树
mkdir -p tmp/a/b/c

好习惯 2 更改路径；不要移动存档
好习惯 2 的示例：使用选项 -C 来解压缩 .tar 存档文件
~ $ tar xvf -C tmp/a/b/c newarc.tar.gz
相对于将存档文件移动到您希望在其中解压缩它的位置，切换到该目录，然后才解压缩它，养成使用 -C 的习惯则更加可取——当存档文件位于其他某个位置时尤其如此。

好习惯 3 将命令与控制操作符组合使用
好习惯 3 的示例：将命令与控制操作符组合使用
~ $ cd tmp/a/b/c && tar xvf ~/archive.tar
好习惯 3 的另一个示例：将命令与控制操作符组合使用
~ $ cd tmp/a/b/c || mkdir -p tmp/a/b/c
好习惯 3 的组合示例：将命令与控制操作符组合使用
~ $ cd tmp/a/b/c || mkdir -p tmp/a/b/c && tar xvf -C tmp/a/b/c ~/archive.tar

好习惯 5使用转义序列来管理较长的输入
好习惯 5 的示例：将反斜杠用于长输入
~ $ cd tmp/a/b/c || \
> mkdir -p tmp/a/b/c && \
> tar xvf -C tmp/a/b/c ~/archive.tar

查找文件
$ locate filename

查找大于 10MB 的所有文件
$ find / -size +10000k –xdev –exec ls –lh {}\;

把当前一个文件copy到远程
scp /home/daisy/full.tar.gz root@172.19.2.75:/root
然后会提示你输入另外那台172.19.2.75主机的root用户的登录密码，就开始copy了。 

把文件从远程主机copy到当前系统
scp root@/full.tar.gz 172.19.2.75:/root/full.tar.gz /home/daisy/full.tar.gz 
