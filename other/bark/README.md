## [原创] 自建bark-server向苹果手机发送通知消息

本文原创：**中国科学技术大学 张焕杰**

修改时间：2024.12.04

[Bark](https://apps.apple.com/cn/app/bark-给你的手机发推送/id1403753865) 是苹果手机上方便接收通知信息的App，本文简介自建服务器的方法。

1. 软件获取

[https://github.com/Finb/bark-server](https://github.com/Finb/bark-server)可以下载编译好的版本，或使用docker运行。

2. 运行

直接运行程序或使用docker运行程序。
```
./bark-server --addr 0.0.0.0:8080 --data ./bark-data
```

3. 使用

3.1 获取手机Device Token

苹果手机安装Bark app后，在 设置/Device Token 处单击，将手机的Device Token拷贝到粘贴板，长度为64字节。

3.2 注册手机，获取device_key

这一步很关键，但文档中未提及。

```
curl http://x.x.x.x:8080/register?devicetoken=64字节长的device_token
```
成功会显示如下内容
```
 {"code":200,"message":"success","data":{"key":"af6kyzEXwb3wfV7fwThbhB","device_key":"af6kyzEXwb3wfV7fwThbhB","device_token":"********"},"timestamp":1733282760}
```
其中device_key是发送信息需要用到的。

3.3 发送消息

```
curl http://x.x.x.x:8080/af6kyzEXwb3wfV7fwThbhB/title/msg
```

其中af6kyzEXwb3wfV7fwThbhB是上一步的device_key，访问上述URL后，手机可以收到消息。


3.4 配置nginx

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
