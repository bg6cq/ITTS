## [原创]nginx反向代理服务器

本文原创：**中国科学技术大学 张焕杰**

参与修改：  
**审计大学 吴鑫**
**西安财经学院 王伟**

修改时间：2017.10.09

## 一、反向代理服务器简介

相对于传统的为用户提供网络访问的代理服务器不同，反向代理服务器主要是为web服务器对外提供服务使用。

高校网络使用反向代理服务器，会提供如下优势：

* 内部服务器更安全。只需要反向代理服务器对外即可，内部服务器可以藏起来。

* 容易串接WAF设备。只要在反向代理服务器对内的接口串接WAF即可保护所有网站。

* 多入口更容易实施。反向代理服务器上可以连接多个ISP的入口。

最简单、最常用的反向代理服务器是nginx。

## 二、多入口环境

假定服务器有3个ISP入口，分别是：

* eth0，CERNET，IP 202.38.64.2/24，网关202.38.64.1，这个入口同时是服务器对外访问的默认出口
* eth1，电信，IP 218.22.21.2/24，网关218.22.21.1
* eth2，联通，IP 218.104.71.2/24，网关218.104.71.1

以下文件 /etc/rc.d/rc.local 可以设置好多入口环境，确保从一个ISP接口进来的数据包，其应答包原路返回：
````
ip addr add 202.38.64.2/24 dev eth0
ip addr add 218.22.21.2/24 dev eth1
ip addr add 218.104.71.2/24 dev eth2

ip link set eth0 up
ip link set eth1 up
ip link set eth2 up

ip route add 0/0 via 202.38.64.1

ip route add 0/0 via 202.38.64.1 table 100
ip route add 0/0 via 218.22.21.1 table 110
ip route add 0/0 via 218.104.71.1 table 120

ip route add from 202.38.64.2 table 100 pref 100
ip route add from 218.22.21.2 table 110 pref 100
ip route add from 218.104.71.2 table 120 pref 100
````

## 三、nginx配置
````
user  nginx;
worker_processes  4;
worker_rlimit_nofile 10240;
pid        /var/run/nginx.pid;

error_log   /var/log/nginx/error.log;

events {
        worker_connections  10240;
}

http {
        server_names_hash_bucket_size  200;
        server_tokens off;

        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '"$status" $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
        log_format  nohost  '$host $remote_addr [$time_local] "$request" '
                      '"$status" $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
        access_log  /var/log/nginx/access.log  main;

        sendfile       on;
        tcp_nopush     on;

        keepalive_timeout  75 20;

        client_max_body_size       100m;
        client_body_buffer_size    128k;
        client_body_temp_path      /var/nginx/client_body_temp;
        proxy_buffering off;
        proxy_connect_timeout      10;
        proxy_send_timeout         900;
        proxy_read_timeout         900;
        #proxy_send_lowat          12000;
        proxy_buffer_size          4k;
        proxy_buffers              4 32k;
        proxy_busy_buffers_size    64k;
        proxy_temp_file_write_size 64k;
        proxy_temp_path            /var/nginx/proxy_temp;
        proxy_redirect     off;

        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;

	# 默认站点，使用IP或非授权域名访问时出现错误提示
        server {
                listen 80 default_server;
                server_name _;
                access_log /var/log/nginx/host.nohost.ustc.edu.cn.access.log nohost;
                root /etc/nginx/nohost;
                default_type 'text/html';
                location / {
                        return 404 "<html><head><meta http-equiv=\"Content-Type\" content=\"text/html; charset=utf-8\"><title>404 Unknown Virtual Host</title><style>div{font-size:12px;}a{color:#1463da;text-decoration:none}a:hover{color:#1463da;text-decoration:underline; }</style></head><body bgcolor=\"white\"><center><h1>404 Unknown Virtual Host</h1></center><p><center>client: $remote_addr, request_host : $host, time: $date_local</center><hr><p><center><h1>您访问的域名不存在或已被关闭</h1></center><p><center>如有问题请联系<a href=http://ustcnet.ustc.edu.cn>中国科学技术大学网络信息中心</a></center></body></html><!-- a padding to disable MSIE and Chrome friendly error page --><!-- a padding to disable MSIE and Chrome friendly error page --><!-- a padding to disable MSIE and Chrome friendly error page --><!-- a padding to disable MSIE and Chrome friendly error page --><!-- a padding to disable MSIE and Chrome friendly error page --><!-- a padding to disable MSIE and Chrome friendly error page -->";
                }
        }

	# 跳转的域名，访问时直接跳转到特定的URL
     	server {
                listen 80;
                server_name giving.ustc.edu.cn;
                access_log /var/log/nginx/host.giving.ustc.edu.cn.access.log;
                location / {
                        rewrite ^ http://www.ustcif.org/WaysToGive/ ;
                }
        }

	# 普通的站点
        server {
                listen 80;
                server_name www.ustc.edu.cn;
                access_log /var/log/nginx/host.www.ustc.edu.cn.access.log;
                location / {
                        proxy_pass http://202.38.64.99/;
                }
        }

}
````

对于 CentoS7，启动nginx时 若出现出现错误: `setrlimit(RLIMIT_NOFILE, 10240) failed (1: Operation not permitted) `

先查看目前系统的设定值
`
ulimit -n
`

若设定值太小，修改 /etc/security/limits.conf  
vi /etc/security/limits.conf  
加上或修改以下两行设定
````
* soft nofile 65535
* hard nofile 65535
````

## 四、后台站点配置

使用以上配置时，nginx 访问后台站点时，自动设置了Host: 头为server_name，因此后台站点需要处理对应的server_name域请求。

