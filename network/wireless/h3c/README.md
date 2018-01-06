## [原创] H3C AC/AP 调试命令

本文原创：**中国科学技术大学 张焕杰**

修改时间：2017.12.25

# 一、开启AP的远程telnet功能

```
<AC>sys
[AC]_hidecmd
[AC-hidecmd] wlan ap ap_name exec-control enable
[AC-hidecmd] wlan ap ap_name telnet enable
```
然后接可以telnet到AP上

# 二、查看AP信道利用率、空口状态

使用上述一开启telnet，并telnet到AP上
```
[WA...]_hidecmd
[WA...-hidecmd]dis ar5drv 2 channelbusy
[WA...-hidecmd]dis ar5drv 2 statistics
# 1 是5G，2 是2.4G
```

# 三、查看一个客户端的漫游状态

```
<AC>display wlan client roam-track mac-address xxxx-xxxx-xxxx
```

# 四、高密度网络优化措施

## 1.（必选）信号强度达标

优化手段都是以满足信号强度为基础，无线覆盖区域内信号强度是第一位。企业级应用程序流量或笔记本，信号强度不能低于-75dBm；要想跑VoIP流量或PDA手持终端等，信号强度至少不低于-65dBm以上。

信号强度参考数值：

* -60dBm及以上：表示信号强度质量尚佳
* -80dBm到-60dBm：表示无线信号质量尚可
* -80dBm到-90dBm：表示无线信号较弱
* -90dBm及以下：表示无线信号极弱

信号覆盖涉及因素

* 天线类型：定向与全向天线 关键词 增益 方向校准
* 安装位置：壁挂、放装、入室 关键词 802.11与非802.11干扰
* 信号障碍物：金属、混泥土墙、玻璃 关键词 电磁波特性

在部署场景的选择上，AP或者天线要尽量离目标区域近，并保证无金属板、厚墙阻隔。对于宿舍或教室类场景，不建议AP楼道部署通过信号穿墙的方式完成覆盖。

## 2.（必选）信道规划和设置固定信道

信道规划和功率调整是保障WLAN网络品质的最佳手段，外加频谱导航手段俗称优化三板斧，效果也就能达到70%~80%左右，但具体还是视实际部署情况而定，如高密环境考虑接入人数，提高的是AP间密度，牺牲了管理、控制报文对空口的占用率增大，同频段可见度增大，发生电磁波碰撞和报文传输等待时间增加，大大降低使用效率。

如果所用AP都工作在相同的信道，这些AP只能共享一个信道的频率资源，造成整个WLAN玩过性能过低。在多楼层无线覆盖时，考虑到三维空间的信号泄漏，在信道规划策略上，除了基于平面的非重叠交叉部署，还要考虑楼层间或立体空间下的信道交叉规划设计。

强烈推荐：802.11n网络在实际部署时，无论是2.4G频段或5G频段，建议都采用20MHz模式进行覆盖，以加强信道隔离与复用，提升WLAN网络整体性能。（注意：我司AP在802.11n 5G频段默认为40MHz频宽方式）

信道规划【V7版本举例】V5/V7无差异

```
<sysname> system-view
[sysname]wlan ap ap1 model WA4320-ACN-SI
[sysname-wlan-ap-ap1]radio 2
[sysname-wlan-ap-ap1-radio-2]channel 1
```

频宽设置【V7版本举例】V5/V7无差异
```
# 配置当前接口的带宽模式为20MHz。
<sysname> system-view
[sysname]wlan ap ap1 model WA4320-ACN-SI
[sysname-wlan-ap-ap1]radio 2
[sysname-wlan-ap-ap1-radio-2]channel band-width 20
This operation might cause channel change. Continue? [Y/N]:
[sysname-wlan-ap-ap1-radio-2]y
```

## 3.（必选）功率规划和设置固定功率

无线局域网，信道资源是非常稀缺的，每个AP只能工作在非常有限的非重叠信道上，ISM频段又为一个公共共有资源，WLAN、蓝牙、ZigBee、微波炉等均工作在2.4Ghz频段，5Ghz频段被部分区域雷达所用。功率规划有效控制无线覆盖区域内的WLAN系统内与系统外的干扰，加强相同信道频谱资源的复用，实现通信的持续进行，为网络的可靠传输提供保证。

功率规划【V7版本举例】V5/V7无差异

配置射频的最大传输功率为5dBm。
```
<sysname> system-view
[sysname] wlan ap ap1 model WA2100
[sysname-wlan-ap-ap1] radio 1
[sysname-wlan-ap-ap1-radio-1]max-power 5
```

