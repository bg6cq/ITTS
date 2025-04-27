## [原创] 使用vxlan实现校区间vlan透传

本文原创：**中国科学技术大学 张焕杰**

修改时间：2025.04.27

# 一、使用vxlan实现校区间vlan透传

![vxlan](img/vxlan.png)

如上网络拓扑，3台支持VXLAN功能的交换机放置在3个校区，交换机loopback 接口IP是192.168.0.1、192.168.0.2、192.168.0.3。3台交换机的loopback 接口间互相IP可达。

实现vlan 100、vlan 200 在校区间透传。

以下为使用华为S5731-H交换机作为VTEP交换机的配置。

# 二、交换机1配置

以下仅仅展示vxlan有关配置。实现loopback接口互通的配置未提供。
```
vlan batch 100 200

bridge-domain 100
 l2 binding vlan 100
 vxlan vni 100
bridge-domain 200
 l2 binding vlan 200
 vxlan vni 200

interface GigabitEthternet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 100 200
 stp disable

interface LoopBack0
 ip address 192.168.0.1 255.255.255.255

interface Nve1
 source 192.168.0.1
 vni 100 head-end peer-list 192.168.0.2
 vni 100 head-end peer-list 192.168.0.3
 vni 200 head-end peer-list 192.168.0.2
 vni 200 head-end peer-list 192.168.0.3
```

# 三、交换机2配置

以下仅仅展示vxlan有关配置。实现loopback接口互通的配置未提供。
```
vlan batch 100 200

bridge-domain 100
 l2 binding vlan 100
 vxlan vni 100
bridge-domain 200
 l2 binding vlan 200
 vxlan vni 200

interface GigabitEthternet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 100 200
 stp disable

interface LoopBack0
 ip address 192.168.0.2 255.255.255.255

interface Nve1
 source 192.168.0.1
 vni 100 head-end peer-list 192.168.0.1
 vni 100 head-end peer-list 192.168.0.3
 vni 200 head-end peer-list 192.168.0.1
 vni 200 head-end peer-list 192.168.0.3
```


# 四、交换机3配置

以下仅仅展示vxlan有关配置。实现loopback接口互通的配置未提供。
```
vlan batch 100 200

bridge-domain 100
 l2 binding vlan 100
 vxlan vni 100
bridge-domain 200
 l2 binding vlan 200
 vxlan vni 200

interface GigabitEthternet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 100 200
 stp disable

interface LoopBack0
 ip address 192.168.0.3 255.255.255.255

interface Nve1
 source 192.168.0.1
 vni 100 head-end peer-list 192.168.0.1
 vni 100 head-end peer-list 192.168.0.2
 vni 200 head-end peer-list 192.168.0.1
 vni 200 head-end peer-list 192.168.0.2
```





***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
