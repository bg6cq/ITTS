# 高校网络信息安全工作组成员发布、共享技术操作规范文档

1. 机房环境
    1. 门禁
    2. 精密空调
    3. UPS
2. 网络
    1. 网络防火墙([使用CentOS 7做NAT设备](network/nat/centos7/README.md))
    2. 路由器
    3. 交换机
    4. AC控制器
    5. VPN([通过互联网桥接2个以太网段](https://github.com/bg6cq/ethudp/blob/master/sample2/README.md))
    6. 流量控制
3. 服务器存储
    1. 服务器
    2. 存储
    3. VMWare
    4. Hyper-V
    5. KVM
    6. citrix
    7. 集群
    8. Docker
4. 操作系统
    1. windows server
    2. windows 7/8/10
    3. CentOS/Redhat
    4. Debian
    5. ubuntu
    6. [主机防火墙](OS/firewall/README.md)
5. 数据库
    1. MS SQL Server
    2. Oracle
    3. MySQL
    4. NoSQL
6. 中间件
    1. tomcat
7. 应用系统
    1. apache
    2. nginx([nginx反向代理服务器](app/nginx/README.md), [nginx-rrd绘图参数](app/nginx/nginx-rrd/README.md), [Nginx 400错误案例](app/nginx/Cases.md))
    3. iis
    4. ftp
    5. dns([自己建立根DNS服务器](app/dns/root/README.md))
    6. [ntp](app/ntp)
    7. dhcp
    8. [使用Nextcloud提供私有网盘服务](app/nextcloud/README.md)
8. 安全管理
    1. [系统上线前安全评测技术要求](security/checklist/README.md)
    2. [使用Let's encrypt免费SSL证书](security/ssl/letsencrypt/README.md)
    3. [使用git监控www文件并自动恢复](security/www/git/README.md)
    4. [自动修改发垃圾邮件的账号密码(针对coremail环境)](security/mail/README.md)
    5. [ntpd/bind/IOS/JunOS等安全配置模板](http://www.team-cymru.org/templates.html)
    6. [使用ExaBGP发送BGP路由信息和清洗DDoS流量](security/bgp/exabgp/README.md)
	7. [使用ipsec加密Linux主机间通信](security/ipsec/README.md)
	8. [安全责任书模板](security/anquanzerenshu.md)

9. 标杆文档
    1. [中山大学信息技术安全管理办法](http://info.sysu.edu.cn/node/160)
	
10. 相关法律法规
    1. [中华人民共和国网络安全法](http://www.npc.gov.cn/npc/xinwen/2016-11/07/content_2001605.htm)
    2. [中华人民共和国国家安全法](http://www.npc.gov.cn/npc/xinwen/2015-07/07/content_1941161.htm)
    3. [中华人民共和国保守国家秘密法](http://www.npc.gov.cn/huiyi/cwh/1114/2010-04/29/content_1571766.htm)
    4. [中华人民共和国反恐怖主义法](http://www.npc.gov.cn/npc/xinwen/2015-12/28/content_1957401.htm)
    5. [中华人民共和国反间谍法](http://www.npc.gov.cn/npc/xinwen/2014-11/02/content_1884660.htm)
    6. [刑法修正案（九）](http://www.npc.gov.cn/npc/xinwen/2015-08/31/content_1945587.htm)
11. 其他
    1. [每日更新的电信,联通,移动等ISP地址段](https://ispip.clang.cn)
    2. [中国运营商IP地址库(每日更新)](https://github.com/gaoyifan/china-operator-ip/tree/ip-lists)
    3. [中国大陆根DNS服务器的奥秘](other/dns/README.md)

欢迎 [加入我们整理资料](work.md)

[Markdown 语法](http://wowubuntu.com/markdown/)
