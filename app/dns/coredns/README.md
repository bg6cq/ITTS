## [原创] CoreDNS 尝试

本文原创：**中国科学技术大学 张焕杰**

修改时间：2020.02.18

Core DNS是一个go语言开发的DNS服务器，主要用来提供权威解析服务。详细请见 https://github.com/coredns/coredns

本次尝试使用Core DNS软件搭建ah.edu.cn域的DNS权威解析服务器，从以下步骤可以看到针对一个独立域名的配置，比bind要简单很多。

Core DNS支持很多plugin，本文仅仅使用最简单的file、errors、log。

Core DNS还不直接支持view等功能。

## 一、CentOS 7 安装

CentOS 7 安装后，更新并安装常用软件
```
yum update -y
yum install -y wget bind-utils net-tools git
```

## 二、CoreDNS安装

到 https://github.com/coredns/coredns/releases 下载，我下载的命令:
```
wget https://github.com/coredns/coredns/releases/download/v1.6.7/coredns_1.6.7_linux_amd64.tgz
tar xzf coredns_1.6.7_linux_amd64.tgz
mv coredns /usr/local/bin
```

## 三、CoreDNS配置

3.1 创建用户

```
useradd coredns -s /sbin/nologin
```

3.2 编辑/etc/coredns/Corefile

```
mkdir /etc/coredns

vi /etc/coredns/Corefile
```
Corefile内容如下：

```
ah.edu.cn {
    file /etc/coredns/ah.edu.cn.db
    errors
    log
} 

224.45.210.in-addr.arpa {
    file /etc/coredns/210.45.224.db
    errors
    log
}
```

3.3 编辑 /etc/coredns/ah.edu.cn.db 

vi /etc/coredns/ah.edu.cn.db 文件是bind格式，内容如下：
```
$TTL	600

@	IN	SOA	ns.ah.edu.cn. root.ustc.edu.cn. (
	116 600 1600 360000 800 )
	IN	NS	ns.ah.edu.cn.
	IN	NS	ns2.ah.edu.cn.
	IN	MX	0	smtp.ustc.edu.cn.
ns		IN	A	210.45.224.1
ns2		IN	A	210.45.224.2

www		IN	A	218.22.21.21

```

3.4 编辑 /etc/coredns/210.45.224.db 

vi /etc/coredns/210.45.224.db 文件是bind格式，内容如下：
```
$TTL	600

@	IN	SOA	ns.ah.edu.cn. root.ustc.edu.cn. (
	87 600 1600 360000 800 )
	IN	NS	ns.ah.edu.cn.
	IN	NS	ns2.ah.edu.cn.

1	IN	PTR	ns.ah.edu.cn.
2	IN	PTR	ns2.ah.edu.cn.
```


3.5 编辑/usr/lib/systemd/system/coredns.service

vi /usr/lib/systemd/system/coredns.service 内容如下
```
[Unit]
Description=CoreDNS DNS server
Documentation=https://coredns.io
After=network.target

[Service]
PermissionsStartOnly=true
LimitNOFILE=1048576
LimitNPROC=512
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE
NoNewPrivileges=true
User=coredns
ExecStart=/usr/local/bin/coredns -conf=/etc/coredns/Corefile
ExecReload=/bin/kill -SIGUSR1 $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

3.6 启动CoreDNS

```
systemctl enable coredns
systemctl start coredns
systemctl status coredns
```

3.7 允许53端口

```
firewall-cmd --permanent --add-service=dns
firewall-cmd --reload
```

注意：上述配置，DNS查询日志记录在/var/log/message 


***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
