## [原创]使用Let's encrypt免费SSL证书

本文原创：**中国科学技术大学 张焕杰**

修改时间：2018.03.14

## 一、SSL证书产生过程介绍

1. SSL证书产生过程涉及以下几个概念：

* 服务器密钥：扩展名一般是.key。一般我们使用的是rsa算法，服务器自己生成的一组数为私钥和对应的公钥。私钥需要安全存放，不让其他人知道。

* 证书签名请求：Certificate Signing Request，扩展名一般是.csr。服务器将自己的公钥hash后，加上希望绑定的域名信息，生成.csr文件。

* 证书：扩展名一般是.crt。服务器将.csr文件交给CA机构，CA机构验证服务器真实拥有该域名后，用CA机构的私钥对这些信息签名，生成.crt文件。

* 证书链：CA机构的证书是安装到客户机系统的，为了安全起见，CA机构会为自己的二级CA签发证书，最后由该二级CA对普通用户签发证书，证书链是记录该关系的。不提供证书链可以工作，但有时候会有告警。

2. CA服务机构

CA服务机构的存在是做为公认的第三方来验证服务器身份，这中间可能需要收取服务费。

为了方便使用，可以选择Let's encrypt免费CA机构签发证书。Let's encrypt已被广泛接受，申请证书也比较快捷，一般来说5分钟内可以完成从开始安装程序到申请证书过程。需要说明的是Let's encrypt签发的证书有效期是90天，在到期之前需完成证书更新。

3. Let's encrypt免费证书颁发过程

* 服务器生成私钥，产生.csr文件
* 服务器将.csr文件交给Let's encrypt服务器
* Let's encrypt服务器提供一个随机字符串，要求放在网站的.well-known/acme-challenge目录下 或者 在DNS上进行发布
* 服务器将随机字符串放在.well-known/acme-challenge目录下 或通过DNS发布
* Let's encrypt服务器通过http或DNS访问到该随机字符串，验证服务器拥有该域名，颁发证书

## 二、Let's encrypt 证书生成工具

有很多种Let's encrypt 证书生成工具，这里介绍完全由shell脚本完成整个过程的acme.sh [https://github.com/Neilpang/acme.sh](https://github.com/Neilpang/acme.sh)。

以下过程使用泛域名*.ustc.edu.cn演示。

1. 安装过程

我没有安装，仅仅是下载：
````
cd /usr/src/
git clone https://github.com/Neilpang/acme.sh.git
cd /acme.sh
````

2. 获取证书

执行命令获取证书
````
./acme.sh --issue --dns  -d *.ustc.edu.cn
````
执行后退出，提示有：
```
Add the following txt record:
Domain:_acme-challenge.ustc.edu.cn
Txt value:9ihDbjYfTExAYeDs4DBUeuTo18KBzwvTEjUnSwd32-c

```
这时，修改DNS记录，增加

```
_acme-challenge.ustc.edu.cn IN	TXT "9ihDbjYfTExAYeDs4DBUeuTo18KBzwvTEjUnSwd32-c"
```
然后继续获取证书的过程，注意下面的命令行中的"renew"
```
./acme.sh --renew  -d *.ustc.edu.cn
```

3. 产生nginx需要的证书

以上执行过程，会在 ~/.acme.sh 产生一些中间文件（包括证书的信息），使用如下命令可以生成nginx需要的两个文件：

```
acme.sh --install-cert -d *.ustc.edu.cn \
--key-file       /etc/nginx/ssl/ustc.edu.cn.key  \
--fullchain-file /etc/nginx/ssl/ustc.edu.cn.pem
```

4. 证书使用

nginx.conf的配置如下所示
````
        server {
                listen 80;
                listen [::]:80;
                listen 443 ssl;
                listen [::]:443 ssl;
                ssl_certificate /etc/nginx/ssl/ustc.edu.cn.pem;
                ssl_certificate_key /etc/nginx/ssl/ustc.edu.cn.key;
                ssl_session_cache shared:SSL:1m;
                ssl_session_timeout  10m;
                ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
                ssl_prefer_server_ciphers on;
                ssl_dhparam /etc/nginx/ssl/dhparam.pem;
                server_name ustcnet.ustc.edu.cn;
                access_log /var/log/nginx/host.ustcnet.ustc.edu.cn.access.log;
                location / {
                        proxy_pass http://202.38.64.99/;
                }
        }
````
使用以上配置，https://www.ssllabs.com/ssltest/analyze.html?d=ustcnet.ustc.edu.cn 测试评分是A (IPv4/IPv6双栈)


5. 关联邮件

以上已经可以工作了，但建议使用
```
./acme.sh --updateaccount --accountemail xyz@xya.edu.cn
```
关联自己的邮箱，这样证书快过期时可以收到提醒邮件。


***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
