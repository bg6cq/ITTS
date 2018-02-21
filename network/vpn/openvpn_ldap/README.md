## OpenVPN的安装与部署（ldap进行身份认证+记录用户访问日志并发送邮件）

本文原创：**中国药科大学 申继年**

修改时间：2018/02/21

**前言：**

本文是先部署后补的文档，所以，文档里难免会留一些小坑，欢迎各位查漏吐槽，哈哈。

文中的三个脚本均来自互联网，只做了小修改，已得到验证，特此感谢。

在部署过程中碰到了不少问题，在网上查到答案，多次得到了东北大学温占考老师在chinaunix上的解答，非常感谢，2004就注册了chinaunix，还是版主，太牛了。

部署还参考了中国科技大学和西南财经大学两家学校的使用说明：

中国科技大学：http://openvpn.ustc.edu.cn/

西南财经大学：http://info.swufe.edu.cn/vpn/openvpn/

本文作为抛砖引玉，希望能引出各路大神的奇技淫巧，各种好玩的用法。


----------

## 1 OpenVPN的优点 

- VPN即虚拟专用通道，是提供给企业之间或者个人与公司之间安全数据传输的隧道，OpenVPN无疑是Linux下开源VPN的先锋，提供了良好的性能和友好的用户GUI。
- OpenVPN能在Solaris、Linux、OpenBSD、FreeBSD、NetBSD、Mac OS X与Microsoft Windows以及Android和iOS上运行，并包含了许多安全性的功能。
- OpenVPN 是一个基于 OpenSSL 库的应用层 VPN 实现。和传统 VPN 相比，它的优点是简单易用。
- OpenVPN提供了多种身份验证方式，包括：预享私钥，第三方证书以及用户名/密码组合。预享密钥最为简单，但同时它只能用于建立点对点的VPN；
- 基于PKI的第三方证书提供了最完善的功能，但是需要额外的精力去维护一个PKI证书体系。 
- OpenVPN2.0后引入了用户名/口令组合的身份验证方式，它可以省略客户端证书，但是仍有一份服务器证书需要被用作加密。
- OpenVPN所有的通信都基于一个单一的IP端口，默认且推荐使用UDP协议通讯，同时TCP也被支持。
- OpenVPN连接能通过大多数的代理服务器，并且能够在NAT的环境中很好地工作。
- 服务端具有向客户端“推送”某些网络配置信息的功能，这些信息包括：IP地址、路由设置等。
- OpenVPN提供了两种虚拟网络接口：通用Tun/Tap驱动，通过它们，可以建立三层IP隧道，或者虚拟二层以太网，后者可以传送任何类型的二层以太网络数据。
- 传送的数据可通过LZO算法压缩。在选择协议时候，需要注意2个加密隧道之间的网络状况，如有高延迟或者丢包较多的情况下，请选择TCP协议作为底层协议，UDP协议由于存在无连接和重传机制，导致要隧道上层的协议进行重传，效率非常低下。
- IANA（Internet Assigned Numbers Authority）指定给OpenVPN的官方端口为1194。

## 2 OpenVPN的原理
OpenVpn的技术核心是虚拟网卡。

虚拟网卡是使用网络底层编程技术实现的一个驱动软件，安装后在主机上多出现一个网卡，可以像其它网卡一样进行配置。

服务程序可以在应用层打开虚拟网卡，如果应用软件（如IE）向虚拟网卡发送数据，则服务程序可以读取到该数据，如果服务程序写合适的数据到虚拟网卡，应用软件也可以接收得到。虚拟网卡在很多的操作系统下都有相应的实现，这也是OpenVpn能够跨平台一个很重要的理由。（建议采用tun模式，支持手机客户端）

在OpenVpn中，如果用户访问一个远程的虚拟地址（属于虚拟网卡配用的地址系列，区别于真实地址），则操作系统会通过路由机制将数据包（TUN模式）或数据帧（TAP模式）发送到虚拟网卡上，服务程序接收该数据并进行相应的处理后，通过SOCKET从外网上发送出去，远程服务程序通过SOCKET从外网上接收数据，并进行相应的处理后，发送给虚拟网卡，则应用软件可以接收到，完成了一个单向传输的过程，反之亦然。


## 3 部署环境
系统：CentOS 6.x  x64最小化安装

网络：单网卡

IP：202.119.191.11

## 4 安装openvpn ##

