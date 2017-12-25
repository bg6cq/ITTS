## [原创] H3C AC/AP 调试命令

本文原创：**中国科学技术大学 张焕杰**

修改时间：2017.12.25

# 一、开启AP的远程telnet功能

```
<AC>sys
[AC]_hidecmd
[AC-hidecmd] wlan ap ap_name exec-control enable
[AC-hidecmd] wlan ap ap_name telnet enable
```
然后接可以telnet到AP上

# 二、查看AP信道利用率、空口状态

使用上述一开启telnet，并telnet到AP上
```
[WA...]_hidecmd
[WA...-hidecmd]dis ar5drv 2 channelbusy
[WA...-hidecmd]dis ar5drv 2 statistics
# 1 是5G，2 是2.4G
```

# 三、查看一个客户端的漫游状态

```
<AC>display wlan client roam-track mac-address xxxx-xxxx-xxxx
```


***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