## 4（必选）为无线业务构建独立的VLAN
Wireless LAN无线局域网，Wireless传输介质为电磁波，易受外界环境的影响，协议本身需要ACK报文来确定数据包发送的完整性，LAN网络最大的特点是ARP广播包，传输介质频谱资源非常有限，ARP广播包往往会大大降低空口的带宽，使无线性能急剧降低。通过把同一个物理无线局域网的不同用户逻辑划分成多个不同广播域，从而控制广播和单播流，增大空口利用率。

注：当有线网络与无线网络共用一个广播域时，有线侧的ARP广播包对无线侧空口的冲击是相当大的，大大占用了空口资源。无线业务使用独立VLAN也可避免来自有线侧arp类的攻击。

## 5.（必选）V7默认开启三层漫游功能，无需特别配置
V7版本WLAN-ESS虚接口与服务模板service-template合并，且三层漫游功能默认开启，在service-template下VLAN优先级小于radio射频口。

三层漫游配置【V7版本举例】
```
wlan ap ap2 model WA2620i-AGN id 2
 serial-id 219801A0CNC124004002
 radio 1
 radio 2
  service-template 1 vlan-id 2
  radio enable
#
wlan ap ap3 model WA2620i-AGN id 3
 serial-id 219801A0CNC124004003
 radio 1
 radio 2
  service-template 1vlan-id 3
  radio enable
```

【V5差异】
```
#V5需额外配置WLAN-ESS接口，端口类型为hybird，并使能mac-vlan功能
interface WLAN-ESS1
link-type hybrid
undo port hybridvlan 1
port hybrid vlan100 untagged
port hybrid pvidvlan 100
mac-vlan enable

#交换机与AC互联口为trunk模式时，尽量避免使用port trunk permit vlan all，建议只放通在用业务VLAN。

interface GigabitEthernet1/0/1
description linkto hexin
port link-typetrunk
undo port trunkpermit vlan 1 #有线网络侧交换机默认端口属于VLAN1建议undo
port trunkpermit vlan 10 20 30 100 200
```

## 6.（强烈推荐）无线用户VLAN内二层隔离

在上述优化4中为无线业务构建独立的VLAN，也就是将大的局域网虚拟分割成若干份，分割份数越多，相互虚拟的局域网之间的广播/组播影响就越小。但在无线接入密度过高的场景下，用户行为无法约束（arp二层网络攻击），用户数量无法预测，无法统一控制管理，强烈建议使用无线用户VLAN二层隔离，AC上控制无线用户只能访问网关设备，而不能互相之间访问，同时配置undouser-isolation permit broadcast禁止有线用户（permit-mac允许的mac地址除外）发送广播、组播报文给无线用户。此功能对无线用户接入密度较大，用户属性不定难以集中控制的场景见效好，如企业无线guest用户，公共会议室guest用户，场馆guest用户。

无线业务VLAN网关尽量配置在核心交换机侧，尽量避免设置于AC控制板。交换机可分为交换模块和控制模块，交换模块更倾向于端口的快速转发功能，所需CPU资源较小，处理性能更快，控制模块用于IP层数据转发、ACL控制、路由计算，对CPU资源的依赖较大。

无线用户二层隔离配置【V7版本举例】
```
# 基于VLAN，配置VLAN 2下用户隔离，放通网关MAC: 7425-8a36-d87d
<sysname> system-view
[sysname] user-isolation vlan 2 enable
[sysname] user-isolation vlan 2 permit-mac 7425-8a36-d87d
[sysname] undo user-isolation permit broadcast
#网关地址10.10.240.1，命令cmd arp –a 或displayinterface vlan 2查看对应网关MAC

#基于SSID，配置SSID下的用户隔离
<sysname> system-view
[sysname]Wlan service-template 1
[sysname]user-isolation enable
#使能ARP Snooping功能，使AC可以显示学习到的IP地址
[sysname] arp-snooping enable
```

## 7.（强烈推荐）关闭RRM低速率

无线WLAN网络中不是使用固定的速率发送所有的报文，而是使用一个速率集进行报文发送（例如11g支持1、2、5.5、11、6、9、12、18、24、36、48、54Mbps），实际无线终端或者AP在发送报文的时候会动态的在这些速率中选择一个速率进行发送。通常提到的11g可以达到速率主要指所有报文都采用54M速率进行发送的情况，而且是指的一个空口信道的能力。而实际上大量的广播报文和无线的管理报文都使用最低速率1Mbps进行发送，所以会消耗一定得空口资源。在无线网络中信号传输的距离不是问题的情况下，可以将1、2、6和9Mbps速率禁用，这样整体上减少广播报文和管理报文对空口资源的占用。

