## [原创]如何拦截恶意解析域名的访问

本文原创：**中国科学技术大学 张焕杰**

修改时间：2026.11.28

要禁止未备案或未授权的恶意域名解析到你的服务器，核心思路是利用 Web 服务器的“默认虚拟主机”机制，将未明确绑定的域名请求全部拦截或丢弃。这样，即使恶意域名解析到了你的 IP，用户也无法通过该域名访问到你的网站内容，从而避免因“未备案域名接入”而导致的封 IP 风险。

## 一、Nginx 配置方法

Nginx 的处理逻辑是：当请求的 Host 头（即域名）没有匹配到任何已配置的 server_name 时，Nginx 会将其交给标记了 default_server 的块处理。

###1. 添加默认拦截块

在 Nginx 配置（通常在 nginx.conf 的 http 块内，或 conf.d/ 目录下）中，在其他所有 server 块之前，添加以下两个专门用于拦截的 server 块：
```
# 拦截 HTTP (80端口) 的未授权域名及 IP 直连
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;
    # 返回 444 会让 Nginx 直接关闭连接，不发送任何响应头，对恶意扫描最节省资源
    return 444; 
}

# 拦截 HTTPS (443端口) 的未授权域名及 IP 直连
server {
    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;
    server_name _;
    
    # 这里需要随便填一个证书（可以是自签的，或者你已有的任意域名证书），因为 SSL 握手需要
    ssl_certificate /path/to/any_cert.pem; 
    ssl_certificate_key /path/to/any_key.pem;
    
    return 444;
}
```

### 2. 注意事项

移除冲突标记：如果你的其他正常网站配置中也写了 default_server，需要先将其删掉，否则 Nginx 启动会报错（端口重复监听）。

效果：配置后，直接用 IP 访问，或用任何未在你的 server_name 中列出的域名访问，都会被直接断开连接。


## 二、Apache 配置方法

Apache 的处理逻辑是：它会将第一个定义的虚拟主机作为默认站点。因此，你需要把“拒绝访问”的配置放在最前面。

### 1. 添加默认拒绝虚拟主机

在 Apache 的主配置（如 httpd.conf 或 sites-available/000-default.conf）中，确保以下 <VirtualHost> 块出现在你所有正常网站配置的最前面：
```
# 默认拦截块：捕获所有未明确绑定的域名和 IP 直连
<VirtualHost *:80>
    ServerName default-catch-all
    DocumentRoot /var/www/html # 可以指向一个空目录
    
    <Location />
        # 旧版 Apache 使用 Order/Deny，新版 (2.4+) 推荐使用 Require
        Require all denied
    </Location>
</VirtualHost>
```

### 2. 正常网站配置
在拦截块之后，再配置你正常的网站：
```
<VirtualHost *:80>
    ServerName www.yourdomain.com
    ServerAlias yourdomain.com
    DocumentRoot /var/www/your_site
    # ... 其他正常配置 ...
</VirtualHost>
```

### 3. 补充说明

HTTPS 端口：如果你开启了 HTTPS，同样需要为 *:443 端口配置一个放在最前面的默认拦截 <VirtualHost>，并指向对应的 SSL 证书（哪怕是不匹配的默认证书），并在内部设置 Require all denied。

旧版指令：如果你使用的是 Apache 2.2 等较老版本，可以使用 Order Allow,Deny 和 Deny from All 代替 Require all denied。


***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
