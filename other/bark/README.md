## [原创] 自建bark-server向苹果手机发送通知消息

本文原创：**中国科学技术大学 张焕杰**

修改时间：2024.12.04

[Bark](https://apps.apple.com/cn/app/bark-给你的手机发推送/id1403753865) 是苹果手机上方便接收通知信息的轻量级App，经常被用于发送日常管理中的通知消息，本文简介自建服务器的方法。

1. 软件获取

[https://github.com/Finb/bark-server](https://github.com/Finb/bark-server)可以下载编译好的版本，或使用docker运行。

2. 运行

直接运行程序或使用docker运行程序。
```
./bark-server --addr 0.0.0.0:8080 --data ./bark-data
```

3. 工作原理

3.1 注册过程

手机有一个64字节长的Device Token，使用该Device Token访问

```
http://x.x.x.x:8080/register?devicetoken=64字节长的device_token
```
在Bark Server中注册，返回长度22字节的device_key。

成功会显示如下内容
```
 {"code":200,"message":"success","data":{"key":"af6kyzEXwb3wfV7fwThbhB","device_key":"af6kyzEXwb3wfV7fwThbhB","device_token":"********"},"timestamp":1733282760}
```

查看手机的Device Token，可以在苹果手机安装Bark app后，在 设置/Device Token 处单击，将手机的Device Token拷贝到粘贴板，长度为64字节。

3.2 发信息过程

```
curl http://x.x.x.x:8080/af6kyzEXwb3wfV7fwThbhB/title/msg
```
其中af6kyzEXwb3wfV7fwThbhB是上一步的device_key，访问上述URL后，手机可以收到消息。


4. 使用

安装自己的Bark Server后，在手机Bark App中点击右上角 + 号，添加自己的服务器。

添加后，Bark App会自动完成register注册过程，将发送消息的URL显示在 App 中。


5. 配置nginx

为了方便，可以使用nginx代理提供https访问
```
	location ^~ /bark/  {
		keepalive_timeout  0;
		proxy_pass http://127.0.0.1:8080/;
	}
```
使用以上配置，将来可以用 https://x.x.x.x/bark/ 来访问。


参考：

* https://github.com/Finb/bark-server



***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
