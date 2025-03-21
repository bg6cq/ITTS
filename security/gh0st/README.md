## [原创]使用deepseek辅助分析安全通报的入侵事件

本文原创：**中国科学技术大学 张焕杰**

修改时间：2025.03.21

注：本文使用中国科学技术大学部署的deepseek https://chat.ustc.edu.cn，相关命令在Linux系统中运行，其他系统结果可能略有出入。

## 一、安全事件通报

收到安全通报，通报说某80端口发生木马安全事件，通报包含如下内容:
```
;payload=Gh0st\AD\00\00\00\E0\00\00\00x\9CKS``\98\C3\C0\C0\C0\06\C4\8C@\BCQ\96\81\81\09H\07\A7\16\95e&\A7*\04$&g+\182\94\F6\B000\AC\A8rc\00\01\11\A0\82\1F\\`&\83\C7K7\86\19\E5n\0C9\95n\0C;\84\0F3\AC\E8sch\A8^\CF4'J\97\A9\82\E30\C3\91h]&\90\F8\CE\97S\CBA4L?2=\E1\C4\92\86\0B@\F5`\0CT\1F\AE\AF]\0Ar\0B\03#\A3\DC\02~\06\86\03+\18m\C2=\FDtC，C\FDL<<==\\\9D\19\88\00\E5 \02\00T\F5+\\;hex=\\x47\\x68\\x30\\x73\\x74\\xAD\\x00\\x00\\x00\\xE0\\x00\\x00\\x00\\x78\\x9C\\x4B\\x53\\x60\\x60\\x98\\xC3\\xC0\\xC0\\xC0\\x06\\xC4\\x8C\\x40\\xBC\\x51\\x96\\x81\\x81\\x09\\x48\\x07\\xA7\\x16\\x95\\x65\\x26\\xA7\\x2A\\x04\\x24\\x26\\x67\\x2B\\x18\\x32\\x94\\xF6\\xB0\\x30\\x30\\xAC\\xA8\\x72\\x63\\x00\\x01\\x11\\xA0\\x82\\x1F\\x5C\\x60\\x26\\x83\\xC7\\x4B\\x37\\x86\\x19\\xE5\\x6E\\x0C\\x39\\x95\\x6E\\x0C\\x3B\\x84\\x0F\\x33\\xAC\\xE8\\x73\\x63\\x68\\xA8\\x5E\\xCF\\x34\\x27\\x4A\\x97\\xA9\\x82\\xE3\\x30\\xC3\\x91\\x68\\x5D\\x26\\x90\\xF8\\xCE\\x97\\x53\\xCB\\x41\\x34\\x4C\\x3F\\x32\\x3D\\xE1\\xC4\\x92\\x86\\x0B\\x40\\xF5\\x60\\x0C\\x54\\x1F\\xAE\\xAF\\x5D\\x0A\\x72\\x0B\\x03\\x23\\xA3\\xDC\\x02\\x7E\\x06\\x86\\x03\\x2B\\x18\\x6D\\xC2\\x3D\\xFD\\x74\\x43\\x2C\\x43\\xFD\\x4C\\x3C\\x3C\\x3D\\x3D\\x5C\\x9D\\x19\\x88\\x00\\xE5\\x20\\x02\\x00\\x54\\xF5\\x2B\\x5C
```

## 二、payload 分析

从中猜测hex=后面是可疑payload的hex编码，提取hex=后内容到文件 payload.hex，得到
```
\\x47\\x68\\x30\\x73\\x74\\xAD\\x00\\x00\\x00\\xE0\\x00\\x00\\x00\\x78\\x9C\\x4B\\x53\\x60\\x60\\x98\\xC3\\xC0\\xC0\\xC0\\x06\\xC4\\x8C\\x40\\xBC\\x51\\x96\\x81\\x81\\x09\\x48\\x07\\xA7\\x16\\x95\\x65\\x26\\xA7\\x2A\\x04\\x24\\x26\\x67\\x2B\\x18\\x32\\x94\\xF6\\xB0\\x30\\x30\\xAC\\xA8\\x72\\x63\\x00\\x01\\x11\\xA0\\x82\\x1F\\x5C\\x60\\x26\\x83\\xC7\\x4B\\x37\\x86\\x19\\xE5\\x6E\\x0C\\x39\\x95\\x6E\\x0C\\x3B\\x84\\x0F\\x33\\xAC\\xE8\\x73\\x63\\x68\\xA8\\x5E\\xCF\\x34\\x27\\x4A\\x97\\xA9\\x82\\xE3\\x30\\xC3\\x91\\x68\\x5D\\x26\\x90\\xF8\\xCE\\x97\\x53\\xCB\\x41\\x34\\x4C\\x3F\\x32\\x3D\\xE1\\xC4\\x92\\x86\\x0B\\x40\\xF5\\x60\\x0C\\x54\\x1F\\xAE\\xAF\\x5D\\x0A\\x72\\x0B\\x03\\x23\\xA3\\xDC\\x02\\x7E\\x06\\x86\\x03\\x2B\\x18\\x6D\\xC2\\x3D\\xFD\\x74\\x43\\x2C\\x43\\xFD\\x4C\\x3C\\x3C\\x3D\\x3D\\x5C\\x9D\\x19\\x88\\x00\\xE5\\x20\\x02\\x00\\x54\\xF5\\x2B\\x5C
```
vi payload.hex，输入`:s/\\\\x//g` 删除 `\\x`，`:w`保存，此时 payload.hex 为
```
4768307374AD000000E0000000789C4B53606098C3C0C0C006C48C40BC51968181094807A716956526A72A042426672B183294F6B03030ACA87263000111A0821F5C602683C74B378619E56E0C39956E0C3B840F33ACE8736368A85ECF34274A97A982E330C391685D2690F8CE9753CB41344C3F323DE1C492860B40F5600C541FAEAF5D0A720B0323A3DC027E0686032B186DC23DFD74432C43FD4C3C3C3D3D5C9D198800E520020054F52B5C
```
payload.hex文件长度346个字符+换行，共347字节。

