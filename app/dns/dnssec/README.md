## [原创] ustc.edu.cn 域增加DNSSEC功能过程

本文原创：**中国科学技术大学 张焕杰**

修改时间：2019.09.13

清华大学段海新老师在2011年就写过有关DNSSEC的技术文章[DNSSEC 原理、配置与布署简介](https://blog.csdn.net/syh_486_007/article/details/50990973)，详细介绍了DNSSEC。本文记录 ustc.edu.cn 域增加DNSSEC功能的过程。


## ustc.edu.cn 域名情况

ustc.edu.cn 分了5个view，每个view有一个zone文件，因此有5个zone文件。bind 软件为CentOS 6中内置，使用chroot模式运行。

由于使用的bind软件并不是最新，因此本文使用的方式不一定是最简单的方式。

bind 大部分文件存放在 /var/named/chroot/var/named 目录，zone文件存放在 /var/named/chroot/var/named/zones 目录。

## 步骤一：生成ZSK和KSK密钥

```
cd /var/named/chroot/var/named
dnssec-keygen -r /dev/urandom -3 ustc.edu.cn
dnssec-keygen -f ksk -r /dev/urandom -3 ustc.edu.cn
cp *.key zones
```

生成了4个文件：
```
Kustc.edu.cn.+007+32747.key
Kustc.edu.cn.+007+32747.private
Kustc.edu.cn.+007+19065.key
Kustc.edu.cn.+007+19065.private
```

由于zone文件放在/var/named/chroot/var/named/zones 目录下，为了后续方便，把 *.key 文件cp到zones目录下

## 步骤二：在zone文件中增加key文件

在zone文件最后面增加：
```
$INCLUDE Kustc.edu.cn.+007+19065.key
$INCLUDE Kustc.edu.cn.+007+32747.key
```

## 步骤三：对原有zone文件进行签名

科大有5个zone文件，因此对5个文件签名，签名后的zone文件自动添加后缀.signed，如ustc.edu.cn.cernet签名后生成文件ustc.edu.cn.cernet.signed

```
cd /var/named/chroot/var/named
dnssec-signzone -A -3 $(head -c 1000 /dev/urandom | sha1sum | cut -b 1-16) -N INCREMENT -o ustc.edu.cn -t zones/ustc.edu.cn.cernet
dnssec-signzone -A -3 $(head -c 1000 /dev/urandom | sha1sum | cut -b 1-16) -N INCREMENT -o ustc.edu.cn -t zones/ustc.edu.cn.chinanet
dnssec-signzone -A -3 $(head -c 1000 /dev/urandom | sha1sum | cut -b 1-16) -N INCREMENT -o ustc.edu.cn -t zones/ustc.edu.cn.cncnet
dnssec-signzone -A -3 $(head -c 1000 /dev/urandom | sha1sum | cut -b 1-16) -N INCREMENT -o ustc.edu.cn -t zones/ustc.edu.cn.cmcc
dnssec-signzone -A -3 $(head -c 1000 /dev/urandom | sha1sum | cut -b 1-16) -N INCREMENT -o ustc.edu.cn -t zones/ustc.edu.cn.other
```

## 步骤四：修改bind配置，使用签名后域文件

如原来配置为
``` 
zone "ustc.edu.cn" in{type master; file "zones/ustc.edu.cn.cernet";};
``` 
修改为
``` 
zone "ustc.edu.cn" in{type master; file "zones/ustc.edu.cn.cernet.signed";};
``` 

同时，确保options中有启用dnssec功能的配置
```
	dnssec-enable yes;
```

## 步骤五：重启bind，生效配置

```
service named restart
```

重启后，访问`https://dnssec-analyzer.verisignlabs.com/ustc.edu.cn`能看到有DNSSEC相关记录，但有个警告是还没有DS记录。

## 步骤六：上级服务器增加DS记录

文件`/var/named/chroot/var/named/dsset-ustc.edu.cn.`中内容如下：
```
ustc.edu.cn.		IN DS 19065 7 1 4EBE527CCF84FC2DD62ACFCD464BE008E0FEAF68
ustc.edu.cn.		IN DS 19065 7 2 EBD1C6420F893D8FF9950ADBF896075D059006439419634128566709 8EBD74F1
```

将这些内容发给edu.cn服务器管理员，添加后，访问`https://dnssec-analyzer.verisignlabs.com/ustc.edu.cn`能看到DNSSEC工作正常。

![DNSSEC](img/dnssec.png)

## 修改域名zone文件的签名步骤

一旦修改了域名zone文件，均要重复步骤三的过程，并重启bind。我们使用git pre-commit hook自动这个过程。

## 域名zone文件签名有效期

默认的域签名有效期30天，30天内要重复步骤三的过程，并重启bind。可以在签名zone文件时制定更长的有效期。

30天内我们的DNS一定会有修改，因此不担心这个问题。


## 自动在线签名

bind 9.9之后支持在线签名，也就是bind服务器启动后自动进行签名，管理更简单。

由于我们服务器上的bind比较老，升级到最新：

```
yum install -y openssl-dev libcap-devel
wget https://downloads.isc.org/isc/bind9/9.14.6/bind-9.14.6.tar.gz
tar zxvf bind-9.14.6.tar.gz 
cd bind-9.14.6

./configure                     \
  --prefix=/usr --exec-prefix=/usr --bindir=/usr/bin                  \
  --sbindir=/usr/sbin --sysconfdir=/etc --localstatedir=/var          \
  --datadir=/usr/share --includedir=/usr/include --libdir=/usr/lib64  \
  --libexecdir=/usr/libexec --sharedstatedir=/var/lib                 \
  --mandir=/usr/share/man --infodir=/usr/share/info --with-libtool    \
  --with-pic --disable-static --disable-isc-spnego   \
  --enable-querytrace              \
  --enable-fixed-rrset --enable-rpz-nsip --enable-rpz-nsdname         \
  --with-dlopen=yes --with-dlz-filesystem=yes --without-python
make
make install
```

升级后执行service named configtest出现错误

```entropy.c:26: fatal error: RAND_bytes(): error:24064064:lib(36):func(100):reason(100)
/etc/init.d/named: line 285:  1652 Aborted                 /usr/sbin/named-checkconf $ckcf_options ${named_conf}
```
原因是/var/named/chroot/dev/下缺少文件 random urandom，执行
```
cd /dev
tar cvf - *rand* | ( cd /var/named/chroot/dev; tar xvf -)
```
并修改 /var/named/chroot/var/named 的owner为named.named后正常

我们使用的配置如下：

```
view "CHINANET" {
   match-clients { CHINANET;};
   allow-recursion { none; };
   include "/etc/named.common.conf";
   zone "ustc.edu.cn" in {
	type master;
	key-directory "/var/named";
	file "zones/ustc.edu.cn.chinanet";
	auto-dnssec maintain;
	inline-signing yes;
   };
};
```

生成的key文件放在 /var/named 目录下

使用在线签名的时候，需要注意以下问题：

1. /var/named 目录对named可写

2. /var/named/zones 目录对named可写，并且不能有前缀相同，多了 .signed 后缀的文件，因为bind运行时要写这样的文件。

3. 如果有多个view，zone文件不能有名字相同的，原因是bind运行时签名写 .signed 文件时后面的view因为文件存在，会错误。


***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
