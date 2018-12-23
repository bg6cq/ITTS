## [原创]自建CA签发证书

本文原创：**中国科学技术大学 张焕杰**

修改时间：2018.12.23


仅仅使用openssl，不依赖其他配置文件，自建CA并签发服务器证书。

## 一、CA私钥和自签证书

执行以下命令，回答提示问题，生成CA的证书。

```
openssl genrsa -out rootCA.key 4096 
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 7000 -out rootCA.crt 
```
rootCA.key 是CA私钥，要存好了。

rootCA.crt 是自签的证书，有效期7000天。这个文件可以交给每个客户端，加到信任根证书CA。

执行`openssl x509 -in rootCA.crt  -text`可以查看证书。


## 二、签发服务器证书

创建一个配置文件 `mycert.conf`，内容如下：
```
[ req ]
default_bits		= 2048
default_md		= sha256
distinguished_name	= req_distinguished_name
attributes		= req_attributes
x509_extensions	= v3_ca	# The extentions to add to the self signed cert
string_mask = utf8only

[ req_distinguished_name ]

[ req_attributes ]

[ v3_ca ]
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer
basicConstraints = CA:true
```

创建一个签发脚本 `gencrt.sh`，内容如下(请修改C=CN...)：

```
#!/bin/bash

openssl genrsa -out $1.key 2048

openssl req -new -subj "/C=CN/ST=Anhui/L=Hefei/O=USTC/OU=NIC/CN=$1" \
    -reqexts SAN \
    -config <(cat mycert.conf <(printf "[SAN]\nsubjectAltName=DNS:$1")) \
    -key $1.key -out $1.csr

openssl x509 -req -CA rootCA.crt -CAkey rootCA.key -CAcreateserial \
    -sha256 -days 3650 -extfile <(printf "subjectAltName=DNS:$1") \
    -in $1.csr -out $1.crt 
```

执行 `bash gencrt.sh test.ustc.edu.cn`既可以产生3个文件，分别是：
```
test.ustc.edu.cn.key 服务器私钥
test.ustc.edu.cn.csr 证书申请文件，无用了
test.ustc.edu.cn.crt 服务器证书
```

执行`openssl x509 -in test.ustc.edu.cn.crt  -text`可以查看证书。


***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