还有一种使用方式，假定 http://www.ustc.edu.cn 是nginx对外提供服务的域名，而后台站点使用 http://p-www.ustc.edu.cn （p-前缀代表是后台站点）。
这时nginx可以采用如下配置方式：
````
......
        proxy_set_header   Host             $proxy_host;
......
        # 普通的站点
        server {
                listen 80;
                server_name www.ustc.edu.cn;
                access_log /var/log/nginx/host.www.ustc.edu.cn.access.log;
                location / {
                        proxy_pass http://p-www.ustc.edu.cn;
                }
        }
````
感谢 **西财 王伟** 老师指出的$host $proxy_host区别。

补充：

发现一个问题，如下配置
````
	location / {
		proxy_pass http://p-www.ustc.edu.cn
	}
````	
如果目标主机的主页是自动刷新跳转的，或者是登录页面验证后跳转的，那么浏览器会跳转到 `http://p-www.ustc.edu.cn`
原因还没有完全分析明白，目前的解决方案是上面的代码补充为：
````
	location / {
		proxy_pass http://p-www.ustc.edu.cn;
		proxy_redirect	http://p-www.ustc.edu.cn/ /;
	}
````
**需要注意**

采用$proxy_host，会带来一些问题。

proxy_host传递给后台站点的主机头是 `p-www.ustc.edu.cn` ， 这样的主机头一般情况是不希望用户之间看到，或者根本就是不存在的域名。
如果一些网站有自动跳转（js跳转、meta refresh等等）语句，将跳转到 `p-www.ustc.edu.cn` 。 如果域名不存在，将返回404错误。

所以一般情况仍建议使用$host。除非了解，不要使用$proxy_host。

**一个现实的例子**

一台服务器上运行了30个网站，分别绑定了不同的域名，现在要做反向代理，需要如何实现？

其实过程很简单：

* DNS解析全部执行Nginx服务器

* Nginx中写hosts文件，将这些域名指向后台服务器的ip

* Nginx中使用$host参数，然后每个网站的配置方式：
````
······
   proxy_pass http://www.ustc.edu.cn;#真实指向Nginx的那个域名
······
````
* **后台站点什么操作都不用做**

**再次注意**

* $proxy_host和$host的区别

* proxy_pass地址的不同

````
······
   proxy_pass http://www.ustc.edu.cn;
······
````
和

````
······
   proxy_pass http://www.ustc.edu.cn/;
······
````
的区别。

## 五、HTTPS站点的配置
可以使用Nginx作为反向代理，在源站点无需任何修改的情况下，将普通站点转换为HTTPS站点。
在server段需要增加
````
......
        listen 443 ssl;
	ssl_certificate /etc/letsencrypt/live/yousite/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/yousite/privkey.pem;
	ssl_session_timeout 5m;
	add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
......
````
此时，通过Nginx访问该站点，浏览器会自动将HTTP协议切换为HTTPS协议。

关于HTTPS证书的申请，请参考https://github.com/bg6cq/ITTS/tree/master/security/ssl/letsencrypt

## 六、Nginx充当反向代理时设置自定义403页面

````
server {
     listen 80;
     server_name  site;

     proxy_intercept_errors on;
     error_page 403   /403.html;

     location = /403.html {
        root	/data/html;
     }
 
     location / {
        allow 10.0.0.0/8;
        allow 172.16.0.0/12;
        deny all;
        proxy_pass_header Server;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;
        proxy_pass http://site;
        recursive_error_pages   on;
     }
}
````

中科大的反向代理上，有若干服务器仅仅允许校内IP访问，做法是：

文件/etc/nginx/ustc.conf
````
allow 114.214.160.0/19;
allow 114.214.192.0/18;
allow 202.38.64.0/19;
allow 210.45.64.0/20;
allow 210.45.112.0/20;
allow 211.86.144.0/20;
allow 222.195.64.0/19;

allow 218.22.21.0/29;
allow 202.141.160.0/19;
allow 202.104.71.160/28;

allow 210.72.22.0/24;

deny all;
````

文件 /etc/nginx/html/403.html
````
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>403 Access denied</title>
<style>
div{font-size:12px;}
a{color:#1463da;text-decoration:none}
a:hover{color:#1463da;text-decoration:underline;}
</style>
</head>
<body bgcolor="white">
<center><h1>禁止被访问/Access denied!</h1></center>
<p>
<center><h1>您访问的域名只能从中国科大校内访问</h1></center><p>
<center>We are sorry you are denied to access this URL</center><p>
<center>请单击<a href=http://revproxy.ustc.edu.cn:8000/yourip.php>这里</a>获取您的IP信息</center><p>
<center>如果您在中国科大校内上网，出现本页提示，请检查浏览器是否使用了代理服务器</center><p>
<center>如有问题请联系<a href=http://ustcnet.ustc.edu.cn>中国科学技术大学网络信息中心</a></center>
</body>
</html>
````

文件nginx.conf
````
	server {
		listen 80;
		server_name image.ustc.edu.cn;
		access_log /var/log/nginx/host.image.ustc.edu.cn.access.log;
		error_page 403   /403.html;
 		location = /403.html {
 			root	/etc/nginx/html;
		}
		location / {
			include /etc/nginx/ustc.conf;
			proxy_pass http://202.38.64.115/;
		}
	}
````

## 七、专业支持的系统


如果觉得以上操作太麻烦，强烈建议购买专业支持的系统，运行起来省事省心，界面高大上，如网瑞达的产品除了提供反向代理外，还提供了VPN等更多功能：

感兴趣的请看视频 [网瑞达 资源安全访问解决方案](http://v.wrdtech.com/vod-show?id=66)


***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
