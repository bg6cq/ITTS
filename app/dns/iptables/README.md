## [原创] DNS服务器的iptables规则

本文原创：**中国科学技术大学 张焕杰**

修改时间：2018.03.19

本文给出我校对内提供DNS服务的iptables规则和简单说明


```
#!/bin/sh

iptables -F

iptables -N dnsin
iptables -N dnsout

#对收到的包，记录最近10秒钟的IP, 限制每个IP每秒钟20个查询
iptables -A dnsin -m hashlimit --hashlimit-name dns --hashlimit 20/sec --hashlimit-burst 10 \
	--hashlimit-mode srcip --hashlimit-htable-expire 10000 -j ACCEPT
iptables -A dnsin -j DROP

#对决绝查询应答包限速每秒10个
iptables -A dnsout -j ACCEPT -m limit --limit 10/sec
iptables -A dnsout -j DROP

iptables -I INPUT -j dnsin -p udp --dport 53

#对拒绝查询的应答限速
iptables -A OUTPUT -j dnsout -m u32 -p udp --sport 53 --u32 "28&0xFFFF=0x8105"

#禁止大包应答
iptables -A OUTPUT -j LOG -p udp --sport 53 -m length --length 1300:  -m limit --limit 10/sec

```

***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
