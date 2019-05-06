## [原创] 使用httptest监测网站服务

本文原创：**宣城职业技术学院 刘石**

修改时间：2019.05.06


## 前言

为保证站点在访问异常的情况下管理员可实时收到通知，及时进行处理，快速恢复站点访问。

使用中国科学技术大学 张焕杰 开发的 [httptest](https://github.com/bg6cq/httptest)，在站点异常时实时发送短信给管理员，管理员第一时间得到服务正常异常报告。

程序详细代码见：https://github.com/bg6cq/httptest

## 安装配置

### 系统环境

* 系统：CentOS7 64位
* CPU：2核
* 内存：2G
* 硬盘：30G

### 安装过程

```
yum -y update
reboot
yum -y install git curl gcc openssl-devel
cd /usr/src
git clone https://github.com/bg6cq/httptest
cd httptest
make
cd httpmonitor
```
  
### 配置服务

/usr/src/httptest/httpmonitor 目录下共有monitor.sh、urllist.txt两个文件

* monitor.sh为Shell执行程序，用户可在里面自定义各项参数
* urllist.txt为要监控的站点信息文件

urllist.txt
```
# 格式为: 要监测的URL 记数文件名 手机号码
https://www.baidu.com www.baidu.com 18905630000
https://www.qq.com www.qq.com 18905630000
https://www.xcvtc.edu.cn www.xcvtc.edu.cn 18905630000
```

monitor.sh
```
COUNT=2 # 设置出现几次连续错误才开始发通知，未恢复前只发一次

log ()
{ 	a=`date`;
       	echo $a $1 >> ${LOGFILE}
       	#sendsms $2 $1, $2是手机号码，$1是消息  这里可根据自己的情况设置发送短信的内容，我的设置如下
       	curl "http://www.test.com/SendSms?phone=$2&content=$1"
 }
```

两个msg为定义异常和恢复正常发送短信内容的变量，可根据自己的情况进行设置。
    
### 设置定时任务

```
crontab -e 
*/1 * * * * sh /usr/src/httptest/httpmonitor/monitor.sh  >/dev/null 2>/dev/null
```

此时系统已可正常运行，当有站点访问异常时，便会发送短信进行通知。


***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
