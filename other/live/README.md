## [原创] 可扩展视频直播设施建设

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

RTMP编码器有很多，我用的购买自jd.com，[https://item.jd.com/11190135281.html](https://item.jd.com/11190135281.html)，使用时要注意：修改各种参数后需重启一下。

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
apt-get install build-essential libpcre3 libpcre3-dev libssl-dev php5 git

cd /usr/src
git clone https://github.com/arut/nginx-rtmp-module.git
wget http://nginx.org/download/nginx-1.10.3.tar.gz

tar zxvf nginx-1.10.3.tar.gz
cd nginx-1.10.3/

./configure --with-http_ssl_module --add-module=../nginx-rtmp-module
make
make install

cp /usr/src/nginx-rtmp-module/stat.xsl /usr/local/nginx/html

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
            hls_fragment_naming system;
            hls_playlist_length 60;
#            deny play all;
	    on_publish http://localhost:8000/on_publish.php;
        }
    }
}

http {
    include mime.types;
    sendfile off;
    tcp_nopush on;
    #aio on;
    #directio 512;
    default_type application/octet-stream;
    server {
        listen 80;
        location / {
            root /usr/local/nginx/html;
        }
 	location /stat {
            rtmp_stat all;   # NOTE: please copy file `stat.xsl` from `nginx-rtmp-module` to nginx's html folder
            rtmp_stat_stylesheet stat.xsl;
        }
        location /stat.xsl {
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

4. 增加发布用户名和密码检查

如果发布者IP固定，建议使用限制发布者访问1935端口方式控制（删除上面nginx.conf中的`on_publish http://localhost:8000/on_publish.php`)

我使用本机8000端口的on_publish.php来检查user和pass，需要修改`/etc/apache2/ports.conf`把端口改为8000以免与nginx使用的80冲突。并增加
如下的`/var/www/html/on_publish.php`(注意参数是POST过来的，很多的例子GET是错误的)：
```
<?php

$user = isset($_POST['user']) ? $_POST['user'] : '';
$pass = isset($_POST['pass']) ? $_POST['pass'] : '';

if (empty($user) || empty($pass)) {
    echo "wrong query input";
    header('HTTP/1.0 404 Not Found');
    exit();
}

$saveuser = "myuser";
$savepass = "mypass";

if (strcmp($user, $saveuser) == 0 && strcmp($pass, $savepass) == 0) {
    echo "Username and Password OK";
} else {
    echo "Username or Password wrong";
    header('HTTP/1.0 404 Not Found');
    exit();
}

?>
```

5. 测试

执行`/usr/local/nginx/sbin/nginx -t`测试配置是否正确，如果正确，执行
`/usr/local/nginx/sbin/nginx`启动nginx进程。

下载一个 .mp4文件(我下载的是USTCStory.mp4)，使用如下命令行测试通过rtmp协议给nginx：
```
如果有on_publish检查，使用
ffmpeg -re -i USTCStory.mp4 -vcodec libx264 -s 640*480 -vprofile baseline -g 30 -acodec aac -strict -2 -f flv "rtmp://x.x.x.x/live/ustc?user=myuser&pass=mypass"
否则使用
ffmpeg -re -i USTCStory.mp4 -vcodec libx264 -s 640*480 -vprofile baseline -g 30 -acodec aac -strict -2 -f flv rtmp://x.x.x.x/live/ustc
```
如果nginx服务器`/run/shm/hls`有文件生成，说明nginx配置基本正确。

浏览器访问 http://x.x.x.x/stat 可以看到统计信息。

使用支持RTMP的播放器，如vlc，可以通过 rtmp://x.x.x.x/live/ustc 播放。

6. 安全加强

使用iptables保护服务器的端口，仅仅对外开放80端口，其他1935/22等端口对特定IP开放。

# 三、WEB播放页面

假定rtmp推送的是 rtmp://x.x.x.x/live/ustc?user=username&pass=password ，web hls播放的url是 http://x.x.x.x/hls/ustc.m3u8。

请参考 http://live.ustc.edu.cn ，这里有个简单的播放器，可以适配大部分支持html5 video的浏览器。

# 四、系统扩展

以上工作完成后，可以播放，但受限于单台nginx服务器的容量，负载能力有限。

可以增加更多的nginx服务器做缓存和分发，支持更大规模的用户。

以下是做分发的nginx服务器关键配置：

```
#缓存路径，使用 /dev/shm ramdisk，注意内存要大
proxy_temp_path   /dev/shm/tmp 1;
proxy_cache_path  /dev/shm/hls-cache   levels=1:1 keys_zone=hls-cache:100m inactive=2m max_size=3500m;
proxy_cache_path  /dev/shm/m3u8-cache  levels=1   keys_zone=m3u8-cache:1m  inactive=5s max_size=10m;

upstream live-server {
   server x.x.x.x:80;   # 上面配置的rtmp nginx ip
}

server {
#    listen       0.0.0.0:80 ;
    listen       [::]:80 ;
    server_name  live.ustc.edu.cn;   # 域名

    location / {
       proxy_pass http://live-server;
    }

    location ~ \.ts$ {
       proxy_pass http://live-server;
       proxy_cache hls-cache;
       proxy_cache_key $host$uri;
       proxy_cache_valid 200 304 2m;
       proxy_set_header  Host $http_host;
       expires 2m;
   }

   location ~ \.m3u8$ {
       proxy_pass http://live-server;
       proxy_cache m3u8-cache;
       proxy_cache_key $host$uri;
       proxy_cache_valid 200 304 1s;
       proxy_set_header  Host $http_host;
       expires -1;
   }
}
```

# 五、录像保存

如果需要将rtmp实时流保存为flv文件，可以安装rtmpdump：

```
apt-get install rtmpdump

rtmpdump -v -m 0 -r rtmp://x.x.x.x/live/ustc -o output.flv
```


***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
