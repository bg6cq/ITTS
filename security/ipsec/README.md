## [原创]使用ipsec加密Linux主机间通信

本文原创：**中国科学技术大学 张焕杰**

修改时间：2017.11.21

## 一、ipsec介绍

ipsec是工作在IP层的安全协议，本文介绍使用ipsec协议来加密Linux主机间的IP通信。

本文仅仅介绍简单的静态密钥设置，并使用加密分组流的封装安全载荷协议（ESP协议）和认证头协议（AH协议）协议来保证数据的机密性、来源可靠性（认证）、无连接的完整性并提供抗重播服务。

使用ipsec加密后，两台主机间的通信对第三方完全保密，并且不受可能的ARP劫持/TCP劫持等攻击。

我们计划在反向代理服务器和校内的Linux主机间逐步采用ipsec加密，以彻底避免校内服务器被TCP连接劫持。

## 二、使用的ipsec实现

ipsec在Linux下有若干实现，本文介绍的是kernel 2.5.49引入的native实现，并不是通常的FeeS/WAN。

在CentOS 6，需要安装epel中ipsec-tools包。
```
yum install ipsec-tools
```

## 三、静态密钥配置

假定2台Linux主机，IP分别是202.141.176.2和202.141.176.3，需要使用ipsec安全通信，在
202.141.176.2主机的配置如下：

ipsec.sh
````
#!/sbin/setkey -f
#202.141.176.2配置
flush;
spdflush;

# AH 定义认证配置
add 202.141.176.2 202.141.176.3 ah 15700 -A hmac-md5 "1234567890123456";
add 202.141.176.3 202.141.176.2 ah 24500 -A hmac-md5 "1234567890123456";

# ESP 定义加密配置
add 202.141.176.2 202.141.176.3 esp 15701 -E 3des-cbc "123456789012123456789012";
add 202.141.176.3 202.141.176.2 esp 24501 -E 3des-cbc "123456789012123456789012";

# 定义安全策略
spdadd 202.141.176.2 202.141.176.3 any -P out ipsec
           esp/transport//require
           ah/transport//require;

spdadd 202.141.176.3 202.141.176.2 any -P in ipsec
           esp/transport//require
           ah/transport//require;
````

202.141.176.3主机的配置如下：

ipsec.sh
````
#!/sbin/setkey -f
#202.141.176.3配置
flush;
spdflush;

# AH 定义认证配置
add 202.141.176.2 202.141.176.3 ah 15700 -A hmac-md5 "1234567890123456";
add 202.141.176.3 202.141.176.2 ah 24500 -A hmac-md5 "1234567890123456";

# ESP 定义加密配置
add 202.141.176.2 202.141.176.3 esp 15701 -E 3des-cbc "123456789012123456789012";
add 202.141.176.3 202.141.176.2 esp 24501 -E 3des-cbc "123456789012123456789012";

# 定义安全策略
spdadd 202.141.176.3 202.141.176.2 any -P out ipsec
           esp/transport//require
           ah/transport//require;

spdadd 202.141.176.2 202.141.176.3 any -P in ipsec
           esp/transport//require
           ah/transport//require;
````

从以上设置可以看到，两台主机的AH和ESP配置是完全相同的，安全策略处 in/out是相反的。

在2台Linux主机上，分别执行以上命令即可完成设置。

## 四、iptables有关注意事项

使用ipsec加密数据包时，每个数据包会两次通过iptables过滤规则。如果同时使用了AH/ESP，第一次经过iptables过滤时看到的协议是最外层的AH。

如在202.141.176.3上，接收到来自202.141.176.2发送的`ping 202.141.176.3`产生的数据包，第一次经过iptables过滤时，看到的协议是最外层的AH；
第二次经过iptables过滤时，数据包才是解密后的icmp数据包。因此设置以下规则允许AH数据包经过。
```
iptables -I INPUT -j ACCEPT -p ah
```

## 五、验证与测试

使用setkey -D 可以查看设置的AH/ESP加密配置
````
# setkey -D
202.141.176.3 202.141.176.2
        esp mode=transport spi=24501(0x00005fb5) reqid=0(0x00000000)
        E: 3des-cbc  31323334 35363738 39303132 31323334 35363738 39303132
        seq=0x00000000 replay=0 flags=0x00000000 state=mature
        created: Nov 21 06:15:17 2017   current: Nov 21 07:07:10 2017
        diff: 3113(s)   hard: 0(s)      soft: 0(s)
        last: Nov 21 06:15:34 2017      hard: 0(s)      soft: 0(s)
        current: 1123603(bytes) hard: 0(bytes)  soft: 0(bytes)
        allocated: 1413 hard: 0 soft: 0
        sadb_seq=1 pid=30211 refcnt=0
