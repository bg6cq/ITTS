# 高校网络信息技术有关文档

这里有大量的FTP文件，请参考 目录下的 files_list.txt

   [FTP大量文档](FTP)


1. 机房环境
    1. 门禁
    2. 精密空调
    3. UPS
    4. 环境监测([通过RS485总线使用Modbus RTU over TCP 协议采集环境传感器信息](env/modbus/README.md))
2. 网络
    1. 网络防火墙([使用CentOS 7做NAT设备](network/nat/centos7/README.md))
    2. 路由器
    3. 交换机([2层交换机生成树TC事件的集中记录和处](network/switch/stptc/README.md), [交换机sflow抓包的简单说明](network/switch/sflow/README.md) )
    4. AC控制器([H3C AC/AP隐藏调试命令](network/wireless/h3c/README.md))
    5. VPN([通过互联网桥接2个以太网段](https://github.com/bg6cq/ethudp/blob/master/sample2/README.md), [
OpenVPN的安装与部署（ldap进行身份认证+记录用户访问日志并发送邮件）](network/vpn/openvpn_ldap/README.md))
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
    1. apache([cenots 7 httpd 安装及安全加固](https://abanger.github.io/maintenance/2018/06/08/centos-7-httpd-security-reinforcement.html))
    2. nginx([step-by-step install nginx反向代理服务器](https://github.com/bg6cq/nginx-install), [nginx反向代理服务器](app/nginx/README.md), [nginx-rrd绘图参数](app/nginx/nginx-rrd/README.md), [Nginx 400错误案例](app/nginx/Cases.md), [使用lua对反向代理做权限控制](https://github.com/bg6cq/nginxauth), [Nginx服务器优化](app/nginx/nginx-opt))
    3. iis
    4. ftp([ProFTPd配置TLS](app/ftp/proftpd-tls.md), [ProFTPd配置LDAP](app/ftp/proftpd-ldap.md))
    5. dns([自己建立根DNS服务器](app/dns/root/README.md), [为何DNS服务器要禁用连接跟踪](app/dns/whynoconntrack/README.md), [DNS服务器的iptables规则](app/dns/iptables/README.md), [git辅助DNS服务器的运行](app/dns/dns_with_git/README.md))
    6. ntp([建立ntp服务器](app/ntp/README.md), [用GPS模块建立高精度ntp服务器](app/ntp/gps/README.md))
    7. dhcp([Linux下ISC dhcpd分配状态显示](app/dhcp/dhcpd-pool/README.md))
    8. [使用Nextcloud提供私有网盘服务](app/nextcloud/README.md)
8. 安全管理
    1. [系统上线前安全评测技术要求](security/checklist/README.md)
    2. [使用Let's encrypt免费SSL证书/getssl](security/ssl/letsencrypt/README.md)
    3. [使用Let's encrypt免费SSL证书/acme.sh](security/ssl/acme.sh/README.md)
    4. [自建CA签发证书](security/ca/README.md)
    5. [使用git监控www文件并自动恢复](security/www/git/README.md)
    6. [自动修改发垃圾邮件的账号密码(针对coremail环境)](security/mail/README.md)
    7. [ntpd/bind/IOS/JunOS等安全配置模板](http://www.team-cymru.org/templates.html)
    8. [使用ExaBGP发送BGP路由信息和清洗DDoS流量](security/bgp/exabgp/README.md)
    9. [使用ipsec加密Linux主机间通信](security/ipsec/README.md)
   10. [安全责任书模板](security/anquanzerenshu.md)
   11. [两步(多因素)认证原理及应用](security/mfa/README.md)
   12. [Top 20 OpenSSH Server Best Security Practices](https://www.cyberciti.biz/tips/linux-unix-bsd-openssh-server-best-practices.html)
   13. [高校等保一级应用系统的管理模式探究](security/l1.md)
   14. [从一个简单的备份需求演示gpg的使用](security/gpg/README.md)
   15. [人人都需要一个yubikey](security/yubikey/README.md)

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
    4. [可扩展视频直播设施建设](other/live/README.md)

欢迎 [加入我们整理资料](work.md)

[Markdown 语法](http://wowubuntu.com/markdown/)
