<center>
<h3>[原创]CentOS 6.6下proftpd-1.3.6源码安装配置</h3>
山东大学信息化工作办公室 陈军<br>
20170524
</center>
	
&emsp;&emsp;在proftpd官网 http://www.proftpd.org/ 下载最新版源码，当前是v1.3.6。
下载后的文件保存在/home/pkgtemp 目录下。

	cd /home/pkgtemp
	tar zxvf proftpd-1.3.6.tar.gz	#解压缩
	cd proftpd-1.3.6
	#设置安装位置，允许设置本地字符集
	./configure --prefix=/usr/local/proftpd --enable-nls 
    make
	make install
	cd /usr/local/prftpd/etc
	cp proftpd.conf proftpd.conf.bak
	vi proftpd.conf		#修改配置文件
	
	# This is a basic ProFTPD configuration file (rename it to 
	# 'proftpd.conf' for actual use.  It establishes a single server
	# and a single anonymous login.  It assumes that you have a user/group
	# "nobody" and "ftp" for normal operation and anon.
	
	ServerName				"ftp"
	ServerType				standalone
	DefaultServer			on
	DefaultAddress			192.168.100.100		#ftp服务器IP地址
	ServerIdent				on "FTP server ready."
	IdentLookups			off
	UseReverseDNS			off 
	RequireValidShell		off
	DefaultRoot				~
	TimesGMT                off
	
	TransferLog             None
	LogFormat logfmt        "%a %t %T %u %f %b \"%r\""	#设置日志格式
	#设置日志文件，日志内容
	ExtendedLog     /var/log/xferlog        read,write      logfmt
	
	
	# Port 21 is the standard FTP port.
	Port				21
	# Umask 022 is a good standard umask to prevent new dirs and files
	# from being group and world writable.
	Umask				022
	# Don't use IPv6 support by default.
	UseIPv6                         off
	
	AllowRetrieveRestart	on           # 允许下载续传，默认即开启
	AllowStoreRestart		on
	PassivePorts 50000 63000			#被动模式使用的临时端口范围
	UseEncoding			UTF-8	GBK		#设置字符集服务器端utf-8，客户端GBK
	
	# To prevent DoS attacks, set the maximum number of child processes
	# to 30.  If you need to allow more than 30 concurrent connections
	# at once, simply increase this value.  Note that this ONLY works
	# in standalone mode, in inetd mode you should use an inetd server
	# that allows you to limit maximum number of processes per service
	# (such as xinetd)
	MaxInstances			30
	MaxClients 			100
	MaxClientsPerHost 		5
	
	TimeoutIdle			120
	# Set the user and group that the server normally runs at.
	User				nobody
	Group				nobody
	
	# Normally, we want files to be overwriteable.
	<Directory /*>
	  AllowOverwrite		on
	</Directory>
	
	DisplayLogin			welcome.msg
	DeferWelcome			on
	# A basic anonymous configuration, no upload directories.
	<Anonymous ~ftp>
	  User				ftp
	  Group				ftp
	  # We want clients to be able to login with "anonymous" as well as "ftp"
	  UserAlias			anonymous ftp
	
	  # Limit the maximum number of anonymous logins
	  MaxClients			none
	
	  # We want 'welcome.msg' displayed at login, and '.message' displayed
	  # in each newly chdired directory.
	  DisplayLogin			welcome.msg
	  #DisplayFirstChdir		.message
	
	  # upload directory is ~ftp/incoming, allow any user to upload,mkdir,
	  # deny any user to delete file,rmdir
	  <Directory incoming>
		<Limit STOR>
		   #AllowAll
		   DenyAll
		</Limit>
		<Limit WRITE>
		   #AllowAll
		   DenyAll
		</Limit>
		<Limit DELE>
		   DenyAll
		</Limit>
		<Limit RMD>
		   DenyAll
		</Limit>
	  </Directory> 	
	
	  # Limit WRITE everywhere in the anonymous chroot
	  <Limit WRITE>
	    DenyAll
	  </Limit>
	
	</Anonymous>
	
	
对1.30及以上版本，需要建立下面符号链接才可以在日志文件中使用本地时间。

	#ln -s /usr/share/zoneinfo/Asia/Shanghai /usr/share/zoneinfo/CST

启动proftpd:

	/usr/local/prpftpd/sbin/proftpd

设置开机自动启动：

	vi /etc/rc.d/rc.local

	在文件末尾增加下面一行：

	/usr/local/proftpd/sbin/proftpd


&nbsp;

**参考文献**：

[1]. ProFTPd 1.30与服务器系统时间相差8小时，http://club.topsage.com/thread-218657-1-1.html