202.141.176.2 202.141.176.3
        esp mode=transport spi=15701(0x00003d55) reqid=0(0x00000000)
        E: 3des-cbc  31323334 35363738 39303132 31323334 35363738 39303132
        seq=0x00000000 replay=0 flags=0x00000000 state=mature
        created: Nov 21 06:15:17 2017   current: Nov 21 07:07:10 2017
        diff: 3113(s)   hard: 0(s)      soft: 0(s)
        last: Nov 21 06:21:35 2017      hard: 0(s)      soft: 0(s)
        current: 36631(bytes)   hard: 0(bytes)  soft: 0(bytes)
        allocated: 629  hard: 0 soft: 0
        sadb_seq=2 pid=30211 refcnt=0
202.141.176.3 202.141.176.2
        ah mode=transport spi=24500(0x00005fb4) reqid=0(0x00000000)
        A: hmac-md5  31323334 35363738 39303132 33343536
        seq=0x00000000 replay=0 flags=0x00000000 state=mature
        created: Nov 21 06:15:17 2017   current: Nov 21 07:07:10 2017
        diff: 3113(s)   hard: 0(s)      soft: 0(s)
        last: Nov 21 06:15:34 2017      hard: 0(s)      soft: 0(s)
        current: 1155520(bytes) hard: 0(bytes)  soft: 0(bytes)
        allocated: 1413 hard: 0 soft: 0
        sadb_seq=3 pid=30211 refcnt=0
202.141.176.2 202.141.176.3
        ah mode=transport spi=15700(0x00003d54) reqid=0(0x00000000)
        A: hmac-md5  31323334 35363738 39303132 33343536
        seq=0x00000000 replay=0 flags=0x00000000 state=mature
        created: Nov 21 06:15:17 2017   current: Nov 21 07:07:10 2017
        diff: 3113(s)   hard: 0(s)      soft: 0(s)
        last: Nov 21 06:21:35 2017      hard: 0(s)      soft: 0(s)
        current: 51096(bytes)   hard: 0(bytes)  soft: 0(bytes)
        allocated: 629  hard: 0 soft: 0
        sadb_seq=0 pid=30211 refcnt=0
````
使用setkey -DP 可以查看设置的安全策略配置
````
# setkey -DP
202.141.176.3[any] 202.141.176.2[any] 255
        fwd prio def ipsec
        esp/transport//require
        ah/transport//require
        created: Nov 21 08:42:43 2017  lastused:
        lifetime: 0(s) validtime: 0(s)
        spid=90 seq=1 pid=28450
        refcnt=1
202.141.176.3[any] 202.141.176.2[any] 255
        in prio def ipsec
        esp/transport//require
        ah/transport//require
        created: Nov 21 08:42:43 2017  lastused:
        lifetime: 0(s) validtime: 0(s)
        spid=80 seq=2 pid=28450
        refcnt=1
202.141.176.2[any] 202.141.176.3[any] 255
        out prio def ipsec
        esp/transport//require
        ah/transport//require
        created: Nov 21 08:42:43 2017  lastused:
        lifetime: 0(s) validtime: 0(s)
        spid=73 seq=0 pid=28450
        refcnt=1
````
		
使用tcpdump抓包可以验证2台Linux主机间的通信是经过加密的。
````
#ping 202.141.176.3
PING 202.141.176.3 (202.141.176.3) 56(84) bytes of data.
64 bytes from 202.141.176.3: icmp_seq=1 ttl=64 time=0.553 ms
64 bytes from 202.141.176.3: icmp_seq=2 ttl=64 time=0.290 ms

# tcpdump  -i eth0 -nn -c 100 host 202.141.176.3 &
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
09:08:22.323311 IP 202.141.176.2 > 202.141.176.3: AH(spi=0x00003d54,seq=0x1): ESP(spi=0x00003d55,seq=0x1), length 88
09:08:22.323767 IP 202.141.176.3 > 202.141.176.2: AH(spi=0x00005fb4,seq=0x586): ESP(spi=0x00005fb5,seq=0x586), length 88
09:08:23.323527 IP 202.141.176.2 > 202.141.176.3: AH(spi=0x00003d54,seq=0x2): ESP(spi=0x00003d55,seq=0x2), length 88
09:08:23.323765 IP 202.141.176.3 > 202.141.176.2: AH(spi=0x00005fb4,seq=0x587): ESP(spi=0x00005fb5,seq=0x587), length 88
````

## 六、参考资料

[IPSEC: secure IP over the Internet](http://lartc.org/howto/lartc.ipsec.html)


***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
