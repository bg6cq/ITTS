## [原创] 简单视频直播设施建设

本文原创：**中国科学技术大学 张焕杰**

修改时间：2017.12.20

本文记录一个RTMP直播系统的建设。

本文建设的RTMP直播系统，包含如下部件：

1. RTMP编码器

用于将HDMI信号，压缩编码后使用RTMP协议传送给RTMP服务器(nginx)。

2. RTMP服务器(nginx)

将RTMP协议收到的视频流，拆分成.ts文件，并更新对应的.m3u8索引文件。

把这些文件使用http协议提供给客户端。

3. 客户端

使用浏览器使用HTTP协议访问nginx，获取.ts文件和.m3u8索引文件，播放视频。

 
# 一、RTMP编码器

RTMP编码器有很多，我用的购买自jd.com，[https://item.jd.com/11190135281.html](https://item.jd.com/11190135281.html)。

设置IP地址，改为RTMP模式，推送地址为  rtmp://x.x.x.x/live/ustc   （x.x.x.x是下面安装的RTMP服务器IP地址）

# 二、RTMP服务器(nginx)的安装过程

安装时参考`https://docs.peer5.com/guides/setting-up-hls-live-streaming-server-using-nginx/`

安装过程：

1. 有人称14.0版本比较容易安装，我使用ISO文件 http://mirrors.ustc.edu.cn/ubuntu-cdimage/releases/14.04.5/release/ubuntu-14.04.5-server-amd64%2Bmac.iso 安装系统。

2. 系统安装后执行命令编译安装带rtmp支持的nginx

```
sudo bash
apt-get update
apt-get upgrade
apt-get install build-essential libpcre3 libpcre3-dev libssl-dev git

cd /usr/src
git clone https://github.com/arut/nginx-rtmp-module.git
wget http://nginx.org/download/nginx-1.10.3.tar.gz

tar zxvf nginx-1.10.3.tar.gz
cd nginx-1.10.3/

./configure --with-http_ssl_module --add-module=../nginx-rtmp-module
make
make install

add-apt-repository ppa:mc3man/trusty-media
sudo apt-get update
sudo apt-get install ffmpeg
```

3. `/usr/local/nginx/conf/nginx.conf`内容

```
worker_processes  auto;

events {
    worker_connections  10240;
}

rtmp {
    server {
        listen 1935;
        chunk_size 4000;
        application live {
            live on;
            hls on;
            hls_path /run/shm/hls;
            hls_fragment 3;
            hls_playlist_length 60;
            deny play all;
        }
    }
}

http {
    include mime.types;
    sendfile off;
    tcp_nopush on;
    #aio on;
    directio 512;
    default_type application/octet-stream;
    server {
        listen 80;
        location / {
            root /usr/local/nginx/html;
        }
        location /hls {
            types {
                application/dash+xml mpd;
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }
            root /run/shm/;
        }
    }
}
```

4. 测试

执行`/usr/local/nginx/sbin/nginx -t`测试配置是否正确，如果正确，执行
`/usr/local/nginx/sbin/nginx`启动nginx进程。

下载一个 .mp4文件(我下载的是USTCStory.mp4)，使用如下命令行测试通过rtmp协议给nginx：
```
ffmpeg -re -i USTCStory.mp4 -vcodec libx264 -s 640*480 -vprofile baseline -g 30 -acodec aac -strict -2 -f flv rtmp://x.x.x.x/live/ustc
```
如果nginx服务器`/run/shm/hls`有文件生成，说明nginx配置基本正确。

5. 安全加强

使用iptables保护服务器的端口，仅仅对外开放80端口，其他1935/22等端口对特定IP开放。

# 三、播放页面

请参考 http://live.ustc.edu.cn 


***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