问deepseek `如何把hex字符串转换为二进制文件`，得到回答是：

```
echo -n "48656C6C6F" | xxd -r -p > output.bin
```
执行 `cat payload.hex | xxd -r -p > payload.bin`，得到文件payload.bin，文件大小为173字节。同时，根据回答中
```
转换后，可以用以下方法检查文件内容：
xxd output.bin  # 查看十六进制转储
```
执行命令`xxd payload.bin`，显示为
```
00000000: 4768 3073 74ad 0000 00e0 0000 0078 9c4b  Gh0st........x.K
00000010: 5360 6098 c3c0 c0c0 06c4 8c40 bc51 9681  S``........@.Q..
00000020: 8109 4807 a716 9565 26a7 2a04 2426 672b  ..H....e&.*.$&g+
00000030: 1832 94f6 b030 30ac a872 6300 0111 a082  .2...00..rc.....
00000040: 1f5c 6026 83c7 4b37 8619 e56e 0c39 956e  .\`&..K7...n.9.n
00000050: 0c3b 840f 33ac e873 6368 a85e cf34 274a  .;..3..sch.^.4'J
00000060: 97a9 82e3 30c3 9168 5d26 90f8 ce97 53cb  ....0..h]&....S.
00000070: 4134 4c3f 323d e1c4 9286 0b40 f560 0c54  A4L?2=.....@.`.T
00000080: 1fae af5d 0a72 0b03 23a3 dc02 7e06 8603  ...].r..#...~...
00000090: 2b18 6dc2 3dfd 7443 2c43 fd4c 3c3c 3d3d  +.m.=.tC,C.L<<==
000000a0: 5c9d 1988 00e5 2002 0054 f52b 5c         \..... ..T.+\
```

问deepseek `Gh0st`得到回答与 Gh0st RAT有关，但无直接有用信息。

在google中搜索`Gh0st RAT 数据包结构`，第一个链接`https://www.freebuf.com/articles/paper/167917.html Gh0st大灰狼RAT家族通讯协议分析`。提到Gh0st通讯实现包结构是
```
|------------|------------|-------------------|-----------------|
|Magic Number|Total Length|Uncompressed Length|Compressed Data  |
|------------|------------|-------------------|-----------------|
|5 Byte      |4 Byte      |4 Byte             |Compressed length|
|------------|------------|-------------------|-----------------|
```
其中Magic Number是`Gh0st`，后续是4字节数据包总长度，4字节解压后的消息长度，zlib压缩后的数据。 总长度0xad = 173, 0xe0 = 224。

问 deepseek `写一段程序解压zlib压缩文件`，参考回答编辑文件` decompress_zlib.py`

