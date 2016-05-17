Title: windows10运行java8的applet
Date: 2016-05-17 13:42:55
Author: peter
Postid: 4
Slug: tomcat
Nicename: tomcat

# windows10运行java8的applet

如果项目是发布到tomcat的，项目测试完毕上线的时候记得修改“tomcat-users.xml”文件,打开tomcat的manager管理功能.
修改配置文件添加管理员权限和密码，这样以后项目在运行的过程中可以做更新和重启，部署，方便处理问题。


apache-tomcat-6.0.14\conf\tomcat-users.xml

`  
<?xml version='1.0' encoding='utf-8'?>
<tomcat-users>
  <role rolename="manager"/>
  <user username="tomcat" password="tomcat" roles="manager"/>
</tomcat-users>
`