```
###4.1基础配置 ###
[root@vpn-ldap ~]# rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
[root@vpn-ldap ~]# sed -i 's@#b@b@g' /etc/yum.repos.d/epel.repo
[root@vpn-ldap ~]# sed  -i 's@mirrorlist@#mirrorlist@g' /etc/yum.repos.d/epel.repo
[root@vpn-ldap ~]# echo "*/10 * * * * /usr/sbin/ntpdate asia.pool.ntp.org  &>/dev/null" >/var/spool/cron/root
[root@vpn-ldap ~]# crontab -l
*/10 * * * * /usr/sbin/ntpdate asia.pool.ntp.org  &>/dev/null

###4.2安装openvpn ###
[root@vpn-ldap ~]# yum install openssl openssl-devel lzo  openvpn easy-rsa  -y

###4.3生成密钥和证书 ###

###修改vars文件信息 ###
[root@vpn-ldap ~]# cd /usr/share/easy-rsa/2.0/
[root@vpn-ldap 2.0]# vim vars 

###修改下面几项 ###

export KEY_COUNTRY="CN"
export KEY_PROVINCE="JS"
export KEY_CITY="NJ"
export KEY_ORG="CPU"
export KEY_EMAIL="sjn@cpu.edu.cn"
export KEY_OU="NOC"

###重新加载环境变量 ###
[root@vpn-ldap 2.0]# source vars

###清除所有证书和相关文件 ###
[root@vpn-ldap 2.0]# ./clean-all 

###生成新的根证书和根秘钥 ###
[root@vpn-ldap 2.0]# ./build-ca 

###给服务器端生成证书和秘钥 ###
[root@vpn-ldap 2.0]# ./build-key-server server

###给vpn客户端创建证书和秘钥，这里我们给client创建 ###
[root@vpn-ldap 2.0]# ./build-key client

###生成Diffie Hellman文件，生成过程可能有点慢，等待一会就好 ###
[root@vpn-ldap 2.0]# ./build-dh 

###生成ta.key文件（防DDos攻击、UDP淹没等恶意攻击） ###
[root@vpn-ldap 2.0]# openvpn --genkey --secret keys/ta.key

###在openvpn的配置目录下新建一个keys目录 ###
[root@vpn-ldap ~]# mkdir -p /etc/openvpn/keys
 
###将openvpn服务端需要用到的证书和秘钥复制到/etc/openvpn/keys目录下 ###
[root@vpn-ldap ~]# cp /usr/share/easy-rsa/2.0/keys/{ca.crt,server.{crt,key},dh2048.pem,ta.key} /etc/openvpn/keys/

```

# 基础篇：证书认证+默认网关

**导读：**

OpenVPN安装成功，相关证书也生成好了，下面就进行OpenVPN服务器端和客户端的配置。
配置先从最简单的证书认证开始，客户端所有流量走OpenVPN，最后为了移动端使用方便，再进行证书合并。

----------

##1 配置OpenVPN##

