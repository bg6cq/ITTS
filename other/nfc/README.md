## [原创] CentOS 6 php 使用ACS ACR1252U USB NFC读卡器 读取卡UID

本文原创：**中国科学技术大学 张焕杰**

修改时间：2020.04.30

本文记录在CentOS 6环境下，ACR 1252U USB NFC读卡器读取卡UID的过程。

1. 硬件设备

ACS ACR1252U USB NFC读卡器，使用USB接口连接到安装CentOS 6的机器。

2. 软件安装

```
yum install -y wget pcsc-lite pcsc-lite-devel php-pear php-devel libusb1 unzip
pecl install pcsc-alpha

wget https://www.acs.com.hk/download-driver-unified/12131/ACS-Unified-PKG-Lnx-118-P.zip
unzip ACS-Unified-PKG-Lnx-118-P.zip
rpm -i ACS-Unified-PKG-Lnx-118-P/epel/pcsc-lite-acsccid-1.1.8-1.el6.x86_64.rpm

service messagebus start
service pcscd start
service haldaemon start

#修改/etc/php.ini, 把
;enable_dl = Off 改为
enable_dl = On
#并增加
extension=pcsc.so

#一段简单的测试程序
wget -O test.php http://linux.ustc.edu.cn/test.phps
```

test.php内容

```php
<?php
// php 访问 NFC 读卡器

if (!extension_loaded('pcsc')) {
  dl('pcsc.so');
}

# Get a PC/SC context
$context = scard_establish_context();
//var_dump($context);

# Get the reader list
$readers = scard_list_readers($context);
//var_dump($readers);

# Use the first reader
$reader = $readers[0];
echo "Using reader: ", $reader, "\n";

# Connect to the card
$laststatus =false;
while(1) {
        $connection = scard_connect($context, $reader);
        if(!$connection) {  // 未找到卡，继续
                $laststatus =false;
                continue;
        }
        if($laststatus) {  // 之前已经读过卡，继续
                scard_disconnect($connection);
                continue;
        }
        $laststatus = true;
        $CMD = "FFCA000000";  // 读UID
        $res = scard_transmit($connection, $CMD);
        $rcode = substr($res, -4, 4);
        $uid = substr($res, 0, -4);
        if($rcode == "9000") {
                echo "read new card: ", $uid, "\n";
        } else {
                var_dump($res);
        }

        scard_disconnect($connection);

}
scard_release_context($context);

?>
```

***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
