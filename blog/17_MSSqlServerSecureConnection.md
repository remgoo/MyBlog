Title: 17_MSSqlServerSecureConnection
Date: 2016-05-31 18:41:32
Author: peter
Postid: 17
Slug: sql server
Nicename: MSSqlServerSecureConnection

# MSSqlServerSecureConnection

==integrated security=true== 的意思是集成验证，也就是说使用Windows验证的方式去连接到数据库服务器。  
这样方式的好处是不需要在连接字符串中编写用户名和密码，从一定程度上说提高了安全性。

==============
Microsoft安全支持提供器接口（SSPI =Security Support Provider Interface）是定义得较全面的公用API，用来获得验证、信息完整性、信息隐私等集成安全服务，以及用于所有分布式应用程序协议的安全方面的服务。应用程序协议设计者能够利用该接口获得不同的安全性服务而不必修改协议本身。  
Integrated Security=SSPI 
Integrated Security 身份验证方式  
当为false时，将在连接中指定用户ID和密码。  
当为true时，将使用当前的Windows帐户凭据进行身份验证。  
可识别的值为true、false、yes、no以及与true等效的sspi。
  
==============

那么到底是用哪一个Windows身份呢？很多朋友说，使用当前用户的身份吧？
这个回答不能算错，至少在Windows应用程序中是这样的。但如果换成是ASP.NET应用程序，则就不是了。

如果是ASP.NET应用程序（网站或者服务），那么根据其运行宿主环境的不一样，可能会有差异

1. Windows XP ：ASPNET帐号

2. Windows 2003或者以后的版本：NetWork Service帐号

知道这个原理之后，那么如果你准备用Integrated security=true，则需要授予这两个帐号对于数据库的访问权限。

但要注意一个问题（也是很多朋友疑惑的），就是在Visual Studio里面调试的时候，貌似又不是使用ASPNET这个帐号的。这是因为Visual Studio总是使用当前开发环境中，用户的Windows身份来发起请求的。

从下面的图可以看到这个差别。在VS里面调试，与在IIS中调试，访问的身份是不一样的

---

1.Sql Server 设置为 “Sql Server and Windows Authentication mode”

2.Sql Server 中为数据库创建一个用户并且分配权限给用户

2.程序连接字符串使用如下格式
```
集成验证连接字符串  
<add name="ConnectionString"  connectionString="Data Source=DBServer;Initial Catalog=DBName;Integrated Security=True" providerName="System.Data.SqlClient" />

sspi身份验证方式  
<add name="ConnectionString"  connectionString="Data Source=DBServer;Initial Catalog=DBName;Integrated Security=SSPI" providerName="System.Data.SqlClient" />


传统连接字符串  
<add name="ConnectionString" connectionString="Data Source=DBServer;Initial Catalog=DBName;User Id=user;Password=password" providerName="System.Data.SqlClient" />
```
3.部署的时候IIS应用程序池配置数据库的用户名和密码

---
