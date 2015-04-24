---
layout:     post
title:      在Window2008上搭建 Apache FTPServer
category: blog
description: windows2003马上要停止维护了，时间好像是在2015年5月7日。阿里云希望用户能够升级服务器，刚好公司的新项目马上要上线了，所以就尝试了一下将云服务器升级到Windows2008。
---

#在Window2008上搭建 Apache FTPServer

windows2003马上要停止维护了，时间好像是在2015年5月7日。阿里云希望用户能够升级服务器，刚好公司的新项目马上要上线了，所以就尝试了一下将云服务器升级到Windows2008.

是一个小项目，所以生产环境也比较简单：

* Java7
* Tomcat
* Mysql
* FTP服务器（选择的Apache FTPServer）

别的安装没什么说的，和在Windows2003上是一样的。具体说说安装FTP的问题。

##下载

下载个人建议不管下载什么，都最好去官方下载，特别是开源的项目，Apache FTPServer的下载地址是：[下载][apache-ftpserver]

##安装

安装非常简单，解压就可以了。简单看一下目录：

* res 配置的主要文件夹
* common Jar包和类
* bin 工具

我们主要是学习res就可以了，别的不用关心，res的里面的目录：

* conf 该目录下主要存放于FtpServer相关的配置
* home Ftp服务器上传的文件默认就保存在这里，可以通过配置来修改
* log 日志
* 剩下的文件不用太在意

##配置

其实对于全栈工程师来说，不用太纠结于FtpServer的细节，能配置能运行，能帮我们上传文件和下载文件就行了。所以主要是要学会配置。

###conf目录

users.properties，该文件主要是对FtpServer的用户进行配置。
	
	#用户名就是admin（可以改），密码明显是加过密的，暂时不用管，一会讲
    ftpserver.user.admin.userpassword=21232F297A57A5A743894A0E4A801FC3
	#上传文件的目录
	ftpserver.user.admin.homedirectory=./res/home
	#当前用户可用
	ftpserver.user.admin.enableflag=true
	#是否具有上传的权限
	ftpserver.user.admin.writepermission=true
	#最大登陆数量
	ftpserver.user.admin.maxloginnumber=0
	#同IP登陆用户数量
	ftpserver.user.admin.maxloginperip=0
	#空闲时间为300秒
	ftpserver.user.admin.idletime=0
	#上传速度限制	
	ftpserver.user.admin.uploadrate=0
	#下载速度限制
	ftpserver.user.admin.downloadrate=0

再来看**ftpd-typical.xml**文件，打开这个xml文件，找到Server根目录，默认的Server元素只有一个id属性，给它添加几个属性和值，然后修改端口（不修改也行），修改后的**ftpd-typical.xml**文件为：

    <server xmlns="http://mina.apache.org/ftpserver/spring/v1"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="
		   http://mina.apache.org/ftpserver/spring/v1 http://mina.apache.org/ftpserver/ftpserver-1.0.xsd	
		   "
		id="myServer"
	    max-logins="20"  
	    anon-enabled="false"  
	    max-anon-logins="0"  
	    max-login-failures="3"  
	    login-failure-delay="30000"
		>
		<listeners>
			<nio-listener name="default" port="2121">
			    <ssl>
	                <keystore file="./res/ftpserver.jks" password="password" />
	            </ssl>
			</nio-listener>
		</listeners>
		<file-user-manager file="./res/conf/users.properties" encrypt-passwords = "clear" />
	</server>

修改后，根据属性名可以看到，匿名用户被禁用，而且去掉了密码加密 `encrypt-passwords = "clear"` ，然后修改users.properties中admin的密码，是什么密码就直接设置成什么就行了。到这里配置就完成了。

##运行

运行很简单，如果不考虑将FtpServer安装成系统服务，那么打开命令行，将目录切换到FtpServer的bin目录，然后输入` ftpd.bat res/conf/ftpd-typical.xml`就完成了。
> 为了方便使用，一般做法是，在bin目录下，建立一个run.bat文件，将刚才的命令拷贝进入，然后把这个批处理文件发送到桌面，以后就直接可以使用了。运行后，如果出现：ftp-server started 的字样，那就成功了。

##问题

我在运行正常后，出现了两个问题（Window2008系统）

1. 这个服务无法访问，说白了就是端口没有开放
2. 开放端口后，无法读取目录，读写被拦截

###开放端口

这个其实也很简单，主要是你要确认是不是这个原因。一般做法是先查看机器上的端口运行情况：

1. Windows查看所有的端口 `netstat -ano`
2. 查询指定的端口占用 `netstat -aon|findstr "提示的端口"`
3. 查询PID对应的进行进程 `tasklist|findstr "PID"`
4. 然后我们输入命令`taskkill /f /t /im 程序名`即可

确认端口正常，那一般就是防火墙的问题了

###防火墙

`右击我的电脑——管理——配置——高级安全Windows防火墙——入站规则`，然后添加规则，根据提示去填写就行了，FTP也属于TCP，选TCP就行了，别都是“允许”或者“是”就完成了（内网权限，可以在选项卡中设置作用域）。完成以后，发现，客户端可以连接了，但是无法读出目录。应该还是被阻止了，真实一波三折啊。

`控制面板——Windows防火墙——允许程序或功能通过Windows防火墙`，然后点击“允许运行另一程序”，然后浏览，那么问题来了，选择哪个程序呢？对，不是FtpServer，而是Java，因为FtpServer就是Java写的，那么是哪个Java呢？因为JDK里面有一个，JRE里面也有一个，这就要看你的具体环境了，总之是选择一个 `java.exe`。

好了，再用客户端连接一次，OK了！（如果还不行，那就在出站规则里面，再添加规则，把对应的端口再添加一次）





[apache-ftpserver]: https://mina.apache.org/ftpserver-project/downloads.html