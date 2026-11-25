## [原创] debian 使用 docker 时iptables 规则的维护

本文原创：**中国科学技术大学 张焕杰**

修改时间：2026.11.25

使用docker时，docker启动过程中，会在iptables中增加若干规则，这里给出一种方便自己iptables规则维护方法

### 1. 原理

docker启动后，会增加若干规则，其中nat表中增加 DOCKER 规则链以处理端口映射，filter表中增加 DOCKER-USER、DOCKER、DOCKER-ISOLATION-STAGE-1、DOCKER-ISOLATION-STAGE-2等规则链处理过滤。

这种情况下，不能简单使用iptables-persistent等管理。


### 2. 自建规则

在docker启动后，可以自在filter表 INPUT、OUTPUT、DOCKER-USER等规则链自建规则，不会影响到docker的运行。其中 INPUT、OUTPUT是系统接收和发送数据包，如限制哪些IP可以连接22端口可以在INPUT中增加规则。DOCKER-USER是docker有关的数据包，如限制哪些IP可以访问某个docker的端口可以在DOCKER-USER中增加规则。

例子，编辑如下文件 `/root/etc/rc.firewall`
```
#!/bin/bash
# 本脚本在docker 启动后执行，仅仅处理 INPUT 和 DOCKER-USER 规则，不能处理其他规则

iptables -F INPUT
iptables -F DOCKER-USER


iptables -A INPUT -j ACCEPT -m state --state ESTABLISHED,RELATED
iptables -A INPUT -j ACCEPT -i lo
iptables -A INPUT -j ACCEPT -p icmp
iptables -A INPUT -j ACCEPT -p tcp --dport 22 -s x.x.x.x
iptables -A INPUT -j ACCEPT -p tcp -m multiport --dports 80,443
iptables -A INPUT -j DROP
```

编辑文件 `/root/etc/rc.local`
```
#!/bin/bash

cd /root/ntfy
docker compose up -d

/etc/root/etc/rc.firewall
```

在crontab -e时，输入以下
```
@reboot /root/etc/rc.local
```

则可以实现系统启动后，docker compose 启动ntfy docker，然后设置INPUT规则限制对22端口等的访问。

### 3. 对某个docker的端口做限制

对某个docker暴露的端口做限制，可以修改 DOCKER-USER 规则链，注意此时已经经过DNAT，目的IP和目的端口已经是修改过的。如果需要匹配DNAT之前的目的端口，可以使用`conntrack`扩展，如下
```
iptables -I DOCKER-USER -p tcp -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -I DOCKER-USER -p tcp -m conntrack --ctorigdst 198.51.100.2 --ctorigdstport 80 -j ACCEPT
```



参考：

* https://docs.docker.com/engine/network/firewall-iptables/
* https://oneuptime.com/blog/post/2026-02-08-how-to-customize-docker-iptables-rules/view



***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
