<center>
<h3>CentOS 6.6下fail2ban0.9.4源码安装配置及使用</h3>
山东大学信息化工作办公室 陈军<br>
20170612
</center>
	
&emsp;&emsp;本文介绍fail2ban在CentOS6.6系统下的安装、配置，保护ssh、ftp服务的使用方法。

#####一、安装与配置
在fail2ban官网 http://www.fail2ban.org/ 下载最新版源码，当前是v0.9.4。
下载后的文件保存在/home/pkgtemp 目录下。

	cd /home/pkgtemp
	tar zxvf fail2ban-0.9.4.tar.gz 
	cd fail2ban-0.9.4
	./setup.py install	#安装软件包
	cd files			#设置开机自动启动
	cp redhat-initd /etc/init.d/fail2ban
	chmod 755 /etc/init.d/fail2ban 
	chkconfig --add fail2ban
	chkconfig --list fail2ban
	cd /etc/logrotate.d/
	vi fail2ban			#设置日志循环保存天数

输入下面内容：

	/var/log/fail2ban.log {
	        weekly
	        rotate 7
	        missingok
	        compress
	        postrotate
	                /usr/bin/fail2ban-client set logtarget /var/log/fail2ban.log > /dev/null
	        endscript
	}

CentOS6.6中iptables的版本是1.4.7，该版本对”-w“选项支持有问题，会导致iptables命令出错。需要修改/etc/fail2ban/action.d/iptables-common.conf文件，去掉倒数第6行左右“lockingopt = -w”的参数“-w“。成如下形式：

	lockingopt =

启动fail2ban服务：

	service fail2ban start

查看fail2ban日志：

	more /var/log/fail2ban.log 



#####二、使用方法
对proftpd、sshd进行保护。在/etc/fail2ban目录下创建jail.local文件，内容如下：

	[proftpd]
	enabled = true
	filter  = proftpd
	port    = 21
	action  = iptables[name=ProFTPD, port=21, protocol=tcp]
	logpath = /var/log/secure
	maxretry= 6
	usedns = no
	bantime  = 3600
	findtime  = 300

	[sshd]
	enabled = true
	filter  = sshd
	port    = 22
	action  = iptables[name=SSH, port="%(port)s", protocol=tcp]
	logpath = /var/log/secure
	bantime = 43200
	maxretry= 6
	usedns = no
	bantime  = 3600
	findtime  = 300

重启fail2ban服务：

	service fail2ban restart

查看fail2ban.log应该如下信息：

	Jail 'proftpd' started
	Jail 'sshd' started

&emsp;&emsp;更多网络服务的保护举例详见csdn网站博文<a href="http://blog.csdn.net/shuchengzhang/article/details/50931123">“Fail2ban 防止暴力破解centos服务器的SSH或者FTP账户”</a>、Linux中国网站网文<a href="https://linux.cn/article-5068-qqmail.html">"如何配置fail2ban来保护Apache服务器"</a>

&nbsp;

**参考文献**：

[1]. http://www.fail2ban.org/wiki/index.php/MANUAL_0_8