```
###1.1直接创建服务端配置文件server.conf ###

[root@vpn-ldap ~]# vim /etc/openvpn/server.conf

#修改openvpn的默认监听端口

;port 1194
port 51194

#使用UDP协议，速度快，防止握手攻击
;proto tcp
proto udp

#采用三层nat模式，在这种模式下openvpn server就相当于一台nat防火墙设备
;dev tap
dev tun

#验证客户端证书是否合法
ca keys/ca.crt
#server端使用的证书
cert keys/server.crt
key keys/server.key  # This file should be kept secret

#dh文件
dh keys/dh2048.pem

#防DDOS攻击，服务器端0,客户端1
tls-auth keys/ta.key 0

#LDAP认证，通过调用openvpn-auth-ldap.so进行LDAP认证
;plugin /usr/lib64/openvpn/plugin/lib/openvpn-auth-ldap.so "/etc/openvpn/auth/ldap.conf cn=%u"

#使用ldap认证，不需要客户端证书
;client-cert-not-required 
;username-as-common-name 

#设定server端虚拟出来的网段，设置给客户端虚拟局域网的网段
server 10.8.0.0 255.255.255.0

#防止Openvpn 重启后忘记client端曾经使用过的IP地址
ifconfig-pool-persist ipp.txt

#通过VPN Server往Client push路由，client通过pull指令获得Server push的所有选项并应用
;push "route 172.31.0.0 255.255.0.0"
;push "route 202.119.185.0 255.255.255.0"
;push "route 202.119.186.0 255.255.255.0"
;push "route 202.119.188.0 255.255.255.0"

#使Client的默认网关指向VPN，让Client的所有Traffic都通过VPN走
push "redirect-gateway def1 bypass-dhcp"

#给客户端push DNS
push "dhcp-option DNS 202.119.191.100"
push "dhcp-option DNS 202.119.191.101"

 
#Nat后面使用VPN，如果长时间不通信，NAT session 可能会失效，导致vpn连接丢失。所有keepalive提供一个类似ping的机制，每10秒通过vpn的control通道ping对方，如果120秒无法ping通，则认为丢失，并重启vpn,重新连接。
keepalive 10 120
 
#可以让vpn的client之间互相访问，直接通过openvpn程序转发
client-to-client
 
#允许多个客户端使用同一个证书连接服务端
duplicate-cn
 
#对数据进行压缩，注意server和client 一致
comp-lzo

#控制最大客户端数量
max-clients 20   

#以nobody用户运行，较安全
user nobody 
group nobody

#通过keepalive检测超时后，重新启动vpn，不重新读取keys,保留第一次使用的keys
persist-key
 
#通过keepalive检测超时后,重新启动vpn,一直保持tun或tap设备是linkup的，否则网络连接会先linkdown然后linkup
persist-tun
 
#定义了10小时之后需要重新验证key
reneg-sec 36000

#openvpn2.1以上版本一定要加此行 
script-security 3  

#把openvpn的状态写入日志中，短日志，每分钟刷新一次
status /var/log/openvpn/openvpn-status.log
 
#log日志，只保存一次启动的日志，每次启动之前都会清除这个文本
log   /var/log/openvpn/openvpn.log
 
#全部日志，每次启动的日志在这个文本中会追加，openvpn重启后会删除log内容，log-append则是追加log内容，并不删除。
log-append  /var/log/openvpn/openvpn.log
 
#日志记录级别
verb 3

#记录了当天登陆openvpn的用户名和时间
;client-connect /etc/openvpn/connect
;client-disconnect /etc/openvpn/disconnectt


###1.2创建日志目录###
[root@vpn-ldap ~]#mkdir -p /var/log/openvpn


###1.3启动openvpn服务并设置开机启动###

 [root@vpn-ldap ~]# chkconfig openvpn on
 [root@vpn-ldap ~]# service openvpn start
 Starting openvpn:  [  OK  ]
 [root@vpn-ldap ~]#netstat -lptun |grep vpn
 udp0  0 0.0.0.0:51194   0.0.0.0:*   6050/openvpn 
```



###2 路由转发###

### 2.1开启路由转发功能  ###

```
 [root@vpn-ldap ~]#vim /etc/sysctl.conf

 net.ipv4.ip_forward =1  //将net.ipv4.ip_forward = 0修改为net.ipv4.ip_forward = 1
```

### 2.2使sysctl.conf文件生效 ###
```
 [root@vpn-ldap ~]#sysctl -p
```

###3 iptables设置###

### 3.1 添加iptables转发规则,在防火墙配置文件添加如下nat ###

```
 [root@vpn-ldap ~]#iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE 
```
 
**说明：**10.8.0.0是vpn服务器提供的局域网，如果服务器是双网卡，那么eth0这块就要视情况变动了，例如有两个网卡，一个是外网的，一个是内网的，如果想连上vpn之后访问内网，这块就要填写内网的网卡

### 3.2 iptable开放51194端口 ###
```
 [root@vpn-ldap ~]#iptables -A INPUT -p udp --dport 51194 -j ACCEPT  //添加51194的UDP端口
```

### 3.3 保存iptables配置 ###
```
 [root@vpn-ldap ~]#service iptables save
```

### 3.4 查看iptables配置文件 ###
```
[root@vpn-ldap ~] /etc/sysconfig/iptables

 # Generated by iptables-save v1.4.7 on Wed Nov  1 10:45:43 2017

*nat
:PREROUTING ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
COMMIT
# Completed on Wed Nov  1 10:45:43 2017
# Generated by iptables-save v1.4.7 on Wed Nov  1 10:45:43 2017
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [4:480]
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -p udp -m state --state NEW -m udp --dport 161 -j ACCEPT
-A INPUT -p udp -m state --state NEW -m udp --dport 51194 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -i tun+ -j ACCEPT
COMMIT
# Completed on Wed Nov  1 10:45:43 2017
```


