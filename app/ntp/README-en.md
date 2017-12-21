# Set up a NTP server

Author: **Yifan Bian, Northwest A&F University**

Last modified: 2017/12/21

## Advantages

A private NTP server can reduce delay of synchronizing time on servers and make time on servers more accurate.

## Steps

### Use Docker Container 

#### Install Docker

Please refer to official documentation of Docker: [https://docs.docker.com/engine/installation/](https://docs.docker.com/engine/installation/).

#### Change Docker Hub Registry (Optional)

Affected by domestic network environment, it will be very slow to directly connect to Docker Hub Registry. You may want to accelerate the speed by changing it to the mirror provided by Aliyun.

Steps:

1. Sign in [Aliyun Container Hub Service Console](https://cr.console.aliyun.com/) with your Aliyun account. Your mirror address will be shown in the help page and seem like `<prefix>.mirror.aliyuncs.com`.

2. Modify the configuration of Docker (`/etc/docker/daemon.json`):
```
{
    "registry-mirrors": ["<your mirror address>"]
}
```
This can only apply to Docker with version greater than 1.10.

3. Restart Docker service: `sudo systemctl restart docker`

#### Modify IP range of Docker containers

By default, Docker will assign addresses in `172.17.0.0/16` to containers. Because it will make collisions when a user from West Student Dorm Area of the North campus attempting to access the server, we have to modify that. Modify `/etc/docker/daemon.json`:
```
{
    "registry-mirrors": ["<your mirror address>"],
    "bip": "192.168.128.1/17"
}
```
A restart of Docker service is required after you modified that.

#### Pull the image

Execute `sudo docker pull cloudwattfr/ntpserver:latest`.

#### Start the container

Execute `docker run -d --name ntpservice -p 123:123 cloudwattfr/ntpserver`.

Explanation of parameters:
- `-d`: aka `--detach`, makes the container run in the background.
- `--name ntpservice`: name the container
- `-p 123:123`: establish a map between host's port 123 and container's port 123.

### Use software package of your distro

1. Install ntp: `sudo apt-get install -y ntp` (Ubuntu/Debian) or `sudo yum install ntp` (CentOS)
2. Modify `/etc/ntp.conf` and set the upstream to ntp servers in China:
```
pool 0.cn.pool.ntp.org iburst
```
You can add other servers after that.

3. Start the ntp service: `sudo systemctl enable ntp && sudo systemctl start ntp`

### Configure the firewall

I use `firewalld` to configure the firewall. The default zone of `firewalld` has been set to `dmz`.

```
sudo firewall-cmd --zone=dmz --add-service=ntp --permanent
sudo firewall-cmd --reload
```
