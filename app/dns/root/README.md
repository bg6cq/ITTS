## [原创] 自己建立根DNS服务器

本文原创：**中国科学技术大学 张焕杰**

修改时间：2017.12.04

# 一、自己建立根DNS服务器的优点

自己建立根DNS服务器，可以减少查询的延迟，不受外部的影响。

# 二、建设根DNS的步骤

## 2.1 安装OS

我使用的是CentOS 6，其他系统也类似。安装`bind`软件作为DNS服务器。

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

`vi /etc/named.conf`修改配置，仅仅修改了默认文件的 `listen-on listen-on-v6 allow-query recursion zone`等很少的地方

```
options {
        listen-on port 53 { any; };
        listen-on-v6 port 53 { 0::0; };
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

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

## 2.4 下载 root.zone 文件

```
cd /var/named
wget http://www.internic.net/domain/root.zone
```

## 2.5 允许53端口

使用iptables对外开放udp/tcp 53端口

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

***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