## 4 客户端配置文件 ##

### 4.1 创建客户端配置文件client.ovpn ###

```
client
dev tun
proto udp
#OpenVPN服务器的外网IP和端口，如果服务器有多条运营商线路，这里可以设置不同运营商的IP，创建多个配置文件
remote 202.119.191.11 51194
resolv-retry infinite
nobind
persist-key
persist-tun
ns-cert-type server

#ca ca.crt　
#cert client.crt　
#key client.key　
#tls-auth ta.key 1　

#ldap使用用户名密码认证
;auth-user-pass  

#定义了100小时之后需要重新验证key
reneg-sec 36000

comp-lzo

verb 3

<ca>
ca.crt文件内容
</ca>

key-direction 1
<tls-auth>
ta.key文件内容
</tls-auth>   

<cert>
client.crt文件内容
</cert>

<key>
client.key文件内容
</key>
```

### 4.2 合并证书到配置文件 ###
客户端需要的一共有5个文件：ca.crt、client.crt、client.key、ta.key(如果不开启tls-auth，则无需该文件)、client.ovpn。为了使移动端（安卓和苹果）顺利安装客户端并导入配置文件，需要将证书合并到配置文件中。
 如上客户端配置文件，将证书注释掉：

```
ca ca.crt　　改为：#ca ca.crt
cert client.crt　　改为：#cert client.crt
key client.key　　改为：#key client.key
tls-auth ta.key 1　　改为：#tls-auth ta.key 1
```

在最后面添加以下内容：
```
 <ca>
ca.crt文件内容
</ca>

key-direction 1
<tls-auth
ta.key文件内容
</tls-auth>   

<cert>
client.crt文件内容
</cert>

<key
client.key文件内容
</key>
```

#进阶篇：ldap认证+推送路由#

**导读：**

证书认证测试成功之后，如果想实现通过用户名和密码认证，就可以通过ldap或者radius实现相关对接了。

其实，OpenVPN的认证方式有多种，ldap的实现方式也有多种，可以通过OpenVPN自带的ldap插件、还可以通过脚本来实现。

这里选用最简单的方式，OpenVPN自带的ldap插件实现。

最后，再对OpenVPN的配置进行优化，OpenVPN向用户推送路由，解决用户所有流量都要绕OpenVPN的问题。

----------
##1 ldap认证##
ldap认证采用OpenVPN自带的ldap插件
###1.1 openvpn-auth-ldap 安装与配置 ###
```
[root@vpn-ldap ~]yum install -y openvpn-auth-ldap 
```

###1.2openvpn-auth-ldap 的配置 ###

```
[root@vpn-ldap ~]vim /etc/openvpn/auth/ldap.conf
```

对/etc/openvpn/auth/ldap.conf文件，修改部分内容如下： 

```
<LDAP>
	# LDAP server URL
	URL		ldap://202.119.191.12:389

	# Bind DN (If your LDAP server doesn't support anonymous binds)
	# BindDN		uid=Manager,ou=People,dc=example,dc=com

	# Bind Password
	# Password	SecretPassword

	# Network timeout (in seconds)
	Timeout		15

	# Enable Start TLS
	TLSEnable	no

	# Follow LDAP Referrals (anonymously)
	  FollowReferrals yes

	# TLS CA Certificate File
	# TLSCACertFile	/usr/local/etc/ssl/ca.pem

	# TLS CA Certificate Directory
	# TLSCACertDir	/etc/ssl/certs

	# Client Certificate and key
	# If TLS client authentication is required
	TLSCertFile	/usr/local/etc/ssl/client-cert.pem
	TLSKeyFile	/usr/local/etc/ssl/client-key.pem

	# Cipher Suite
	# The defaults are usually fine here
	# TLSCipherSuite	ALL:!ADH:@STRENGTH
</LDAP>

<Authorization>
	# Base DN
	BaseDN		"dc=cpu,dc=edu,dc=cn"

	# User Search Filter
	SearchFilter	"(uid=%u)"

	# Require Group Membership
	RequireGroup	false

	# Add non-group members to a PF table (disabled)
	#PFTable	ips_vpn_users

	#<Group>
		#BaseDN		"ou=Groups,dc=example,dc=com"
		#SearchFilter	"(|(cn=developers)(cn=artists))"
		#MemberAttribute	uniqueMember
		# Add group members to a PF table (disabled)
		#PFTable	ips_vpn_eng
	#</Group>
</Authorization>
```

