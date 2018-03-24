## [原创]2层交换机生成树TC事件的集中记录和处理

本文原创：**中国科学技术大学 张焕杰**

修改时间：2018.03.24

在2层交换机中，生成树的TC（topology change）事件是需要重点关注的。

如果是生成树root发生变化的TC事件，会导致局部网络中断几十秒钟。其他的TC事件会触发所有交换机刷新转发表，瞬间产生较多的
flood数据包。这些都会影响局域网的正常运行。

## 一、2层交换机典型设置

我校的2层交换机主要设置的有：

* 楼汇聚交换机，设置为生成树root
* 楼层交换机接入用户的接口典型配置如下(以H3C交换机为例)：
```
interface GigabitEthernet1/0/1
 port access vlan xxx
 loopback-detection enable
 broadcast-suppression pps 10
 multicast-suppression pps 10
 stp edged-port enable
```

## 二、TC事件产生的原因

设置成`spanning tree edge port`的接口up/down时不会触发TC更新。这样的2层交换机设置在运行时，正常情况下不应该出现TC事件。

对于上述设置，交换机出现TC事件有以下几种原因：

1. 管理员忘记将用户侧的接口设置为`spanning tree edge port`。
2. 用户侧接口间存在环路，使得`spanning tree edge port`不起作用，并且触发环路，这是需要尽快发现并解决的。
3. 户侧接口又连接了交换机，并且开启了生成树。如果该交换机未将接入端口设置为`spanning tree edge port`，接口up/down时引发TC。
4. 某些交换机重启，重启时与其他交换机间链路的up/down会引发TC。

对于上述1、2、3都要重点关注。

## 三、TC事件的集中记录和处理

为了集中记录和处理TC事件，我们设置一台rsyslog服务器，用来收集楼内汇聚交换机的日志。楼内发生TC事件时，汇聚交换机会将日志发送给rsyslog服务器，并被记录和后续处理。

### 3.1 rsyslog服务器配置

我们使用的是CentOS 6.9系统，安装后已经有rsyslog服务。只要修改`/etc/rsyslog.conf`后使用`service rsyslog restart`重启服务，并确保UDP端口514对交换机IP开放即可。

```
#允许处理udp接收到的syslog日志
$ModLoad imudp
$UDPServerRun 514

#增加swlog模版，记录发送设置的IP地址
$template swlog,"%fromhost-ip% %TIMESTAMP% %HOSTNAME% %syslogtag%%msg:::sp-if-no-1st-sp%%msg:::drop-last-lf%\n"

#禁止将local3有关的日志写到 message
local3.none;*.info;mail.none;authpriv.none;cron.none                /var/log/messages

#将交换机发来的local3日志写到 /var/log/sw.log，并使用上面定义的模版 swlog
local3.*                                                /var/log/sw.log;swlog
```

为了避免sw.log一直增长，可以修改`/etc/logrotate.d/syslog`，在其中增加一行`/var/log/sw.log`。

### 3.2 交换机的设置

我校使用的H3C交换机，只要增加以下配置即可

```
 info-center loghost x.x.x.x facility local3
```

`x.x.x.x`是上述rsyslog服务器IP地址。


### 3.3 TC 事件的筛选

设置以下crontab任务，定时将TC事件日志存成文件，管理员定期查看文件`topology.txt`即可了解发生过的TC事件。

登录到相关交换机上可以进一步查找TC事件的根源，针对不同的原因进行处理即可。


```
*/5 * * * grep "topology" /var/log/sw.log > /var/www/html/topology.txt
```
## 四、TC事件的处理例子

如发现有以下的TC日志，说明Ten-GigabitEthernet1/4/6触发了TC事件：
```
202.38.xx.xx Mar 24 15:38:18 2018 Core-7504-ZhongQu %%10STP/6/STP_NOTIFIED_TC: Instance 0's port Ten-GigabitEthernet1/4/6 was notified a topology change.
202.38.xx.xx Mar 24 15:38:21 2018 Core-7504-ZhongQu %%10STP/6/STP_NOTIFIED_TC: Instance 0's port Ten-GigabitEthernet1/4/6 was notified a topology change.
202.38.xx.xx Mar 24 15:38:33 2018 Core-7504-ZhongQu %%10STP/6/STP_NOTIFIED_TC: Instance 0's port Ten-GigabitEthernet1/4/6 was notified a topology change.
202.38.xx.xx Mar 24 15:38:34 2018 Core-7504-ZhongQu %%10STP/6/STP_NOTIFIED_TC: Instance 0's port Ten-GigabitEthernet1/4/6 was notified a topology change.
202.38.xx.xx Mar 24 15:38:36 2018 Core-7504-ZhongQu %%10STP/6/STP_NOTIFIED_TC: Instance 0's port Ten-GigabitEthernet1/4/6 was notified a topology change.
```
登录该汇聚交换机，执行命令`dis lldp neighbor-information interface ten1/4/6 ver`得知连接的是一台接入交换机。

登录该接入交换机，查看对应时间的日志，可以看到那时一个与其他交换机相连的接口up/down引发了TC。经检查连接的是视频监控交换机，估计是管理人员调试插拔网线引起的。


***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
