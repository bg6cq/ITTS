# 搭建 NTP 服务器

本文原创：**西北农林科技大学 卞一帆**

修改时间：2017/12/21

## 优点

减少各服务器时间同步时的延迟，让服务器时间更加准确。

## 步骤

### 使用 Docker Container 

#### 安装 Docker

请参考 Docker 官方文档：[https://docs.docker.com/engine/installation/](https://docs.docker.com/engine/installation/)

#### 更换 Docker Hub 镜像源 (可选)

受国内网络环境影响，直接连接 Docker Hub Registry 速度很慢。可以将 Docker Hub Registry 更换为阿里云提供的镜像以提升部署速度。

获取方法：

1. 使用阿里云账号登录 [阿里云容器 Hub 服务控制台](https://cr.console.aliyun.com/)，左面的加速器帮助页面会显示为每个账号独立分配的镜像地址，格式为 `<系统分配前缀>.mirror.aliyuncs.com`。

2. 修改 Docker 配置文件 `/etc/docker/daemon.json` 为：

```
{
    "registry-mirrors": ["<your mirror address>"]
}
```

本方法仅适用于 Docker 1.10 以上版本。老版本需要修改不同的配置文件。

3. 重启 Docker 服务：`sudo systemctl restart docker`

#### 更换 Docker 容器 IP 段

Docker 默认为容器分配 `172.17.0.0/16` 网段的 IP 地址。由于我校内网有线接入分配 IP 段为 `172.16.0.0/12` 会与容器 IP 冲突导致部分 IP 段用户无法访问，因此需要更换。编辑 `/etc/docker/daemon.json` ：
```
{
    "registry-mirrors": ["<your mirror address>"],
    "bip": "192.168.128.1/17"
}
```
保存后应重启 Docker 服务。

#### 拉取镜像

执行 `sudo docker pull cloudwattfr/ntpserver:latest`。

#### 启动容器

执行 `docker run -d --name ntpservice -p 123:123 cloudwattfr/ntpserver`。

参数解释：
- `-d`: 即 `--detach`，使容器以非交互形式在后台运行。
- `--name ntpservice`: 将容器名字指定为 `ntpservice` 。
- `-p 123:123`: 在主机 123 端口与容器 123 端口间建立映射。

### 使用软件包

1. 安装 ntp : `sudo apt-get install -y ntp` (Ubuntu/Debian) 或 `sudo yum install ntp` (CentOS)
2. 配置 `/etc/ntp.conf` ，将上游修改为中国境内的 NTP 服务器:
```
pool 0.cn.pool.ntp.org iburst
```
可以在后面添加多个服务器。

3. 启动 ntp 服务: `sudo systemctl enable ntp && sudo systemctl start ntp`

### 配置防火墙

这里使用 `firewalld` 配置防火墙规则，默认区域为 dmz 。

```
sudo firewall-cmd --zone=dmz --add-service=ntp --permanent
sudo firewall-cmd --reload
```