**说明：**

由于我们的OpenVPN定位于内部运维，不对面向全校开放，所以没有与数字化校园的ldap服务器对接。
但是考虑到账号密码管理方便，利用Openldap又自建了一套ldap，专门用于内部测试系统的账号和密码的管理。
这里的ldap采用了匿名访问的方式，所以BindDN和Bind Password都注释掉了。

Openldap的部署可参照：https://www.cnblogs.com/lemon-le/p/6266921.html

### 2 服务器端server.conf的修改 ###

```
 [root@vpn-ldap ~]vim /etc/openvpn/server.conf
```
对server.conf文件，对部分内容进行修改如下： 

```
port 51194
proto udp
dev tun

ca keys/ca.crt
cert keys/server.crt
key keys/server.key  
dh keys/dh2048.pem  
tls-auth keys/ta.key 0

#LDAP认证，通过调用openvpn-auth-ldap.so进行LDAP认证
plugin /usr/lib64/openvpn/plugin/lib/openvpn-auth-ldap.so "/etc/openvpn/auth/ldap.conf cn=%u"

#使用ldap认证，不需要客户端证书
client-cert-not-required 
username-as-common-name 

server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt

#通过VPN Server往Client push路由，client通过pull指令获得Server push的所有选项并应用
push "route 172.31.0.0 255.255.0.0"
push "route 202.119.185.0 255.255.255.0"
push "route 202.119.186.0 255.255.255.0"
push "route 202.119.188.0 255.255.255.0"
push "route 202.119.191.0 255.255.255.0"
push "route 202.119.189.64 255.255.255.192"
push "route 202.119.189.192 255.255.255.192"
push "route 202.119.189.2 255.255.255.255"
push "route 202.119.189.3 255.255.255.255"
push "route 202.119.189.4 255.255.255.255"
push "route 202.119.189.5 255.255.255.255"
push "route 202.119.189.6 255.255.255.255"
push "route 202.119.189.7 255.255.255.255"
push "route 202.119.189.8 255.255.255.255"
push "route 192.168.199.123 255.255.255.255"
push "route 202.119.190.132 255.255.255.255"

#使Client的默认网关指向VPN，让Client的所有Traffic都通过VPN走
;push "redirect-gateway def1 bypass-dhcp"

#给客户端push DNS
push "dhcp-option DNS 202.119.191.100"
push "dhcp-option DNS 202.119.191.101"

keepalive 10 120
client-to-client
duplicate-cn
comp-lzo
max-clients 20   
user nobody 
group nobody
persist-key
persist-tun
reneg-sec 36000
script-security 3  

status /var/log/openvpn/openvpn-status.log
log   /var/log/openvpn/openvpn.log
log-append  /var/log/openvpn/openvpn.log
verb 3

#记录了当天登陆openvpn的用户名和时间
；client-connect /etc/openvpn/connect
；client-disconnect /etc/openvpn/disconnectt
```

## 3 客户端修改 ##

```
client
dev tun
proto udp
remote 202.119.191.11 51194
resolv-retry infinite
nobind
persist-key
persist-tun
ns-cert-type server

#ldap使用用户名密码认证
auth-user-pass  

reneg-sec 36000
comp-lzo
verb 3

<ca>
ca.crt文件内容
</ca>

key-direction 1
<tls-auth
ta.key文件内容
</tls-auth>   
```

## 4 向客户端推送路由 ##
如果客户端将OpenVPN作为默认网关，客户端的所有流量都会绕VPN访问，不是很合理。

所以OpenVPN要向客户端推送路由。如上server.conf配置。

将推送默认网关注释掉

```
;push "redirect-gateway def1 bypass-dhcp"
```

将push "route 172.31.0.0 255.255.0.0"等语句注释去掉

VPN Server会向Client push路由，client通过pull指令获得Server push的所有选项并应用



# 高级篇：记录用户的访问情况，并发邮件给管理员

**导读：**

ldap对接成功后，OpenVPN就实现了通过用户名和密码认证，管理员可以很方便的对用户进行管理。

但是管理员还想了解更多的信息，比如：谁什么时候登录的，又什么时候退出的，登录IP是多少等。

下面就介绍一下通过脚本来实现上述功能。

----------


## 1 记录OpenVPN用户访问日志 ##

