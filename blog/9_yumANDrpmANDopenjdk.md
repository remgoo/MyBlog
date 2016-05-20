Title: yumANDrpmANDopenjdk
Date: 2016-05-17 13:42:55
Author: peter
Postid: 8
Slug: linux
Nicename: linux

# yumANDrpmANDopenjdk

*查看Linux自带的JDK是否已安装 
>java -version


*查看本机jdk的信息
>rpm -qa|grep java 

>rpm -qa|grep openjdk


*卸载当前的jdk版本
>yum -y remove java java-1.6-*

>rpm -qa|grep openjdk


*yum库查看是否有java安装包
>yum -y list java*

>yum search openjdk


*用yum安装jdk
>yum -y install  java-1.6.0-openjdk*