## [原创]使用Let's encrypt免费SSL证书

本文原创：**中国科学技术大学 张焕杰**

修改时间：2017.10.04

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
* Let's encrypt服务器提供一个随机字符串，要求放在网站的.well-known/acme-challenge目录下
* 服务器将随机字符串放在.well-known/acme-challenge目录下
* Let's encrypt服务器通过http访问到该随机字符串，验证服务器拥有该域名，颁发证书

## 二、Let's encrypt 证书生成工具

有很多种Let's encrypt 证书生成工具，这里介绍完全由shell脚本完成整个过程的getssl [https://github.com/srvrco/getssl](https://github.com/srvrco/getssl)。

以下过程使用域名blackip.ustc.edu.cn演示，服务器是apache。

1. 安装过程
````
mkdir /usr/src/getssl
cd /usr/src/getssl
curl --silent https://raw.githubusercontent.com/srvrco/getssl/master/getssl > getssl ; chmod 700 getssl
````

2. 生成基本配置
````
cd /usr/src/getssl
./getssl -c blackip.ustc.edu.cn
````
3. 修改配置
````
vi /root/.getssl/getssl.cfg /root/.getssl/blackip.ustc.edu.cn/getssl.cfg
````
其中/root/.getssl/getssl.cfg修改为：
````
CA="https://acme-v01.api.letsencrypt.org"
ACCOUNT_EMAIL="james@ustc.edu.cn"
````
/root/.getssl/blackip.ustc.edu.cn/getssl.cfg修改为：
````
ACL=('/var/www/html/.well-known/acme-challenge')

DOMAIN_CERT_LOCATION="/etc/ssl/blackip.ustc.edu.cn.crt"
DOMAIN_KEY_LOCATION="/etc/ssl/blackip.ustc.edu.cn.key"
CA_CERT_LOCATION="/etc/ssl/chain.crt"

#对于nginx, 需要full_chain.pem(其实这个文件就是blackip.ustc.edu.cn.crt + chain.crt)，可以使用
#DOMAIN_CHAIN_LOCATION="/etc/ssl/blackip.ustc.edu.cn.full_chain.pem"

RELOAD_CMD="/sbin/service httpd restart"
````

4. 获取证书

执行命令获取证书
````
./getssl -d blackip.ustc.edu.cn
````
执行完毕后/root/.getssl/blackip.ustc.edu.cn会有四个文件，使用以下命令可以看到文件的内容：

````
#服务器私钥
openssl rsa -noout -text -in blackip.ustc.edu.cn.key 
#证书签名请求
openssl req -noout -text -in blackip.ustc.edu.cn.csr
#证书
openssl x509 -in blackip.ustc.edu.cn.crt -text
#证书链
openssl x509 -in chain.crt -text
````

5. 证书使用

编辑/etc/httpd/conf.d/ssl.conf，修改以下内容：
````
SSLCertificateFile /etc/ssl/blackip.ustc.edu.cn.crt
SSLCertificateKeyFile /etc/ssl/blackip.ustc.edu.cn.key
SSLCertificateChainFile /etc/ssl/chain.crt
````
执行````service httpd restart````证书生效。 

这时可以使用 https://www.ssllabs.com/ssltest/analyze.html?d=blackip.ustc.edu.cn 测试服务器证书是否工作正常。

如果是nginx，配置是：
```
server {
	listen 443 ssl;
	server_name blackip.ustc.edu.cn;
	ssl_certificate /etc/ssl/blackip.ustc.edu.cn.full_chain.pem;
	ssl_certificate_key /etc/ssl/blackip.ustc.edu.cn.key;
	location / {
		root /usr/share/nginx/html;
	}
```

6. 证书自动更新

Let's encrypt证书有效期为90天，需要在90天内更新，更新方式是执行命令
````/usr/src/getssl/getssl -d blackip.ustc.edu.cn````
即可，离失效期还有30天的证书会得到更新，并自动执行上面定义的RELOAD_CMD启动服务进程。可以使用crontab每天执行一次。

7. 一台服务器有多个域名时的证书生成

假如一台服务器同时服务多个域名，可以生成含有多个域名的证书。

如果这些域名完全是同一个网站，如blackip.ustc.edu.cn 还有个域名是www.blackip.ustc.edu.cn、blacklist.ustc.edu.cn、www.blacklist.ustc.edu.cn，他们是同一个网站，
只要在/root/.getssl/blackip.ustc.edu.cn应增加如下的配置：
````
SANS="www.blackip.ustc.edu.cn,blacklist.ustc.edu.cn,www.blacklist.ustc.edu.cn"
ACL=('/var/www/html/.well-known/acme-challenge')
USE_SINGLE_ACL="true"
````

如果域名是不同的网站，如blackip.ustc.edu.cn服务器通过虚拟主机的方式需要服务noc.ustc.edu.cn live.ustc.edu.cn，它们的根目录分别是/var/www/html/noc /var/www/html/live，则
在/root/.getssl/blackip.ustc.edu.cn应增加如下的配置：
````
SANS="noc.ustc.edu.cn,live.ustc.edu.cn"
ACL=('/var/www/html/.well-known/acme-challenge'
     '/var/www/html/noc/.well-known/acme-challenge'
     '/var/www/html/live/.well-known/acme-challenge')
````


***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
