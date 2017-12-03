## [原创] 中国大陆根DNS服务器的奥秘

本文原创：**中国科学技术大学 张焕杰**

修改时间：2017.12.03

前几天一条标题是"中国部署4台IPv6根DNS，自主掌控互联网中枢命脉"的新闻刷了屏，针对该新闻暂不发表意见。

我们来看看中国大陆根DNS服务器的奥秘。

根DNS服务器是互联网上DNS工作最核心的服务器，它存放的信息和处理功能其实很简单。

# 一、根DNS服务器存有什么信息

根DNS服务器上存放有顶级域的域名，以及负责该顶级域解析的域名服务器信息。

比如中国的顶级域.cn，在根DNS服务器中就有如下的信息：

````
cn.                     172800  IN      NS      a.dns.cn.
cn.                     172800  IN      NS      b.dns.cn.
cn.                     172800  IN      NS      c.dns.cn.
cn.                     172800  IN      NS      d.dns.cn.
cn.                     172800  IN      NS      e.dns.cn.
cn.                     172800  IN      NS      ns.cernet.net.
ns.cernet.net.          172800  IN      A       202.112.0.44
a.dns.cn.               172800  IN      A       203.119.25.1
a.dns.cn.               172800  IN      AAAA    2001:dc7:0:0:0:0:0:1
b.dns.cn.               172800  IN      A       203.119.26.1
c.dns.cn.               172800  IN      A       203.119.27.1
d.dns.cn.               172800  IN      A       203.119.28.1
d.dns.cn.               172800  IN      AAAA    2001:dc7:1000:0:0:0:0:1
e.dns.cn.               172800  IN      A       203.119.29.1
h.dns.cn.               172800  IN      A       125.208.32.1
h.dns.cn.               172800  IN      AAAA    2001:dc7:fffe:0:0:0:0:1
i.dns.cn.               172800  IN      A       125.208.33.1
i.dns.cn.               172800  IN      AAAA    2001:dc7:ffff:0:0:0:0:1
j.dns.cn.               172800  IN      A       125.208.34.1
k.dns.cn.               172800  IN      A       125.208.35.1
l.dns.cn.               172800  IN      A       125.208.36.1
````

细心的会看出来，前面只有a-e.dns.cn，后面的h-l.dns.cn是干什么用的呢？后面的那些是".中国"之类的中文域名解析
使用的。

这些信息不是机密，负责DNS的机构IANA在 [Root Zone File](https://www.iana.org/domains/root/files) 公开提供，目前该文件大小是2.2MB。

# 二、根DNS服务器的处理过程

当一台DNS服务器解析一个域名时，首先向根DNS域名服务器发送请求。

比如一台DNS服务器需要查询科大的主页`www.ustc.edu.cn`对应的IP地址，它会首先向某一个根DNS服务器发送
`wwww.ustc.edu.cn`的域名请求。根DNS服务器接到请求后，根据自己保存的信息，返回一个下面的应答
```
;; AUTHORITY SECTION:
cn.                     172800  IN      NS      a.dns.cn.
cn.                     172800  IN      NS      e.dns.cn.
cn.                     172800  IN      NS      d.dns.cn.
cn.                     172800  IN      NS      c.dns.cn.
cn.                     172800  IN      NS      ns.cernet.net.
cn.                     172800  IN      NS      b.dns.cn.
;; ADDITIONAL SECTION:
a.dns.cn.               172800  IN      A       203.119.25.1
a.dns.cn.               172800  IN      AAAA    2001:dc7::1
b.dns.cn.               172800  IN      A       203.119.26.1
c.dns.cn.               172800  IN      A       203.119.27.1
d.dns.cn.               172800  IN      A       203.119.28.1
d.dns.cn.               172800  IN      AAAA    2001:dc7:1000::1
e.dns.cn.               172800  IN      A       203.119.29.1
ns.cernet.net.          172800  IN      A       202.112.0.44
```

这些应答的含义是：请向`a.dns.cn、b.dns.cn、... ns.cernet.net`查询，它们会负责处理`www.ustc.edu.cn`有关的信息。

域名服务器会从`a.dns.cn、b.dns.cn、... ns.cernet.net`挑一个，向它发送查询。这样的步骤一直持续，直到有个域名服务器返回了IP地址信息。

从上面过程看，根DNS服务器需要进行的处理真的很简单。

那问题来了，一个普通的DNS服务器，怎么知道谁是根DNS服务器呢？

答案很简单，每个DNS服务器都有个文件，存放有根DNS服务器的信息，这个文件很小，有用的信息只有13*3=39行，每3行是一个根DNS服务器的信息，如下所示：
```
.                        3600000      NS    A.ROOT-SERVERS.NET.
A.ROOT-SERVERS.NET.      3600000      A     198.41.0.4
A.ROOT-SERVERS.NET.      3600000      AAAA  2001:503:ba3e::2:30
``` 
DNS服务软件会附带提供这个文件，一般很少修改。IANA也有提供，放在 [Root Hints File](https://www.iana.org/domains/root/files)。如果你维护有DNS服务器，建议从这里下载最新的更新一下。

如果每次域名查询都这样从根DNS开始，查询响应的速度会很慢。实际上DNS服务器会尽量使用缓存，减少不必要的查询，大大加快域名查询的响应速度。

# 三、一共有多少个根DNS服务器呢

这个问题很不好回答。

由于UDP数据包长度的原因，根DNS服务器最多只有13个，就是 a-m.root-servers.net。

但这仅仅是有13个不同名字（地址）的根DNS服务器，实际上每个名字（地址）对应有多台服务器，或者说存在很多台根DNS服务器，它们具有相同的IP地址。

一般来说，互联网上的IP很少会重叠使用，也就是说很少有2台设备使用相同的IP地址。但例外就是DNS这种服务，称为AnyCast服务，可以由很多设备使用相同的IP，就近对外提供服务。

[http://root-servers.org/](http://root-servers.org/) 网站公布有分布在全球的根DNS服务器，从这里可以看到中国大陆有若干。


# 四、自己可以做根DNS服务器吗

当然可以，而且非常简单。只要安装DNS服务软件，定期从IANA网站下载那个2.2MB的根文件，就可以做根DNS服务器了。

为了方便使用，需要在公布的那些IP地址提供服务，一般来说需要利用BGP/OSPF/静态路由之类手段的把路由注入到网络中
（注入是文雅的说法，其实就是劫持到根DNS服务器IP的路由），因此最好是ISP来做才比较方便。

使用BGP注入路由并不复杂，远比想象的简单，可以参考 [使用ExaBGP发送BGP路由信息和清洗DDoS流量](../../security/bgp/exabgp/README.md)

# 五、国内有ISP做根DNS服务器吗

当然有了，比如中国教育和科研计算机网，从合肥测试的话，与a-m.root-servers.net的通信延迟，主要分3档：

|延迟|服务器|
|----|------|
|0ms |f j|
|30ms|a b c d e g i k l m|
|300ms|h|

从延迟可以推断，f j这两个服务器就在合肥(确切的说就在中国科学技术大学网络信息中心机房内)，h服务器不在国内，其它的在中国教育和科研计算机网内的某个地方。

仅仅中国教育和科研计算机网内就至少有12个根DNS服务器，而且其中2个就在合肥。

到这里就明白为什么很难回答有多少个根DNS服务器，因为谁也不知道。


***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
