## [原创]nginx反向代理服务器

本文原创：**中国科学技术大学 张焕杰**

参与修改：  
**审计大学 吴鑫**  
**西安财经学院 王伟**  
**中国科学技术大学 高一凡**

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

* DNS解析全部指向Nginx服务器

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
	add_header Strict-Transport-Security "max-age=31536000" always;
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

## 七、统一反向代理配置

当反向代理服务器同时服务众多网站时，管理配置将变得十分繁琐。为了方便维护，可以借助映射("map")统一反向代理配置。

### 1. 一个基本的例子

```
map $http_host $upstream {
	default			nil;
	www.ustc.edu.cn		www-p.ustc.edu.cn;
	bbs.ustc.edu.cn		10.38.95.2;
	// more rules here ...
}

server {
	listen 80;
	listen 443 ssl http2;
	server_name _;

	if ($upstream = nil) {
		return 404;
	}

	location / {
		proxy_pass	http://$upstream$request_uri;
		proxy_set_header Host $http_host;
	}
}
```

*注：上例中省略了http配置块。*

映射("map")是一个惰性求值器，与数学中的[映射](https://baike.baidu.com/item/%E6%98%A0%E5%B0%84/20402621)极为相似。nginx在处理HTTP请求时，如果使用到了映射，则会触发一次求值。

`$http_host`是nginx的一个内建变量，他的值总是等于用户请求头的主机名("Host")字段。`$request_uri`是另一个内建变量，值为请求的URI部分（包含参数）。

在上例中，定义了一个映射`$upstream`，映射的变量是`$http_host`，值为`$upstream`（即：后端的地址），默认值为`nil`。 

举例说明：对于请求`http://www.ustc.edu.cn/index.html`。

* `$http_host`的值为`www.ustc.edu.cn`, 那么映射`$upstream`就会被赋予`www-p.ustc.edu.cn`。
* `$request_uri`的值为`/index.html`。
* `proxy_pass http://$upstream$request_uri;` 将这条指令中的变量用字面值代替，就变为了：`proxy_pass http://www-p.ustc.edu.cn/index.html`。
* `proxy_set_header Host $http_host;` 这条指令的作用是：在访问后端时使用用户发来的Host（即`www.ustc.edu.cn`），而非`www-p.ustc.edu.cn`。

如果用户请求了一个不在映射内的地址，如`http://unknown.ustc.edu.cn/`，则映射`$upstream`被赋予默认值`nil`，因此会被判断指令`if ($upstream = nil)`捕获，返回404错误。

### 2. 更加丰富的功能

同理，可以借助配置不同的映射来实现不同的功能，如：

* 限制来源IP
* 强制使用HTTPS（HTTP->HTTPS跳转）
* 301站点跳转

但受限于 nginx 的 if 指令的实现方式，部分功能无法在纯 nginx 配置中实现。为了解决这个问题， nginx 官方引入了 lua 扩展，可以使用 lua 语言扩展出许多灵活的特性。

下面是一个更加丰富的例子：

```
map $http_host $upstream {
	default				nil;
	internal.ustc.edu.cn		internal-p.ustc.edu.cn;
	mail.ustc.edu.cn		10.38.95.2;
	test.ustc.edu.cn		192.168.95.100/test;
	// more rules here ...
}

map $http_host $ustcnet_only {
	default				0;
	internal.ustc.edu.cn		1;
}

map $http_host $https_only {
	default				0;
	mail.ustc.edu.cn		1;
}

map $http_host $upstream_scheme {
	default				"http";
	test.ustc.edu.cn		"https";
}

map $https_only $redirect_replacement {
	default				"http";
	1				"https";
}

map $http_host $redirect {
	default 			nil;
	overdue.ustc.edu.cn		new.ustc.edu.cn;
}

geo $ustcnet {
    default 0;
    202.38.64.0/19 1;
    210.45.64.0/20 1;
    210.45.112.0/20 1;
    211.86.144.0/20 1;
    222.195.64.0/19 1;
    114.214.160.0/19 1;
    114.214.192.0/18 1;
    210.72.22.0/24 1;
    218.22.21.0/27 1;
    202.141.160.0/20 1;
    202.141.176.0/20 1;
    121.255.0.0/16 1;
}

server {
	listen 80;
	listen [::]:80;
	listen 443 ssl http2;
	listen [::]:443 ssl http2;
	server_name _;

	if ($redirect != nil) {
		return 301 $scheme://$redirect$request_uri;
	}

	if ($upstream = nil) {
		return 404;
	}

	access_by_lua_block {
		-- block non-ustcnet user
		if ngx.var.ustcnet == "0" and
		   ngx.var.ustcnet_only == "1"
		then
			ngx.exit(ngx.HTTP_FORBIDDEN)
		end

		-- redirect from HTTP to HTTPS
		if ngx.var.scheme == "http" and
		   ngx.var.https_only == "1"
		then
			return ngx.redirect("https://" .. ngx.var.http_host .. ngx.var.request_uri, ngx.HTTP_MOVED_PERMANENTLY)
		end
	}

	location / {
		proxy_pass	http://$upstream$request_uri;
		proxy_set_header Host $http_host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Forwarded-Proto $scheme;
	}
}
```

对各个映射做一些说明：

* upstream：反向代理的后端地址。默认值为`nil`。
* ustcnet_only：仅限科大校内地址访问。默认允许所有IP。
* https_only：仅限使用HTTPS协议访问，将HTTP请求跳转为HTTPS。默认不跳转。
* upstream_scheme：访问后端服务器使用的协议。默认为http。
* redirect_replacement：对后端30x跳转指令的协议修正。无需手动更改。
* redirect：对整个域名做301跳转。

`access_by_lua_block`指令需要 nginx 模块 [lua-nginx-module](https://github.com/openresty/lua-nginx-module) ，且版本大于v0.9.17。如果使用 Debian 系的发行版，`nginx-extras`这个软件包中已经包含了我们需要的一切模块，只需要执行`apt install nginx-extras`即可。

作为另一种选择，可以使用 [openresty](https://openresty.org/) 这个 nginx 发行版，提供了几乎所有常见系统的 nginx 预编译软件包。

上面的例子只是抛砖引玉，nginx + lua 能实现十分丰富的功能，可以与数据库通讯，可以生成响应体，也可以发起新的请求，且能承载极大量的并发访问。甚至有许多公司使用这种方式构建复杂的商业服务。

## 八、CentOS 6.9 nginx支持http2

参考 https://linwm.com/16.html

CentOS 6.9 系统安装默认的nginx，由于openssl版本低，不支持http/2，使用如下步骤重新编译支持http/2的nginx，放在
`/usr/local/nginx/sbin/nginx`，并将`/etc/init.d/nginx`中的可执行文件命令替换，系统的其他部分不做任何修改。

```
1. 准备
yum update
yum upgrade
yum install -y patch libtool gcc gcc-c++ autoconf automake zlib zlib-devel pcre-devel make unzip git wget libxslt-devel

2. 下载openssl
cd /usr/src
wget https://www.openssl.org/source/openssl-1.1.0h.tar.gz
tar zxvf openssl-1.1.0h.tar.gz

3. 下载 nginx 安装包
cd /usr/src
wget http://nginx.org/download/nginx-1.14.0.tar.gz
tar zxvf nginx-1.14.0.tar.gz
cd nginx-1.14.0

4. 开始执行编译安装
除了安装的位置，禁用perl、geoip，其余都是用系统默认
./configure --prefix=/usr/local/nginx --with-openssl=/usr/src/openssl-1.1.0h  --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi --pid-path=/var/run/nginx.pid --lock-path=/var/lock/subsys/nginx --user=nginx --group=nginx --with-file-aio --with-ipv6 --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_addition_module --with-http_xslt_module=dynamic --with-http_image_filter_module=dynamic --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_slice_module --with-http_stub_status_module --with-mail=dynamic --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream=dynamic --with-stream_ssl_module --with-debug --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic' --with-ld-opt=' -Wl,-E'

make
mkdir -p /usr/local/nginx/sbin/
cp objs/nginx /usr/local/nginx/sbin

vi /etc/init.d/nginx, 把
nginx="/usr/sbin/nginx"
修改为
nginx="/usr/local/nginx/sbin/nginx"
```

## 九、平滑切换至https

目前还有大约2%左右windows xp设备，这些设备对https支持有限，往往无法访问https网站，如果把所有http访问强制
定向成https，会导致这些用户无法使用。

为了解决平滑切换，可以采用如下方法：

在nginx中配置如下，当访问/dummy_hsts时返回空内容，但是有个特殊的HTTP header: Strict-Transport-Security "max-age=604800"。

```

http {

        map $scheme $hsts_header {
                https   "max-age=604800";
        }

...

server {
	listen 80 ;
	listen [::]:80 ;
	listen 443 ssl http2;
	listen [::]:443 ssl http2;
	ssl_certificate /etc/nginx/ssl/ustcnet.ustc.edu.cn.pem;
	ssl_certificate_key /etc/nginx/ssl/ustcnet.ustc.edu.cn.key;
	ssl_session_cache shared:SSL:1m;
	ssl_session_timeout  10m;
	ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
	ssl_prefer_server_ciphers on;
	ssl_dhparam /etc/nginx/ssl/dhparam.pem;
	server_name ustcnet.ustc.edu.cn;
	access_log /var/log/nginx/host.ustcnet.ustc.edu.cn.access.log main;
	add_header Strict-Transport-Security $hsts_header;
	location / {
		proxy_pass http://x.x.x.x/;
	}
	location /dummy_hsts {
		return 200 "";
	}
}
```

在 http://ustcnet.ustc.edu.cn 的第一个页面增加如下代码
```
<iframe src="https://ustcnet.ustc.edu.cn/dummy_hsts" width="0" height="0" frameborder="0"></iframe>
```

其工作原理是：

1. 浏览器访问 http://ustcnet.ustc.edu.cn 时，由于iframe会去访问https://ustcnet.ustc.edu.cn/dummy_hsts
2. 如果访问https://ustcnet.ustc.edu.cn/dummy_hsts成功，会得到HTTP Header: Strict-Transport-Security "max-age=604800"
3. 对于现代浏览器，在604800秒，即7天内对http://ustcnet.ustc.edu.cn的访问，都会强制使用https
4. 对于每次https访问，都会返回HTTP Header: Strict-Transport-Security "max-age=604800"，也就是只要7天内访问过，一直会用https

如果经测试工作稳定，可以将上面的604800换成一个足够大的值。

## 十、专业支持的系统

如果觉得以上操作太麻烦，强烈建议购买专业支持的系统，运行起来省事省心，界面高大上，如网瑞达的产品除了提供反向代理外，还提供了VPN等更多功能：

感兴趣的请看视频 [网瑞达 资源安全访问解决方案](http://v.wrdtech.com/vod-show?id=66)

***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
