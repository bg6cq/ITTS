## [原创]使用Let's encrypt免费SSL证书

本文原创：**中国科学技术大学 张焕杰**

修改时间：2017.10.04

## 一、SSL证书产生过程介绍

1. SSL证书产生过程涉及以下几个概念：

* 服务器私有密钥：扩展名一般是.key。一般我们使用的是rsa算法，服务器自己生成的一组随机数做为私钥，同时产生对应的公钥。私钥需要安全存放，不让其他人知道。

* 证书签名请求：Certificate Signing Request，扩展名一般是.csr。服务器将自己的公钥hash后，加上希望绑定的域名信息，生成.csr文件。

* 证书：扩展名一般是.crt。服务器将.csr文件交给CA机构，CA机构验证服务器真实拥有该域名后，用CA机构的私钥对这些信息签名，生成.crt文件。

* 证书链：CA机构的证书是安装到客户机系统的，为了安全起见，CA机构会为自己的二级CA签发证书，最后由该二级CA对普通用户签发证书，证书链是记录该关系的。不提供证书链可以工作，但有时候会有告警。

2. CA服务机构

CA服务机构的存在是做为公认的第三方来验证服务器身份，这中间收取服务费。

为了方便使用，可以选择Let's encrypt免费CA机构。Let's encrypt已被广泛接受，申请证书也比较快捷，一般来说5分钟内可以完成从开始安装程序到申请证书过程。需要说明的是Let's encrypt签发的证书有效期是90天，需要在到期之前完成证书更新。


3. Let's encrypt免费证书颁发过程

* 服务器生成私钥，产生.csr文件
* 服务器将.csr文件交给Let's encrypt服务器
* Let's encrypt服务器提供一个随机字符串，要求放在网站的.well-known/acme-challenge目录下
* 服务器将随机字符串放在.well-known/acme-challenge目录下
* Let's encrypt服务器通过http访问到该随机字符串，验证服务器拥有该域名，颁发证书

## 二、Let's encrypt 证书生成工具

有很多中Let's encrypt 证书生成工具，这里介绍完全由shell脚本完成整个过程的getssl。

以下过程使用域名blackip.ustc.edu.cn演示，服务器是apache。

1. 安装过程
````
mkdir /usr/src/getssl
cd /usr/src/getsll
curl --silent https://raw.githubusercontent.com/srvrco/getssl/master/getssl > getssl ; chmod 700 getssl
````

2. 生成基本配置
````
cd /usr/src/getsll
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

6. 证书自动更新

Let's encrypt证书有效期为90天，需要在90天内更新，更新方式是执行命令
````/usr/src/get/ssl -d blackip.ustc.edu.cn````
即可，离失效期还有30天的证书会得到更新，并自动执行上面定义的RELOAD_CMD启动服务进程。可以使用crontab每天执行一次。

***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)