## [原创] 常用脚本

本文原创：**中国科学技术大学 张焕杰**

修改时间：2025.08.13

### 1. sshd pubkey 与sshd日志的SHA256显示对应

sshd登录日志中会显示pubkey的SHA256 经过base64编码的结果，如下：

```
Accepted publickey for root from ... port 45344 ssh2: RSA SHA256:cmGGoa7Ft4vrgd+Zi1NSRODBzcRGTjtM3GMOdgsKsl0
```
这一段对应的是pubkey
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDMIOGPraZIjr+a7pmzar/X2iQy/oS6LvNazVTRfYB1TOQpnWHjDPwJOJseapVxH9Rka2ODJonHvbhJzl/KKqyzay1GY82PRWoj2QXNGhPgNpQd3hztKV2ukP403+wFFbyBvdp3CTfIEmK35vXraGZWmpZXlYIOtPxbI/1pYQTRYd2xFpjOorAQBdUKIzDfK+jQplqcjfoffIs8E2dvcKRgtXSpsNbbqye97h8+GCA2SI3D0l51zbMqSOw+54a8mMPIcBftk8PHcHjPxbjaWfOEAzk9/Ly4cYwZ51+BdEWu5ed5paNNLHr7ty2jJJTSlSF8+PoAc73x+r5JIC4Yxt436iEVYk8DjeVmBzGD1xL112y6XqoyVeey9RbkmdrUon3NVD/yLu86DWVHqvyt6JY+6V/LKYUGT0okR78K0Xho4/rXD7NtME3hDBquC6QqkPdKWwfdCB9fDoGfq6BGxanH2T9lnOMX2WnN7QOsJ1BBqtQDx2Ruhc20xPvYmPaxUwC7Wxd4F4zCEAsbD46oLaWlC5hc6DhTnv/kLU89RGinv/pUDXhxqoqQi5YmGVt1Pnw+nK6Ixz/BhEQmCvRJwSK1vrNoKkasQOqEkK8V51HHRPaeTl/54YZccKT+T/kOw6t4oGjFlws60VjxPHrn6uukQBrsX30TPcVl2P+VgDKOjQ==
```

将pubkey转换成SHA256的base64脚本如下：
```
cat authorized_keys    |
    awk '{ print $2 }' | # Only the actual key data without prefix or comments
    base64 -d          | # decode as base64
    sha256sum          | # SHA256 hash (returns hex)
    awk '{ print $1 }' | # only the hex data
    xxd -r -p          | # hex to bytes
    base64
```
输出的结果是
```
cmGGoa7Ft4vrgd+Zi1NSRODBzcRGTjtM3GMOdgsKsl0
```


### 2. 显示文件在系统中的建立时间

黑客入侵系统时经常会修改文件的修改时间，而很少去修改文件的建立时间。下面的脚本`show_birthtime.sh`可以显示一个目录下文件的建立时间。
```
#!/bin/bash
# 显示一个目录下文件的创建时间
# 定义要遍历的目录，如果没有参数，则使用当前目录
dir=${1:-.}

find "$dir" -type f -print | while read file; do
    birth_time=$(stat -c '%w' "$file")
    printf "%s %s\n" "$birth_time" "$file"
done
```
使用例子：获取/usr下的文件建立时间

```
./show_birthtime.sh /usr | sort -n 
```



***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
