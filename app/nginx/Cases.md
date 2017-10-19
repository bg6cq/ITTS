## Nginx 400错误案例

**西安财经学院 王伟**

修改时间：2017.10.17

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

**3. 注意事项**

* “欺骗”规则需要DNS和Nginx两者共同完成，任何一方都不能出问题。

* Nginx服务器的hosts文件为何会被修改？原因应该是`NetworkManager`会在重启后根据自己的配置修改hosts。所以修改hosts文件时先临时关闭`NetworkManager`


```
service  NetworkManager stop
```
或者


```
/bin/systemctl stop NetworkManager.service
```

* 这种方式并不是通用解决办法，如果没有特殊情况，还是强烈建议使用：

````
   proxy_pass http://ip:port;#
````
