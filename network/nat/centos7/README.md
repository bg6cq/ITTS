## [原创]使用CentOS 7做NAT设备

本文原创：**中国科学技术大学 张焕杰**

修改时间：2017.11.26

本文介绍使用CentOS 7用作NAT设备的安装和配置过程。

## 一、系统安装

安装CentOS 7系统，选择最小安装即可。

安装后执行以下命令更新和安装必备软件。

```
yum -y update

yum -y install git gcc libnetfilter_conntrack libnetfilter_queue libmnl gd httpd screen \
tcpdump ntpdate conntrack-tools gd-devel net-tools bind-utils telnet

```

## 二、禁用SELinux、禁止系统防火墙、启用httpd服务

1. 禁用SELinux

`vi /etc/sysconfig/selinux`把其中的`SELINUX=enforcing`修改为`SELINUX=disabled`

2. 禁止系统防火墙、启用httpd服务

```
systemctl stop firewalld.service
systemctl disable firewalld.service
systemctl enable httpd.service
```

## 三、相关配置文件

为了方便，不使用系统来管理网卡的配置，而完全由脚本设置。

因此首先编辑网卡配置文件，设置为启动时不启用网卡。我的系统默认网卡是em1，因此编辑文件`/etc/sysconfig/network-scripts/ifcfg-em1`，将其中的`ONBOOT=yes`修改为`ONBOOT=no`。

编辑文件`rc.local`和`rc.firewall`：

`vi /etc/rc.d/rc.local`


````
#!/bin/sh

ip link set p1p1 up
ip link set p1p2 up

# 这是对外的网卡IP地址和网关
ip addr add 202.141.176.10/25 dev p1p1
ip addr add 202.141.176.9/25 dev p1p1
ip route add 0/0 via 202.141.176.126

# 这是对内的网卡IP地址和静态路由
ip addr add 202.38.96.208/26 dev p1p2
ip route add 202.38.64.0/19 via 202.38.96.194
ip route add 210.45.64.0/20 via 202.38.96.194
ip route add 210.45.112.0/20 via 202.38.96.194
ip route add 211.86.144.0/20 via 202.38.96.194
ip route add 222.195.64.0/19 via 202.38.96.194
ip route add 114.214.160.0/19 via 202.38.96.194
ip route add 114.214.192.0/19 via 202.38.96.194

# 启用IP路由转发
echo 1 > /proc/sys/net/ipv4/ip_forward

# 加载连接跟踪模块，设置最大连接数为40*8=320万
modprobe nf_conntrack hashsize=400000
modprobe nf_conntrack_ipv4
modprobe nf_conntrack_ftp
modprobe iptable_nat
modprobe ip_nat_ftp
modprobe nf_conntrack_pptp

# 调整默认的超时时间
cd /proc/sys/net/netfilter
echo 1 > nf_conntrack_acct 
echo 60 > nf_conntrack_generic_timeout
echo 10 > nf_conntrack_icmp_timeout
echo 10 > nf_conntrack_tcp_timeout_close_wait
echo 900 > nf_conntrack_tcp_timeout_established
echo 10 > nf_conntrack_tcp_timeout_fin_wait
echo 10 > nf_conntrack_tcp_timeout_last_ack
echo 10 > nf_conntrack_tcp_timeout_syn_recv
echo 10 > nf_conntrack_tcp_timeout_syn_sent
echo 10 > nf_conntrack_tcp_timeout_time_wait

/etc/rc.d/rc.firewall

/usr/src/traffic/iftrafficd &

/usr/sbin/conntrack -U | wc

screen -d -m /usr/src/natlog/run  &
````

`vi /etc/rc.d/rc.firewall`

````
#!/bin/sh

iptables -F
iptables -t nat -F

#iptables -t nat -A POSTROUTING -j SNAT -o eth0 --to 202.141.176.10
iptables -t nat -A POSTROUTING -j SNAT -o p1p1 --to 202.141.176.9

iptables -A FORWARD -j DROP -p udp --dport 135:139
iptables -A FORWARD -j DROP -p tcp --dport 135:139
iptables -A FORWARD -j DROP -p tcp --dport 445

iptables -A INPUT -j ACCEPT -i lo
iptables -A INPUT -j ACCEPT -p icmp
iptables -A INPUT -j ACCEPT -p tcp --dport 80 -s 202.38.64.0/18
iptables -A INPUT -j ACCEPT -p tcp --dport 80 -s 211.86.158.0/23
iptables -A INPUT -j ACCEPT -p tcp --dport 80 -s 210.45.0.0/16
iptables -A INPUT -j DROP -p tcp --dport 80
iptables -A INPUT -j ACCEPT -m state --state ESTABLISHED
iptables -A INPUT -j ACCEPT -p tcp --dport 22 -s 202.38.64.0/24
iptables -A INPUT -j ACCEPT -p tcp --dport 22 -s 202.38.96.0/24
iptables -A INPUT -j DROP  

````
将这两个文件设置为可执行
```
chmod u+x /etc/rc.d/rc.local /etc/rc.d/rc.firewall
```

## 四、natlog配置

`natlog`使用conntrack把连接终止时的NAT信息保存成文本文件，放在/home/natlog目录下。

安装:

```
cd /usr/src/
git clone https://github.com/bg6cq/natlog.git
cd natlog
make
mkdir /home/natlog
```

## 五、traffic配置

traffic监视用于快速监视网卡的流量，每秒钟更新。

安装：

```
cd /usr/src/
git clone https://github.com/bg6cq/traffic.git
cd traffic
make

vi traffic.html 根据自己网卡名字，修改
cp traffic.html /var/www/html/index.html
```

## 六、调试和功能验证

1. 重启CentOS 7服务器后，使用root登录，使用以下命令查看状态是否正常

```
ip addr
ip route
iptables -t nat -L -nv
iptables -L -nv
conntrack -L
ls -al /home/natlog
```

## 七、注意事项

/home/natlog 中每天产生一个NAT日志文件，请注意磁盘空间的占用。

可以增加如下的crontab自动清理200天的文件：

```
0 5 * * * find /home/natlog -maxdepth 1 -mtime +200 -name "2*gz" -exec  rm -rf {} \;
```

***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