关闭RRM低速率【V7版本举例】
```
[sysname-wlan-ap-ap1-radio-2]rate disabled 1 2 5.5 6 9 //APradio视图
[sysname]wlan ap-group 1
[sysname-wlan-ap-group-1]ap-model WA2610E
[sysname-wlan-ap-group-1-ap-model-WA2610E]radio 1
[sysname-wlan-ap-group-1-ap-model-WA2610E-radio-1]ratedisabled 1 2 5.5 6 9

//AP组 radio视图
```
【V5版本举例】
```
[sysname] wlan rrm
[sysname-rrm] rate disabled 1 2 5.5 6 9
```

V7版本关闭低速率更加灵活，可以基于单个AP实现，更有利于无线优化。

## 8.（强烈推荐）开启无线用户限速

WLAN网络中每一个AP提供的可用带宽有限，且由接入的无线客户端共享，如果个别的无线用户通过WLAN使用网络工具下载文件，可能达到非常大的流量，进而直接耗尽当前共享带宽，造成其他无线用户访问网络慢、ping抖动丢包等问题。通过配置用户限速功能，可以限制部分无线客户端对带宽的过多消耗，保证所有接入无线客户端均能正常使用网络业务。基于无线客户端的速率限制功能有两种模式：动态模式和静态模式，其中静态模式为静态的配置每个客户端的速率，即配置的速率是同一个AP内，每个客户端的最大速率。

无线用户限速【V7版本举例】

```
#基于服务模板下的限速，单位kbps
wlan service-template 1
ssid wangp
client-rate-limitenable
client-rate-limitinbound mode static cir 512
client-rate-limitoutbound mode static cir 512
service-template enable

#基于radio射频口的限速，单位kbps
radio 2
channel 6
max-power 5
radio enable
client-rate-limitenable
client-rate-limitinbound mode static cir 512
client-rate-limitoutbound mode static cir 512

#基于AP-group组的限速，单位kbps
[sysname-wlan-ap-group-1-ap-model-WA1208E-GNP-radio-1]client-rate-limitenable
[sysname-wlan-ap-group-1-ap-model-WA1208E-GNP-radio-1]client-rate-limitinbound mode static cir 512
[sysname-wlan-ap-group-1-ap-model-WA1208E-GNP-radio-1]client-rate-limitoutbound mode static cir 512
```

## 9.（强烈推荐）关闭广播Probe探测回应

WLAN有两种探测机制：一种为无线终端被动的侦听Beacon帧之后，根据获取的无线网络情况，选择AP建立连接；另外一种为无线终端主动发送Probe request探测周围的无线网络，然后根据获取的Probe Response报文获取周围的无线网络，之后选择AP建立连接。

本功能主要针对Probe探测方式。根据Probe Request帧（探测请求帧）是否携带SSID，可以将主动扫描分为两种：1、广播方式的Probe探测，客户端发送Probe Request帧（Probe Request中SSID为空，也就是SSID IE的长度为0）；2、单播方式的Probe探测，客户端发送的Probe Request帧（携带指定的SSID）。

而大部分的无线终端都不会指定要链接的“无线接入服务”，这样就造成了无线终端会大量发送广播Probe Request探测，造成所有的接收到该报文的AP设备都会回应Probe Response报文。因此，在无线用户比较多的网络中，可能会出现一定量的Probe Response报文，而且这些报文都是使用低速率进行发送，会消耗一定的空间资源。如果网络条件允许可以考虑关闭广播Probe探测功能，AP针对SSID为空的探测请求不进行回复，有效降低空口的消耗，使整个WLAN网络应用得到一定的提升。

关闭广播Probe探测回应【V7版本举例】

```
#基于单AP终端设备
wlan ap ap1 model WA4320-ACN-SI
serial-id219801A0T78166E07840
broadcast-probereply disable

#基于AP组
[sysname-wlan-ap-group-1]broadcast-probe reply disable
<sysname> system-view
[sysname] wlan ap ap3 model WA2100
[sysname-wlan-ap-ap3] undo broadcast-probe reply
```
【V5版本举例】
```
<sysname> system-view
[sysname] wlan ap ap3 model WA2100
[sysname-wlan-ap-ap3] undo broadcast-probe reply
```

## 10.（强烈推荐）开启频谱导航

在实际无线网络环境中，某些客户端只能工作在2.4GHz频段上，也有一部分客户端可以同时支持2.4GHz和5GHz频段，如果支持双频的客户端都工作在2.4GHz频段上，会导致2.4GHz频段过载，5GHz射频相对空余。在这种情况下，可以在设备上开启频谱导航功能。频谱导航功能可以将支持双频工作的客户端优先接入5GHz射频，使得两个频段上的客户端数量相对均衡，从而提高整网性能。

开启频谱导航功能后，AP会对发起连接请求的客户端进行导航，将其均衡地连接至该AP的不同射频上。首先当客户端与某个AP连接时，若该客户端只支持单频2.4GHz，则频谱导航功能不生效，客户端直接关联至AP的2.4GHz射频上。若客户端支持双频，AP则会将客户端优先引导至5GHz射频上。若客户端只支持单频5GHz，则会直接关联至AP的5GHz射频上。在双频客户端关联到5GHz射频前，AP会检查5GHz射频接收到的客户端的RSSI值，若该RSSI值低于设定值，则不会将此客户端导航至5GHz射频。

