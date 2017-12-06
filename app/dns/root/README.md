## [原创] 自己建立根DNS服务器

本文原创：**中国科学技术大学 张焕杰**

修改时间：2017.12.04

# 一、自己建立根DNS服务器的优点

自己建立根DNS服务器，可以减少查询的延迟，少受外部网络的影响。

# 二、建设根DNS的步骤

## 2.1 安装OS

我使用的是CentOS 6，其他系统也类似。安装`bind`软件作为DNS服务器。

设置好以太网口的IPv4/v6地址，能正常上网即可。

## 2.2 设置lo接口

将需要劫持的根DNS IP地址设置在 lo接口。我劫持了13个根DNS，使用如下命令
```
ip addr add 198.41.0.4/32 dev lo
ip addr add 2001:503:ba3e::2:30/128 dev lo
ip addr add 199.9.14.201/32 dev lo
ip addr add 2001:500:200::b/128 dev lo
ip addr add 192.33.4.12/32 dev lo
ip addr add 2001:500:2::c/128 dev lo
ip addr add 199.7.91.13/32 dev lo
ip addr add 2001:500:2d::d/128 dev lo
ip addr add 192.203.230.10/32 dev lo
ip addr add 2001:500:a8::e/128 dev lo
ip addr add 192.5.5.241/32 dev lo
ip addr add 2001:500:2f::f/128 dev lo
ip addr add 192.112.36.4/32 dev lo
ip addr add 2001:500:12::d0d/128 dev lo
ip addr add 198.97.190.53/32 dev lo
ip addr add 2001:500:1::53/128 dev lo
ip addr add 192.36.148.17/32 dev lo
ip addr add 2001:7fe::53/128 dev lo
ip addr add 192.58.128.30/32 dev lo
ip addr add 2001:503:c27::2:30/128 dev lo
ip addr add 193.0.14.129/32 dev lo
ip addr add 2001:7fd::1/128 dev lo
ip addr add 199.7.83.42/32 dev lo
ip addr add 2001:500:9f::42/128 dev lo
ip addr add 202.12.27.33/32 dev lo
ip addr add 2001:dc3::35/128 dev lo
```

## 2.3 设置bind

`vi /etc/named.conf`修改配置，仅仅修改了默认文件的 `listen-on listen-on-v6 allow-query recursion zone`等很少的地方，增加了zone root-servers.net

