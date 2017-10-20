#### Nginx 40x错误案例

**西安财经学院 王伟**

修改时间：2017.10.20

**前言**

之前为了不改变现有网站的任何配置，以及可以使用$host参数，我们采用了一个“欺骗”的办法：修改DNS解析以及Nginx服务器的hosts文件。这样现有目标服务器可以不做任何变化。

**原理：**
对用户来说，域名被“欺骗”解析到Nginx服务器；对Nginx来说，域名被“欺骗”解析到目标服务器。

此时，DNS解析和Nginx服务器的hosts文件，成为能否正常进行反向代理的关键。任何一方出现问题，这种“欺骗”规则就失效了。

下面先重复一下真实的例子，然后看看出现的错误。

**1. 一个Nginx反向代理的例子**

一台服务器上运行了30个网站，分别绑定了不同的域名，现在要做反向代理，需要如何实现？

其实过程很简单：

* DNS解析全部执行Nginx服务器

* Nginx中写hosts文件，将这些域名指向后台服务器的ip

* Nginx中使用$host参数，然后每个网站的配置方式：
````
······
   proxy_pass http://www.ustc.edu.cn;#真实指向Nginx的那个域名
······
````
* **后台站点什么操作都不用做**


**2. 一个400错误案例**

按照上述配置，我们配置了60个网站。但其中一个网站出现了400错误：


````
400 Bad Request  Request Header Or Cookie Too Large
````

根据字面理解，是“头文件 或者 COOKIE文件太大了”，搜索到的大多数解决办法也是调整参数：
   

````
    
    client_header_buffer_size 16k;
    large_client_header_buffers 4 32k;

````

但如果是这个参数设置的问题，应该所有网站都返回相同的错误。很明显这里不是这种情况。

后经检查发现，是我们的欺骗规则出现了问题。Nginx服务器的hosts文件有一行被修改，缺少了对`www.ustc.edu.cn`地址的绑定。

这就造成了：DNS将用户“欺骗”到Nginx服务器，Nginx服务器查询本地hosts无果，向DNS请求解析后，又被“欺骗”到自己。最终导致Nginx反向代理到自己。

这样的死循环造成400错误。

**3. 一个404错误案例**

直接看配置：

````
......
	#第一段配置
	location  /tzcs/ 	{
		proxy_pass http://tzcs.xaufe.edu.cn/;
	}
	#第二段配置
	location / {
		proxy_pass http://tiyubu.xaufe.edu.cn;	
	}
......
````

这是发布在不同服务器上的网站，还是利用“欺骗”规则，只是多了一段配置，目的是访问`http://tiyubu.xaufe.edu.cn/tzcs/`时，可以显示`http://tzcs.xaufe.edu.cn`的内容。

配置看着是没什么问题的，第二段的跳转也正常。但第一段的跳转却会出现404错误。

问题还是出在$host参数上！因为“欺骗”规则需要nginx服务器和目标主机合作才能完成。

先考虑第二段配置（使用$host参数）：

* 用户请求`tiyubu.xaufe.edu.cn`，从DNS解析认为`tiyubu.xaufe.edu.cn`是nginx服务器
* nginx服务器认为`tiyubu.xaufe.edu.cn`是目标主机
* 目标主机认为`tiyubu.xaufe.edu.cn`是自己
* 整个过程$host都是`tiyubu.xaufe.edu.cn`

所以，用户请求`tiyubu.xaufe.edu.cn`会指向nginx服务器，nginx服务器根据规则跳转到目标主机，目标主机应答请求。

再看看第一段配置（同样是使用$host参数）：

* 用户请求`tiyubu.xaufe.edu.cn/tzcs/`，从DNS解析认为`tiyubu.xaufe.edu.cn`是nginx服务器
* nginx服务器认为`tiyubu.xaufe.edu.cn/tzcs/`是目标主机（根据配置，目标主机变成了`tzcs.xaufe.edu.cn`）
* 整个过程$host都是`tiyubu.xaufe.edu.cn`
* 目标主机认为`tzcs.xaufe.edu.cn`是自己，所以不会应答主机头`tiyubu.xaufe.edu.cn`，返回404错误
* 这种404错误，目前测试只会在IIS上出现，非IIS发布的网站仍然可以正常显示（应该和主机头判断的机制有关）

要修改这种错误，需要将：
````
proxy_set_header   Host   $host;
````

改为

````
proxy_set_header   Host   $proxy_host;

````
即可解决。

**注意**

这种解决方案并不通用，如果目标主机存在自动刷新，便会跳转至`$proxy_host`，即`tzcs.xaufe.edu.cn`。

**4.关于`/`符号**

注意到上面的例子`/tzcs/`和`http://tzcs.xaufe.edu.cn/`后面都有`/`，两者的含义如下：

* `/tzcs/`后面带`/`，输入`http://tiyubu.xaufe.edu.cn/tzcs`时会自动补齐后面的`/`
* `http://tzcs.xaufe.edu.cn/`后面带`/`，会传递`/tzcs/`；否则，`http://tiyubu.xaufe.edu.cn/tzcs/TiYuChengJi/123.html`会变成`http://tiyubu.xaufe.edu.cn/TiYuChengJi/123.html`

**5. 总结**

* “欺骗”规则需要DNS和Nginx两者共同完成，任何一方都不能出问题。

* Nginx服务器的hosts文件为何会被修改？原因应该是`NetworkManager`会在重启后根据自己的配置修改hosts。所以修改hosts文件时先临时关闭`NetworkManager`


```
service  NetworkManager stop
```
或者

 
```
/bin/systemctl stop NetworkManager.service
```

* 这种“欺骗”规则并不是通用解决办法，容易出现各种意想不到的问题。如果没有特殊情况，还是**强烈建议**使用：

````
   proxy_pass http://ip:port;#
````