如果5GHz射频上已连接的客户端数量达到门限，且5GHz射频与2.4GHz射频上连接的客户端差值达到或超过差值门限，AP会拒绝客户端接入5GHz射频，且允许新客户端接入2.4GHz射频（即不会引导双频客户端优先接入5GHz射频）。如果客户端反复向该AP的5GHz射频上发起关联请求，且AP拒绝客户端关联请求次数达到/超过设定的最大拒绝关联请求次数，那么该AP会认为此时该客户端不能连接到其它任何的AP，在这种情况下，AP上的5GHz射频也会接受该客户端的关联请求。

【V7版本举例】
```
# 开启全局频谱导航功能。
[AC] wlan band-navigation enable
# 开启AP组或单AP频谱导航功能。
[Sysname] wlan ap-group 1
[Sysname-wlan-ap-group-1] band-navigationenable

# 开启频谱导航负载均衡功能功能，客户端连接数门限为5，客户端连接数差值门限为2。
[AC] wlan band-navigation balance session10 gap 5

# 配置拒绝客户端对5GHz射频关联请求的最大次数为3。
[AC] wlan band-navigation balanceaccess-denial 3

# 配置频谱导航RSSI门限值为30。
[AC] wlan band-navigation rssi-threshold 20

# 配置频谱导航的客户端信息老化时间为200秒。
[AC] wlan band-navigation aging-time 200
```

【实际配置】可参考
```
[AC] wlan band-navigation enable
[AC] wlan band-navigation rssi-threshold 25 #视具体项目环境而定
[Sysname] wlan ap-group 1
[Sysname-wlan-ap-group-1] band-navigationenable
[Sysname-wlan-ap-group-1]apname #添加ap
```

## 11.（推荐）Beacon帧间隔调整到160TU

默认情况下，射频卡radio上的每个SSID每100TU就会发送一个Beacon信标报文，这个报文通告WLAN网络服务，同时和无线网卡进行信息同步。Beacon报文通常使用最小速率进行发送，而且优先级比较高，所以考虑将Beacon发送的时间间隔从100TU调整到160-200TU之间，这样可以有效降低空口的消耗，使整个WLAN网络应用得到一定的提升。

通常情况下，一个radio下配置SSID的数量建议不超过5个。

调整Beacon帧间隔【V7版本举例】
```
#基于radio射频口
[sysname-wlan-ap-ap1]radio 1
[sysname-wlan-ap-ap1-radio-1]beacon-interval 160

#基于AP-group组
[sysname]wlan ap-group 1
[sysname-wlan-ap-group-1]ap-model WA1208E-GNP
[sysname-wlan-ap-group-1-ap-model-WA1208E-GNP]radio 1
[sysname-wlan-ap-group-1-ap-model-WA1208E-GNP-radio-1]beacon-interval160
```
【V5版本举例】
```
<sysname> system-view
[sysname] wlan radio-policy radio1
[sysname-wlan-rp-radio1] beacon-interval 160
```
然后将radio-policy在各个AP的Radio接口上应用。

## 12.（推荐）禁止弱信号终端接入

在WLAN网络中，信号强度较弱的无线客户端，虽然也可以接入到网络中，但是所能够获取的网络性能和服务质量要比信号强度较强的无线客户端差很多。如果弱信号的无线客户端在接入到WLAN网络的同时还在大量地下载数据，就会占用较多的信道资源，最终必然对其他的无线客户端造成很大的影响。

禁止弱信号客户端接入功能，通过配置允许接入的无线客户端的最小信号强度门限值，可以直接拒绝信号强度低于指定门限的无线客户端接入到WLAN网络中，减少弱信号客户端对其他无线客户端的影响，从而提升整个WLAN网络的应用效果。

禁止弱信号终端接入【V7版本举例】
```
#基于radio射频口
[sysname-wlan-ap-ap1]radio 1
[sysname-wlan-ap-ap1-radio-1]option client rejectenable rssi 30

#基于AP-group组
[sysname-wlan-ap-group-1-ap-model-WA1208E-GNP-radio-1]optionclient reject enable rssi 30
```

【V5版本举例】
```
<sysname> system-view
[sysname] wlan option client-reject 15
```

## 13. 主动出发客户端重新连接

wlan option client-reconnect-trigger 

## 14. 漫游导航

wlan optoin roam-navigateion level


参考资料：

https://wenku.baidu.com/view/e5b43a61524de518964b7df4.html

http://m.yopai.com/show-2-60868.html




***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
