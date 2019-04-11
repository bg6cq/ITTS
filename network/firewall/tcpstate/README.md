## [原创] TCP状态防火墙带来的故障

本文原创：**中国科学技术大学 张焕杰**

修改时间：2019.04.11

防火墙支持所谓的TCP状态跟踪，不幸的是这个功能在应用中会带来不容易定位或处理的疑难杂症。

## 一、状态防火墙

最早的防火墙是无状态方式工作，仅仅根据数据包中的信息来判断是否允许一个数据包通过。

所谓的状态防火墙，会记录连接的状态，只有符合一定时序状态的数据包，才允许通过。

比如，某个IP_server是web服务器，开放了tcp 80端口。对于无状态防火墙，任何发送给IP_server的tcp 80端口数据包，都会被允许通过。

但是对于状态防火墙，如果IP_Client和IP_server有如下的4个数据包通信，则第3个数据包经过防火墙后，
防火墙会认为对应的TCP连接已经处于建立状态。此后一段时间（所谓的TCP超时时间）内，如果有第4个数
据包试图经过防火墙，则会被禁止通过，原因是处于TCP连接建立状态时，不应该出现带有SYN标志的数据包。

```
IP_client:2000 --> IP_server:80 SYN
IP_client:2000 <-- IP_server:80 SYN+ACK
IP_client:2000 --> IP_server:80 ACK 
IP_client:2000 --> IP_server:80 SYN (这个包会被禁止通过)
```

当存在NAT时，状态防火墙是必须的，否则无法工作。

其他情况下，状态防火墙不是必须的。一般来说，当启用状态防火墙时，一旦一个连接允许建立，后续的数据包都会允许
通过防火墙，也就是启用状态防火墙时仅仅考虑新建连接的数据包，不必太多考虑返回的数据包，可以简化防火墙的配置。

状态防火墙工作时最容易带来故障的是连接跟踪的超时时间。这类故障的出现有一定的随机性，不容易定位。根据应用环境的不同，要调整合适的超时时间，才能解决故障。

下面以几个案例来详细说明和分析。

## 二、Linux iptables设置不当引起的Nginx 504错误

如下非常简单的网络结构，NginxServer对外提供服务，转发给Apache_Server：

```
NginxServer  --->  Apache_Server
```

Apache_Server上设置了如下的iptables规则:

```
iptables -A INPUT -j ACCEPT -m state --state NEW -p tcp --dport 80 -s NginxServer
iptables -A INPUT -j REJECT
```

系统比较繁忙，大部分时候工作正常，偶尔会碰到Nginx 504错误，特别是一旦重启NginxServer，出现504错误的几率会高很多。

504错误往往是Nginx无法连接Apache_Server引起的。经过抓包，发现出现错误时NginxServer收到了REJECT引发的不可达icmp包。

未匹配第一行ACCEPT规则与Linux中netfilter的conntrack连接跟踪默认超时时间太长(432000秒，5天)有关。
一旦出现某些异常，如重启NginxServer，会导致大量的tcp连接仍被认为处于已经建立状态，需要等5天后才会从Apache_server中删除。
在状态删除之前，NginxServer新建立的连接，如果源端口使用的是处于已经建立状态的连接，会不匹配NEW状态，从而被REJECT，也就是
回应icmp不可达包。

解决办法：
1. 修改iptables规则，不检查状态。对于高并发的服务器，建议都这样做：
```
iptables -A INPUT -j ACCEPT -p tcp --dport 80 -s NginxServer
iptables -A INPUT -j REJECT
```

2. 如果不涉及NAT，建议完全禁用conntrack连接跟踪，对于高并发的服务器，建议都这样做

3. 如果一定要启用连接跟踪，修改conntrack中有关超时时间，可以参考以下链接优化

* https://github.com/bg6cq/nginx-centos-6
* https://github.com/bg6cq/nginx-centos-7

延伸问题：

对于高并发的环境，一旦其中牵涉了连接跟踪，重启设备时需要多考虑。比如上面的环境，如果先重启NginxServer，
再重启Apache_Server，一切正常。如果先重启Apache_Server，再重启NginxServer，就会带来问题。

## 三、某防火墙tcp默认超时时间引起的Nginx 504错误

如下网络结构，NginxServer对外提供服务，转发给 防火墙 后面的Web_server，防火墙做了NAT。

```
NginxServer  --->  防火墙(NAT)  --->  Web_Server
```

NginxServer上观察到偶尔有504错误，某个时间段会突然一分钟出现上百次错误。

在Nginx日志中增加 `TIME:$request_time/$upstream_connect_time/$upstream_header_time/$upstream_response_time`，记录处理时间。日志中显示
出现504错误的访问，$upstream_connect_time全部是设置的proxy_connect_timeout时间10秒。也就是因为Nginx连接Web_Server超时引发了504错误。

经检查Web_Server上未开启iptables，仅仅在 防火墙 上有拦截过滤。

调整防火墙的超时时间后，未再出现504错误。

## 四、某防火墙连接跟踪超时处理bug引起的故障

这是10多年前的故障，现在防火墙厂家早就修复了bug。

一个防火墙，工作了很久，某一天突然发现连接数持续增多。后重启防火墙后恢复。

分析该防火墙内部用32位无符号数存放时间计数器，计数器每秒钟增加100，32位计数器在2^32/100/3600/24=497天，也就是16个半月后就会溢出。时间计数器溢出后，处理超时的代码有bug不再将超时的连接删除，导致连接数越来越多。

经检查该防火墙设备上一次重启是在497天左右，将情况向防火墙公司反馈，后升级软件，故障不再出现。



***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