```
options {
        listen-on port 53 { any; };
        listen-on-v6 port 53 { any; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        allow-query     { any; };
        recursion no;

        dnssec-enable yes;
        dnssec-validation yes;

        /* Path to ISC DLV key */
        bindkeys-file "/etc/named.iscdlv.key";

        managed-keys-directory "/var/named/dynamic";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
        type master;
        file "root.zone";
};

zone "ROOT-SERVERS.NET." IN {
        type master;
        file "root-servers.net.zone";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

编辑 /var/named/root-servers.net.zone文件(之前我没有加这个域，会有问题，原因可以参见 [这里](https://yeti-dns.org/documents.html)的Root Server Glue and BIND9)，内容为：
```
$TTL 1D
@       IN SOA  @ fake.root. (
                                        1       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        IN      NS      a.root-servers.net.
        IN      NS      b.root-servers.net.
        IN      NS      c.root-servers.net.
        IN      NS      d.root-servers.net.
        IN      NS      e.root-servers.net.
        IN      NS      f.root-servers.net.
        IN      NS      g.root-servers.net.
        IN      NS      h.root-servers.net.
        IN      NS      i.root-servers.net.
        IN      NS      j.root-servers.net.
        IN      NS      k.root-servers.net.
        IN      NS      l.root-servers.net.
        IN      NS      m.root-servers.net.

A.ROOT-SERVERS.NET.      A     198.41.0.4
A.ROOT-SERVERS.NET.      AAAA  2001:503:ba3e::2:30
B.ROOT-SERVERS.NET.      A     199.9.14.201
B.ROOT-SERVERS.NET.      AAAA  2001:500:200::b
C.ROOT-SERVERS.NET.      A     192.33.4.12
C.ROOT-SERVERS.NET.      AAAA  2001:500:2::c
D.ROOT-SERVERS.NET.      A     199.7.91.13
D.ROOT-SERVERS.NET.      AAAA  2001:500:2d::d
E.ROOT-SERVERS.NET.      A     192.203.230.10
E.ROOT-SERVERS.NET.      AAAA  2001:500:a8::e
F.ROOT-SERVERS.NET.      A     192.5.5.241
F.ROOT-SERVERS.NET.      AAAA  2001:500:2f::f
G.ROOT-SERVERS.NET.      A     192.112.36.4
G.ROOT-SERVERS.NET.      AAAA  2001:500:12::d0d
H.ROOT-SERVERS.NET.      A     198.97.190.53
H.ROOT-SERVERS.NET.      AAAA  2001:500:1::53
I.ROOT-SERVERS.NET.      A     192.36.148.17
I.ROOT-SERVERS.NET.      AAAA  2001:7fe::53
J.ROOT-SERVERS.NET.      A     192.58.128.30
J.ROOT-SERVERS.NET.      AAAA  2001:503:c27::2:30
K.ROOT-SERVERS.NET.      A     193.0.14.129
K.ROOT-SERVERS.NET.      AAAA  2001:7fd::1
L.ROOT-SERVERS.NET.      A     199.7.83.42
L.ROOT-SERVERS.NET.      AAAA  2001:500:9f::42
M.ROOT-SERVERS.NET.      A     202.12.27.33
M.ROOT-SERVERS.NET.      AAAA  2001:dc3::35
```
注意这个文件需要对named可读。


## 2.4 下载 root.zone 文件

```
cd /var/named
wget http://www.internic.net/domain/root.zone
```

## 2.5 允许53端口

使用iptables对外开放udp/tcp 53端口，注意IPv4/IPv6的都要允许。

## 2.6 启动named服务，测试

```
service named start

dig @127.0.0.1 com. ns
```
延迟应该是0ms


# 三、劫持路由

劫持路由可以使用静态路由，也可以使用BGP动态路由。

以静态路由为例，只要在交换机上把13个根DNS IP的路由指向该Linux服务器即可。

我使用ExaBGP动态劫持，exabgp.conf配置为
```
neighbor 202.38.64.126 {
        local-address 202.38.64.12;
        peer-as 45081;
        local-as 65500;
        router-id 202.38.64.12;
        static {
                route 198.41.0.4/32 next-hop 202.38.64.12;
                route 2001:503:ba3e::2:30/128 next-hop 2001:da8:d800::12;
                route 199.9.14.201/32 next-hop 202.38.64.12;
                route 2001:500:200::b/128 next-hop 2001:da8:d800::12;
                route 192.33.4.12/32 next-hop 202.38.64.12;
                route 2001:500:2::c/128 next-hop 2001:da8:d800::12;
                route 199.7.91.13/32 next-hop 202.38.64.12;
                route 2001:500:2d::d/128 next-hop 2001:da8:d800::12;
                route 192.203.230.10/32 next-hop 202.38.64.12;
                route 2001:500:a8::e/128 next-hop 2001:da8:d800::12;
                route 192.5.5.241/32 next-hop 202.38.64.12;
                route 2001:500:2f::f/128 next-hop 2001:da8:d800::12;
                route 192.112.36.4/32 next-hop 202.38.64.12;
                route 2001:500:12::d0d/128 next-hop 2001:da8:d800::12;
                route 198.97.190.53/32 next-hop 202.38.64.12;
                route 2001:500:1::53/128 next-hop 2001:da8:d800::12;
                route 192.36.148.17/32 next-hop 202.38.64.12;
                route 2001:7fe::53/128 next-hop 2001:da8:d800::12;
                route 192.58.128.30/32 next-hop 202.38.64.12;
                route 2001:503:c27::2:30/128 next-hop 2001:da8:d800::12;
                route 193.0.14.129/32 next-hop 202.38.64.12;
                route 2001:7fd::1/128 next-hop 2001:da8:d800::12;
                route 199.7.83.42/32 next-hop 202.38.64.12;
                route 2001:500:9f::42/128 next-hop 2001:da8:d800::12;
                route 202.12.27.33/32 next-hop 202.38.64.12;
                route 2001:dc3::35/128 next-hop 2001:da8:d800::12;
        }
}
```

# 四、从其他DNS测试

从其他DNS服务器（当然是路由受劫持影响的）上测试:
```
ping a.root-servers.net.

dig @a.root-servers.net 
```
延迟应该为0ms

# 五、优化措施

根DNS服务器中，CERNET已经对除了h.root-servers.net之外的做了镜像（劫持），所以我们仅仅对
以下14个IP地址进行劫持
```
ip addr add 198.97.190.53/32 dev lo
ip addr add 2001:503:ba3e::2:30/128 dev lo
ip addr add 2001:500:200::b/128 dev lo
ip addr add 2001:500:2::c/128 dev lo
ip addr add 2001:500:2d::d/128 dev lo
ip addr add 2001:500:a8::e/128 dev lo
ip addr add 2001:500:2f::f/128 dev lo
ip addr add 2001:500:12::d0d/128 dev lo
ip addr add 2001:500:1::53/128 dev lo
ip addr add 2001:7fe::53/128 dev lo
ip addr add 2001:503:c27::2:30/128 dev lo
ip addr add 2001:7fd::1/128 dev lo
ip addr add 2001:500:9f::42/128 dev lo
ip addr add 2001:dc3::35/128 dev lo

```
为了能在named进程正常时发布路由，异常时自动撤回路由，增加了脚本
`/var/named/healthcheck.sh`

```
#!/bin/bash

status=0

while true; do
        dig +time=1 @127.0.0.1 a.root-servers.net > /dev/null 2>/dev/null
        newstatus=$?
        if [ $newstatus -eq 0 ];  then
                newstatus=1
        else
                newstatus=0
        fi

#       echo status $status newstatus $newstatus

        if [ $status -eq 0 ] ; then
                if [ $newstatus -eq 1 ]; then   # service up
                        status=1
echo announce route 198.97.190.53/32 next-hop 202.38.64.12
echo announce route 2001:503:ba3e::2:30/128 next-hop 2001:da8:d800::12
echo announce route 2001:500:200::b/128 next-hop 2001:da8:d800::12
echo announce route 2001:500:2::c/128 next-hop 2001:da8:d800::12
echo announce route 2001:500:2d::d/128 next-hop 2001:da8:d800::12
echo announce route 2001:500:a8::e/128 next-hop 2001:da8:d800::12
echo announce route 2001:500:2f::f/128 next-hop 2001:da8:d800::12
echo announce route 2001:500:12::d0d/128 next-hop 2001:da8:d800::12
echo announce route 2001:500:1::53/128 next-hop 2001:da8:d800::12
echo announce route 2001:7fe::53/128 next-hop 2001:da8:d800::12
echo announce route 2001:503:c27::2:30/128 next-hop 2001:da8:d800::12
echo announce route 2001:7fd::1/128 next-hop 2001:da8:d800::12
echo announce route 2001:500:9f::42/128 next-hop 2001:da8:d800::12
echo announce route 2001:dc3::35/128 next-hop 2001:da8:d800::12
                fi
        fi
        if [ $status -eq 1 ] ; then
                if [ $newstatus -eq 0 ]; then   # service down
                        status=0
echo withdraw route 198.97.190.53/32
echo withdraw route 2001:503:ba3e::2:30/128
echo withdraw route 2001:500:200::b/128
echo withdraw route 2001:500:2::c/128
echo withdraw route 2001:500:2d::d/128
echo withdraw route 2001:500:a8::e/128
echo withdraw route 2001:500:2f::f/128
echo withdraw route 2001:500:12::d0d/128
echo withdraw route 2001:500:1::53/128
echo withdraw route 2001:7fe::53/128
echo withdraw route 2001:503:c27::2:30/128
echo withdraw route 2001:7fd::1/128
echo withdraw route 2001:500:9f::42/128
echo withdraw route 2001:dc3::35/128
                fi
        fi

        sleep 1
done
```

然后把 `/etc/exabgp.conf`修改为

```
neighbor 202.38.64.126 {
        local-address 202.38.64.12;
        peer-as 45081;
        local-as 65500;
        router-id 202.38.64.12;
        process service-dns {
                run /var/named/healthcheck.sh;
        }
}

```

# 六、后续更新
定期下载root.zone文件，有更新时重启named服务即可。

建立 `/var/named/Makefile`:
```
all:
        wget -O root.zone http://www.internic.net/domain/root.zone
        chgrp named *
        /sbin/service named configtest
        /sbin/service named restart
        git commit -a -m `date +"%F_%T"`
```

增加如下crontabl
```
15 * * * * cd /var/named && make
```

***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
