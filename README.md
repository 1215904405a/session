# session
session的存储方式和配置


session的存储方式和配置
Session又称为会话状态，是Web系统中最常用的状态，用于维护和当前浏览器实例相关的一些信息。我们控制用户去权限中经常用到Session来存储用户状态，这篇文章会讲下Session的存储方式、在web.config中如何配置Session、Session的生命周期等内容。

 

　　1、Session的存储方式。

　　session其实分为客户端Session和服务器端Session。

　　当用户首次与Web服务器建立连接的时候，服务器会给用户分发一个 SessionID作为标识。SessionID是一个由24个字符组成的随机字符串。用户每次提交页面，浏览器都会把这个SessionID包含在 HTTP头中提交给Web服务器，这样Web服务器就能区分当前请求页面的是哪一个客户端。这个SessionID就是保存在客户端的，属于客户端Session。

　　其实客户端Session默认是以cookie的形式来存储的，所以当用户禁用了cookie的话，服务器端就得不到SessionID。这时我们可以使用url的方式来存储客户端Session。也就是将SessionID直接写在了url中，当然这种方法不常用。

 

　　我们大多数提到的Session都是指服务器端Session。他有三种存储方式(自定义存储在这里不做讨论)：

　　1.1保存在IIS进程中：

　　保存在IIS进程中是指把Session数据保存在IIS的运行的进程中，也就是inetinfo.exe这个进程中，这也是默认的Session的存方式，也是最常用的。

　　这种方式的优点是简单，性能最高。但是当重启IIS服务器时Session丢失。

 

　　1.2.保存在StateServer上

　　这种存储模式是指将Session数据存储在一个称为Asp.Net状态服务进程中，该进程独立于Asp.Net辅助进程或IIS应用程序池的单独进程，使用此模式可以确保在重新启动Web应用程序时保留会话状态，并使会话状态可以用于网络中的多个Web服务器。

 

　　1.3.保存在SQL Server数据库中

　　可以配置把Session数据存储到SQL Server数据库中，为了进行这样的配置，程序员首先需要准备SQL Server数据服务器，然后在运行.NET自带安装工具安装状态数据库。

　　这种方式在服务器挂掉重启后都还在，因为他存储在内存和磁盘中。

　　下面是这三种方式的比较：

 	
InProc

StateServer

SQLServer

存储物理位置

IIS进程（内存）

Windows服务进程（内存）

SQLServer数据库（磁盘）

存储类型限制

无限制

可以序列化的类型

可以序列化的类型

存储大小限制

无限制

使用范围

当前请求上下文，对于每个用户独立

生命周期

第一次访问网站的时候创建Session超时后销毁

优点

性能比较高

Session不依赖Web服务器，不容易丢失

缺点

容易丢失

序列化与反序列化消耗CPU资源

序列化与反序列化消耗CPU资源，从磁盘读取Session比较慢

使用原则

不要存放大量数据

 
 

　　2、在web.config中配置Session

　　Web.config文件中的Session配置信息：

复制代码
复制代码
<sessionState mode="Off|InProc|StateServer|SQLServer"

cookieless="true|false"

timeout="number of minutes"

stateConnectionString="tcpip=server:port"

sqlConnectionString="sql connection string"

stateNetworkTimeout="number of seconds"

/>
复制代码
复制代码
　　mode 设置将Session信息存储到哪里：

　　　　— Off 设置为不使用Session功能；

　　　　— InProc 设置为将Session存储在进程内，就是ASP中的存储方式，这是默认值；

　　　　— StateServer 设置为将Session存储在独立的状态服务中；

　　　　— SQLServer 设置将Session存储在SQL Server中。

　　

　　cookieless 设置客户端的Session信息存储到哪里：

　　　　— ture 使用Cookieless模式；这时客户端的Session信息就不再使用Cookie存储了，而是将其通过URL存储。比如网址为http://localhost/MyTestApplication/(ulqsek45heu3ic2a5zgdl245)/default.aspx

　　　　— false 使用Cookie模式，这是默认值。

 

　　timeout 设置经过多少分钟后服务器自动放弃Session信息。默认为20分钟。

 

　　stateConnectionString 设置将Session信息存储在状态服务中时使用的服务器名称和端口号，例如："tcpip=127.0.0.1:42424”。当mode的值是StateServer是，这个属性是必需的。（42424是默认端口）。

 

　　sqlConnectionString 设置与SQL Server连接时的连接字符串。例如"data source=localhost;Integrated Security=SSPI;Initial Catalog=northwind"。当mode的值是SQLServer时，这个属性是必需的。

 

　　stateNetworkTimeout 设置当使用StateServer模式存储Session状态时，经过多少秒空闲后，断开Web服务器与存储状态信息的服务器的TCP/IP连接的。默认值是10秒钟。

 

　　下面来说下用StateServer和SqlServer来存储Session的方法

　　2.1 StateServer

　　第1步是打开状态服务。依次打开“控制面板”→“管理工具”→“服务”命令，找到ASP.NET状态服务一项，右键单击服务选择启动。

　　如果你正式决定使用状态服务存储Session前，别忘记修改服务为自启动（在操作系统重启后服务能自己启动）以免忘记启动服务而造成网站Session不能使用

　　第2步，在system.web节点中加入：stateNetworkTimeout="20">  stateConnectionString表示状态服务器的通信地址（IP：服务端口号）。由于我们现在在本机进行测试，这里设置成本机地址127.0.0.1。状态服务默认的监听端口为42422。当然，您也可以通过修改注册表来修改状态服务的端口号。

　　(修改注册表来修改状态服务的端口号的方法：在运行中输入regedit启动注册表编辑器—依次打开HKEY_LOCAL_MACHINESYSTEMCurrentControlSetServicesaspnet_stateParameters节点，双击Port选项—选择基数为十进制，然后输入一个端口号即可。)

 

　　2.2 SqlServer

　　在SQL Server中执行一个叫做InstallSqlState.sql的脚本文件。这个脚本文件将在SQL Server中创建一个用来专门存储Session信息的数据库，及一个维护Session信息数据库的SQL Server代理作业。我们可以在以下路径中找到那个文件：

[system drive]\winnt\Microsoft.NET\Framework\[version]\ 

然后打开查询分析器，连接到SQL Server服务器，打开刚才的那个文件并且执行。稍等片刻，数据库及作业就建立好了。这时，你可以打开企业管理器，看到新增了一个叫ASPState的数据库。

　　修改mode的值改为SQLServer。注意，还要同时修改sqlConnectionString的值，格式为：sqlConnectionString="data source=localhost; Integrated Security=SSPI;"(这种是通过windows集成身份验证)

 

　　3、Session的生命周期

　　Session的生命周期其实在第一节已经讲过了，和不同的存储过程有关。

 

　　4、遍历以及销毁Session

　　4.1遍历：

System.Collections.IEnumerator SessionEnum = Session.Keys.GetEnumerator();
while (SessionEnum.MoveNext())
{
    Response.Write(Session[SessionEnum.Current.ToString()].ToString() + " ");
}
　　4.2 销毁：Session.Abandon()。
