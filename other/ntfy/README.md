## [原创] 自建ntfy.sh server向安卓手机发送通知消息

本文原创：**中国科学技术大学 张焕杰**

修改时间：2025.12.23

[Bark](https://apps.apple.com/cn/app/bark-给你的手机发推送/id1403753865) 是苹果手机上方便接收通知信息的轻量级App，经常被用于发送日常管理中的通知消息。

[noty.sh](https://ntfy.sh)是安卓手机上方便接收通知消息的轻量级App，本文简介自建服务器的方法。

### 1. 软件获取

[https://github.com/binwiederhier/ntfy](https://github.com/binwiederhier/ntfy)可以下载编译好的版本，或使用docker运行。

### 2. 运行

直接运行程序或使用docker运行程序，我使用的rpm包安装，配置文件如下:

/etc/ntfy/server.yml
```
base-url: http://xxx.ustc.edu.cn:2586
listen-http: "-"
listen-https: ":2586"
key-file:  /etc/nginx/xxx.ustc.edu.cn.key
cert-file:  /etc/nginx/xxx.ustc.edu.cn.pem
```

然后执行 nfty serv 可以调试运行，正常了使用systemctl start nfty即可正式运行。


使用以上配置，将来可以用 https://xxx.ustc.edu.cn:2586/ 来访问。

### 3. 客户端App及发送

#### 3.1 安卓手机安装ntfy App

按 + 新增订阅主题，如主题名为 test，选择使用其他服务器，输入 https://xxxx.ustc.edu.cn:2586/


#### 3.2 发信息过程

```
curl -d "Hi test" https://xxx.ustc.edu.cn:2586/test
```
其中 test 是前面订阅的主题，访问上述URL后，手机可以收到消息。


### 4. 高级使用

可以增加用户验证，但太麻烦不建议


参考：

* https://ntfy.sh/



***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
