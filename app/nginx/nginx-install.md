## [原创]step-by-step install nginx反向代理服务器(unbutu 18.04 LTS)

本文原创：

* **中国科学技术大学 张焕杰**
* **厦门大学 郑海山**

修改时间：2018.06.13

## 一、unbutu 18.04 LTS安装

获取安装包 iso，您可以从以下站点获取 `ubuntu-18.04-live-server-amd64.iso`，文件大小大约是806MB。

* [中国科大镜像站](http://mirrors.ustc.edu.cn/ubuntu-releases/18.04/)
* [上海交大镜像站](http://ftp.sjtu.edu.cn/ubuntu-cd/18.04/)
* [163镜像站](http://mirrors.163.com/ubuntu-releases/18.04/)

说明：这里还有个安装程序，更加灵活，熟练人士可以选择 [中国科大镜像站](http://mirrors.ustc.edu.cn/ubuntu-cdimage/releases/18.04/release/)。

使用物理服务器或新建虚拟机，如果使用虚拟机，选择4个虚拟CPU，2G内存，40G硬盘一般就够用，类型可以选ubuntu 64bit。

使用光盘镜像引导，安装即可，一般在10分钟内完成。如果有疑问，可以参考 
[Ubuntu 18.04 Server 版安装过程图文详解](https://blog.csdn.net/zhengchaooo/article/details/80145744)，如果安装时没有设置网络，请参见下面的 配置网络部分。

安装完的系统占用磁盘空间为3.5G（可以用`df`查看）。

注意：Ubuntu 系统要求必须使用一个普通用户登录，执行需要特权的命令时，使用`sudo ....`来临时切换为root用户进行。如果需要以root身份执行较多的命令，可以使用`sudo su -`切换为root用户（虽然不建议这样做），这样一来就不需要每次输入`sudo`了。

## 二、配置网络

反向代理服务器需要IPv4/IPv6的连通性，对外需要开放22、80、443端口，如果您有防火墙，请放开这些端口。

使用安装时设置的普通用户登录系统，使用以下命令测试网络是否正常：
```
ip add    #查看网卡设置的ipv4/ipv6地址
ip route  #查看ipv4网关
ip -f inet6 route #查看ipv6网关
ping 202.38.64.1  #检查ipv4连通性
ping6 2001:da8:d800::1 #检查ipv6连通性
```
如果网络存在问题，请按照以下说明修改配置，直到网络正常。

Ubuntu网络配置与之前的变化较大，采用netplan管理，配置文件存放在`/etc/netplan/*.yaml`。

下面是我使用的例子，文件是`/etc/netplan/50-cloud-init.yaml`，内容如下：

请根据自己的网络情况，修改文件，修改后执行`sudo netplan apply`应用即可。

```
network:
    version: 2
    ethernets:
        ens160:
            dhcp4: no
            addresses: [222.195.81.200/24,'2001:da8:d800:381::200/64']
            gateway4: 222.195.81.1
            nameservers:
                    addresses: [202.38.64.1,202.38.64.56]

```


## 三、安装nginx

执行`sudo apt-get install -y nginx`即可。









***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
