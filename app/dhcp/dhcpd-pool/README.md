## [原创] Linux下ISC dhcpd分配状态显示

本文原创：**中国科学技术大学 张焕杰**

修改时间：2017.12.27

Linux下的ISC dhcpd是最常用的dhcp服务器，本文记录一个显示dhcpd分配状态的程序dhcpd-pool安装过程。

dhcpd-pool可以显示dhcpd的分配状态，项目主页在 [http://dhcpd-pools.sourceforge.net/](http://dhcpd-pools.sourceforge.net/)。

安装过程：

1. 下载文件

```
cd /usr/src
从 [https://sourceforge.net/projects/dhcpd-pools/files/](https://sourceforge.net/projects/dhcpd-pools/files/) 下载，我下载的是 dhcpd-pools-3.0.tar.xz 
```

2. 解压文件

`tar zxvf dhcpd-pools-3.0.tar.xz`

3. 下载需要的uthash.h 

`curl https://raw.githubusercontent.com/troydhanson/uthash/master/src/uthash.h > /usr/include/uthash.h`

4. 编译

```
cd dhcpd-pools-3.0
./configure --with-dhcpd-leases=/var/lib/dhcpd/dhcpd.leases
make
```

5. 测试运行
```
./dhcpd-pools
```

注意：如果使用shared-network, 在dhcpd.conf中 `shared-network xxx {`，xxx与后面的{一定要有空格。

6. 定时运行，输出html

crontab
```
* * * * * /usr/src/dhcpd-pools-3.0/dhcpd-pools  -f H > /var/www/html/dhcp.html
```

7. 案例

[http://202.38.64.17/dhcp.html](http://202.38.64.17/dhcp.html)

8. 汉化版本

[https://github.com/bg6cq/dhcpd-pools-cn](https://github.com/bg6cq/dhcpd-pools-cn) 有我汉化了html输出的版本，如果需要可以安装。

***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
