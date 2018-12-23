## nginx服务器优化

**中国科学技术大学 张焕杰**

创建时间：2018.12.20

nginx服务器优化有两方面工作，一是增加nginx单个进程打开的文件数，二是优化Linux系统的网络参数。


### 1. nginx打开文件数优化

Linux系统默认的单个进程可以打开1024个文件，这对nginx来说远远不够。如果不调整，很容易因为连接多占满文件数，导致出现明显的异常。

检查运行的nginx进程打开文件数，可以用`ps ax | grep nginx`找到nginx的pid，然后 `grep "Max open files" /proc/$pid/limits` 看到限制值。

注意nginx有多个进程，要分别检查master和worker进程的打开文件数。

网上查到的很多文档有错误和误导的地方，表现在下面的地方：

* 修改 /etc/security/limits.conf 系统重启时对nginx无效

limits.conf只在用户执行login时生效，而系统重启nginx启动时并未执行login过程，因此无效。

系统启动后，使用root用户执行 service nginx restart可以生效。也就是修改limits.conf后，每次系统重启，都需要root执行一次service nginx restart才行。

* nginx.conf中设置worker_rlimit_nofile 会带来隐患

nginx.conf中设置worker_rlimit_nofile虽然可以加大nginx worker进程的文件数，但是并不加大nginx master进程的文件数，导致在nginx reload时碰到错误无法正常生效。

修改配置后，使用service nginx reload可以不中断服务而生效。如果nginx reload无法使用，对nginx的持续服务有严重影响。

正确的增加nginx打开文件数分2步：

#### 1.1 编辑文件`/etc/sysctl.d/file-max.conf`，增加1行:
```
echo "fs.file-max = 655360" > /etc/sysctl.d/file-max.conf
```

#### 1.2 想办法在启动nginx前执行`ulimit -HSn 655360`

不同的系统具体配置方式不一样，主要有：

* CentOS 6等使用sysvinit的系统

修改`/etc/sysconfig/nginx`，增加一行即可。
```
ulimit -HSn 655360
```

* CentOS 7、Ubuntu 18.04 等使用systemd的系统

编辑文件`/etc/systemd/system/nginx.service.d/my-limit.conf`，增加[Service]段并添加行：
```
mkdir /etc/systemd/system/nginx.service.d/
echo "[Service]" > /etc/systemd/system/nginx.service.d/my-limit.conf
echo "LimitNOFILE=655360" >> /etc/systemd/system/nginx.service.d/my-limit.conf
```

### 2. Linux网络参数优化

如果可能，最好不开启conntrack连接跟踪。如果开启conntrack，可以参考以下链接优化

* https://github.com/bg6cq/nginx-centos-6
* https://github.com/bg6cq/nginx-centos-7


***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