```
## 1.1 记录用户拨入脚本 ##

建立/etc/openvpn/connect文件

vim etc/openvpn/connect

#!/bin/bash
day=`date +%F`
if [ -f /var/log/openvpn/log$day ];then
echo "`date '+%F %H:%M:%S'` User $common_name IP $trusted_ip is logged in" >>/var/log/openvpn/log$day
else
touch /var/log/openvpn/log$day
echo "`date '+%F %H:%M:%S'` User $common_name IP $trusted_ip is logged in" >>/var/log/openvpn/log$day
fi

## 1.2 记录用户退出脚本 ##

建立/etc/openvpn/disconnect文件

[root@vpn-ldap ~]vim /etc/openvpn/disconnect

#!/bin/bash
day=`date +%F`
if [ -f /var/log/openvpn/log$day ];then
echo "`date '+%F %H:%M:%S'` User $common_name IP $trusted_ip is logged off" >>/var/log/openvpn/log$day
else
touch /var/log/openvpn/log$day
echo "`date '+%F %H:%M:%S'` User $common_name IP $trusted_ip is logged off" >>/var/log/openvpn/log$day
fi

## 1.3 要将这两个脚本赋予执行权限 ##
[root@vpn-ldap ~] chmod +x /etc/openvpn/connect
[root@vpn-ldap ~] chmod +x /etc/openvpn/disconnect
```

**特别注意：**

因为openvpn是以nodody帐号在运行，因此必须赋予nodody帐号对/var/log/openvpn这个目录的写权限，否则openvpn的运行将受到影响，用户登录过程不能完成。

```bash
sudo chown -R nobody:nobody /var/log/openvpn
```

## 2.1 修改openvpn服务器配置文件，启用脚本 ##

修改/etc/openvpn/server.conf，将如下两行注释去掉

```
client-connect /etc/openvpn/connect
client-disconnect /etc/openvpn/disconnect
```

这样每天就会在/var/log/openvpn下面建立文件名为2017-12-27这样的文件，该文件记录了当天登陆openvpn的用户名和时间，输出例如：

```
2017-12-27 14:26:23 User shen IP 10.7.6.45 is logged off
2017-12-27 14:26:27 User shen IP 10.7.6.45 is logged in 
```

**注意**
因为openvpn本身判断机制的原因，登出时间比实际登出时间慢3分钟

## 3 每天将访问日志通过邮件定时发送给管理员 ##

### 3.1 安装sendmail与mail ###

### 3.1.1安装sendmail ###
```
[root@vpn-ldap ~]#yum -y install sendmail
```
###3.1.2 启动并设置开机启动 ###
```
[root@vpn-ldap ~]#service sendmail start
[root@vpn-ldap ~]#chkconfig --level 3 sendmail on
```
###3.1.3 安装mail ###
```
[root@vpn-ldap ~]#yum install -y mailx
```

### 3.2设置发件人信息 ###

邮件默认会使用linux当前登录用户信，通常会被当成垃圾邮件

所以这里指定发件人邮箱信息

```
[root@vpn-ldap ~]vim /etc/mail.rc

set from=noc@cpu.edu.cn
set smtp=smtphm.qiye.163.com
set smtp-auth-user=sjn@cpu.edu.cn
set smtp-auth-password=*******
set smtp-auth=login
```

**注意**
如果发件服务器开通了客户端授权码，则配置中的smtp-auth-password不是邮箱登录密码，是邮箱服务器开启smtp的授权码，每个邮箱开启授权码操作不同（网易126邮箱开启菜单：设置-> 客户端授权密码）。

### 3.3发信脚本 ###
### 3.3.1 创建发信脚本 ###
建立/etc/openvpn/mail2admin

```
[root@vpn-ldap ~]vim /etc/openvpn/mail2admin

#!/bin/bash
today=`date +%F`
if [ -f /var/log/openvpn/log$today ];then
mail -s “$today openvpn user access log” sjn@cpu.edu.cn < /var/log/openvpn/log$today
fi
```

###3.3.2 赋予执行权限 
```
[root@vpn-ldap ~]#chmod +x mail2admin
```

###4 建立cron工作表，每天定时23：55分发送当天的日志给管理员 ###
```
[root@vpn-ldap ~]#crontab -e
55 23 * * * /etc/openvpn/mail2admin
```

到这里，一切都搞定，每天晚上都会收到一封通知邮件

##未完待续......##

## 期待各路大神更精彩的使用方案...... ##