```
import zlib
import sys

def decompress_zlib_file(input_path, output_path):
    try:
        # 读取压缩文件的二进制内容
        with open(input_path, 'rb') as f_in:
            compressed_data = f_in.read()
        
        # 解压数据
        decompressed_data = zlib.decompress(compressed_data)
        
        # 保存到输出文件
        with open(output_path, 'wb') as f_out:
            f_out.write(decompressed_data)
        
        print(f"解压成功！输出文件：{output_path}")

    except FileNotFoundError:
        print(f"错误：输入文件 {input_path} 未找到！")
    except zlib.error as e:
        print(f"错误：解压失败，可能不是有效的 zlib 压缩数据。详情：{str(e)}")
    except Exception as e:
        print(f"发生未知错误：{str(e)}")

if __name__ == "__main__":
    # 使用示例：命令行传入输入和输出文件路径
    if len(sys.argv) != 3:
        print("用法：python decompress_zlib.py <输入压缩文件> <输出解压文件>")
        sys.exit(1)
    
    input_file = sys.argv[1]
    output_file = sys.argv[2]
    decompress_zlib_file(input_file, output_file)
```

将前面生成的payload.hex备份，并删除789C之前的Magic Number、Total Length、Uncompressed Length部分，保留如下内容:
```
789C4B53606098C3C0C0C006C48C40BC51968181094807A716956526A72A042426672B183294F6B03030ACA87263000111A0821F5C602683C74B378619E56E0C39956E0C3B840F33ACE8736368A85ECF34274A97A982E330C391685D2690F8CE9753CB41344C3F323DE1C492860B40F5600C541FAEAF5D0A720B0323A3DC027E0686032B186DC23DFD74432C43FD4C3C3C3D3D5C9D198800E520020054F52B5C
```
执行命令 `cat payload.hex | xxd -r -p > zlib.bin`，执行命令 `python3 decompress_zlib.py zlib.bin zlib.out.bin`，输出`解压成功！输出文件：zlib.out.bin`，输出文件zlib.out.bin长度224字节，与数据包头处的Uncompressed Length 0xe0一致。

执行命令`xxd payload.bin`，显示为:
```
00000000: 6620 0000 9c00 0000 0600 0000 0100 0000  f ..............
00000010: b11d 0000 0200 0000 5365 7276 6963 6520  ........Service
00000020: 5061 636b 2031 0075 8c04 0000 a87a 4600  Pack 1.u.....zF.
00000030: 0000 0000 1402 0000 f80a 0000 0000 0000  ................
00000040: 48e9 4600 9877 4600 6c79 4600 b813 c300  H.F..wF.lyF.....
00000050: a88e 4600 807b af02 9c5a 2d02 7808 c300  ..F..{...Z-.x...
00000060: c45b 2d02 b813 c300 b9e9 9577 b813 c300  .[-........w....
00000070: 0000 0000 1402 0000 0000 0000 1402 0000  ................
00000080: 0000 0000 90c8 a480 d05a 2d02 d05a 2d02  .........Z-..Z-.
00000090: d05b 2d02 572f 2b75 0100 0000 0001 011e  .[-.W/+u........
000000a0: a00f 0000 c0a8 013c 5749 4e2d 5439 554e  .......<WIN-T9UN
000000b0: 3448 4949 4845 4300 0000 0000 0000 0000  4HIIHEC.........
000000c0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
000000d0: 0000 0000 0000 0000 0000 0077 0000 0000  ...........w....
```

对比https://www.freebuf.com/articles/paper/167917.html可知:

```
0x66 = 102 是 TOKEN_LOGIN,                    // 上线包
后续是如下结构：
typedef struct
{   
    BYTE            bToken;         // = 1
    OSVERSIONINFOEX OsVerInfoEx;    // 版本信息
    int             CPUClockMhz;    // CPU主频
    IN_ADDR         IPAddress;      // 存储32位的IPv4的地址数据结构
    char            HostName[50];   // 主机名
    bool            bIsWebCam;      // 是否有摄像头
    DWORD           dwSpeed;        // 网速
}LOGININFO;
```
具体分析略，主要内容是192.168.1.60 主机名是WIN-T9UN4HIIHEC向C2服务器发出的上线包。

## 三、安全事件分析

黑客连接被通报IP的80端口，发送了Gh0st RAT远控软件客户端上线包，触发安全规则被发现。

被通报IP的80端口运行的是HTTP服务，并不是Gh0st RAT远控软件服务器端，这些数据包是没有意义的。

## 四、安全事件协查

在网络流量中查找TCP数据包中带有 Gh0st 字样的数据包，能观察到一些payload完全相同的数据包。数据包源地址不同，目的地址也不同，目的端口均是80端口。在运行Apache的WEB服务器上，有如下日志记录：
```
[Thu Mar 20 14:51:54 2025] [error] [client 159.89.147.253] Invalid method in req
uest Gh0st\xad
```

猜测黑客发送这些数据包的目的是探测是否存在Gh0st RAT远控软件服务器端。

这些数据包并不能对服务端造成任何负面影响。


***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
