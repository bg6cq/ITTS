# 在 ProFTPd 中配置 FTPS (FTP over TLS)

作者：西北农林科技大学 卞一帆

## 为什么要配置 TLS

类似于 HTTP 和 HTTPS ，建立在 TLS 上的应用层传输提供了更高的安全性，可以避免用户的凭据被通过窃听的方式获得，也可保证文件传输的安全性。

## 环境准备

1. Linux 服务器（ CentOS 和 Debian 系均可，本例中使用 CentOS 6.9 ）
2. 安装 ProFTPd （本例中使用编译安装，参考[官方文档](http://www.proftpd.org/docs/howto/Compiling.html)）

## 配置 TLS

本例选择在 `proftpd.conf` 中配置一个 VirtualHost 。如果需要配置到全局，只需要将 TLS 相关指令放到全局配置里。用方括号包括的内容需要根据实际情况修改。

```
<VirtualHost [Your Server IP]>
        ServerName [Your Server Name]
        DefaultRoot [Path To FTP Root]

        ExtendedLog [Path to extended log]
        TransferLog [Path to transfer log]
        UseEncoding on

        TLSEngine on
        TLSLog [Path to TLS log]
        TLSProtocol TLSv1.1 TLSv1.2
        TLSRequired off
        TLSRSACertificateFile [Path to certificate]
        TLSRSACertificateKeyFile [Path to private key]
        TLSVerifyClient off
        TLSRenegotiate none

        # Your own configuration goes here
</VirtualHost>
```

其中，FTPS 使用的证书和 nginx 配置 HTTPS 的证书可以是一样的。Common Name 字段应为用户连接 FTP 服务器时输入的主机名（建议域名）。
在实际测试中，FileZilla 会在初次连接时将证书信息显示给用户，让用户选择是否信任此证书。