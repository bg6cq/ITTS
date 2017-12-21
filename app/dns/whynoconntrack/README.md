## [原创] 为何DNS服务器要禁用连接跟踪

本文原创：**中国科学技术大学 张焕杰**

修改时间：2017.12.21

# 一、什么是连接跟踪

连接跟踪(conntrack)是Linux kernel中的一个功能，可以记录通过Linux kernel处理的数据包的连接信息。

启动连接跟踪后，每个连接的第一个数据包经过Linux时，会在连接表中增加一个表项。如果一段时间后没有这
个连接的数据包，连接表项会超时删除。这些处理不光会消耗CPU处理，而且由于默认的连接表项并
不多，当连接表项用光后，会出现丢包现象。

如果Linux开启了NAT功能或状态防火墙（iptables 中使用state模块），连接跟踪必须要启动，其他情况下可以禁用。

[这里](http://www.10tiao.com/html/488/201701/2247484116/1.html) 有个稍微详细的说明，有兴趣可以去看看。

# 二、为何DNS服务器与连接跟踪关系密切

DNS是轻量级服务，几乎每个数据包都是独立的新连接，因此启用连接跟踪时，
会占用大量的连接表项，白白浪费CPU时间，并且连接表满了后，导致丢包，引起DNS查询异常。

如果`dmesg`看到诸如`table full, dropping packet`，说明连接表已经满了。

# 三、DNS服务器可以不用连接跟踪吗

当然可以不用。由于DNS服务器开启的tcp/udp端口较少，不使用连接跟踪也照样可以很安全。

比如我们的DNS，`netstat -anp`查看服务器进程，仅仅在tcp/udp 53和tcp 22有监听，
因此iptables增加有对tcp 22端口限制的规则，其他都允许：
```
#!/bin/sh
iptables -F
iptables -A INPUT -j ACCEPT -s 202.38.64.x -p tcp --dport 22
iptables -A INPUT -j DROP -p tcp --dport 22
```

# 四、如果不得不启用连接跟踪，有解决办法吗

1. 增加连接表项

加载nf_conntrack时增加参数hashsize，最大连接数是hashsize*8。
每个连接数大约占用200多个字节，可以根据自己的内存算一算最多能支持多少。
```
# 加载连接跟踪模块，设置最大连接数为40*8=320万
modprobe nf_conntrack hashsize=400000
```

2. 修改超时时间

可以修改连接跟踪的超时时间，让连接尽快删除。对于UDP，完全可以设置超时时间5秒。


***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
