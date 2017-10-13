## nginx-rrd绘图参数

**西安财经学院 王伟**

创建时间：2017.10.13

nginx-rrd工具为nginx提供了一个简单的监控图，但是默认生成的监控图实在不好看。
![image](upload/nginx-rrd-old.png)

这里不提如何下载及安装，教程及源码网上很多，对于`rrdtool`图像参数的例子非常少。这里主要讲几个绘图可以使用参数。

nginx-rrd 生成图像的文件是 `nginx-graph.pl`，其中
````
RRDs::graph "$img/requests-$ServerName-$period.png",
.....
````
这里开始就是绘图的参数了，每个参数一行。每行`,`结束，语句`;`结束。

#### 1. 中文支持

添加一行

```
"-n","TITLE:10:'/usr/share/fonts/zh_CN/SIMHEI.TTF'",
```
`SIMHEI.TTF`从Windows系统拷贝就可以。

#### 2. 修改样式

原始的参数：

```
"LINE2:total#FFD300:Total:STACK",
```
其中

`LINE2`代表线条的粗细（像素），可选1-3。其实3个粗细都不好看，默认是`LINE2`（每个点2像素）。

`total`代表`.rdd`文件中对应的变量

`#FFD300`当然是代表颜色了

`Total`是图例的名字

`STACK`是堆叠图，默认不带这个参数

这个样式不像MRTG或者Cacti，仅仅会画出线条。

修改`LINE2`为`AREA`，变成类似MRTG或Cacti的样式：

```
"AREA:total#FFD300:连接数(Total):STACK",
```
#### 3. 增加平滑

```
"--slope-mode",
```
默认`rrdtool`绘图是画水平线，这个参数可以画比较流畅的线性。

#### 4. 效果图

![image](upload/nginx-rrd.png)

#### 5.数据抓取

一些基本参数是由`nginx-rrd.conf`文件完成，可以监控多个nginx主机，用空格分隔。

```
# server_url;server_name
SERVERS_URL="http://127.0.0.1/status;127.0.0.1 http://localhost/status;localhost"
```
对于每个目标，用`;`分隔，`server_url`是nginx-rrd可以访问的url地址，`server_name`是生成图像的标题，可以随便命名。

***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
