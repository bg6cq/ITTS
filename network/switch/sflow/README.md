## [原创]交换机sflow抓包的简单说明

本文原创：**中国科学技术大学 张焕杰**

修改时间：2018.04.10

最近新买的H3C S5130交换机支持sflow功能，下面是使用sflow抓包的简单说明。

## 一、sflow原理

交换机支持sflow，可以将接口发送/接收的数据包按照一定比例抓包，通过sflow协议把抓到的数据包交给collector，collector收到后可以对数据包进行分析。


## 二、sflow collector

有很多sflow collector，最简单的是 [InMon sFlow Toolkit](https://inmon.com/technology/sflowTools.php)。

其中还有windows下的sflowtool.ext，下载执行即可(默认在UDP端口6343)。


## 三、H3C S5130交换机侧配置

假定运行sflowtool.exe的机器IP是x.x.x.x，采集 Gi1/0/1端口数据包(采集比例是1000:1，即每1000个包采集1个)的配置如下：

```
 sflow collector 1 ip x.x.x.x description "CLI Collector"

interface GigabitEthernet1/0/1
 sflow flow collector 1
 sflow sampling-rate 1000
```

## 四、运行效果

运行 `sflowtool.exe` ，可以看到有sflow数据。

运行 `sflowtool.exe -t > t.cap`，运行一段时间，`CTRL-C`终止，使用wireshark打开t.cap可以看到抓取的数据包。


***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
