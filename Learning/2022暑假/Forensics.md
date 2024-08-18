# 取证隐写

[TOC]

## 文件格式
1. `file xxx`


通过文件头识别文件类型

常见文件头：<https://www.garykessler.net/library/file_sigs.html>

> png: 89 50 4E 47 0D 0A 1A 0A

2. `strings xxx | grep xxx`

打印文件中可打印的字符，经常用来发现文件中的一些提示信息或是一些特殊的编码信息，常常用来发现题目的突破口。

3. `binwalk -e xxx`

根据文件头识别一个文件中的其它文件

4. `exiftool`

查看元数据
> 图片
> - 日期
> - 相机参数
> - GPS
> 
> 音乐
> - 名称
> - 歌手
> - 专辑
> 
> **...** 

## 时间戳 Timestamps

文件修改，读取，创建 (MAC) 时间，来源于Unix文件系统，Windows系统可能有出入

- mtime: Modification time
- atime: Accesse time
- ctime: Change time (Unix) and creation time or birth time (Windows)

e.g. 
- 打开文件 $\rightarrow$ Accessed
- 文件重命名 $\rightarrow$ Accessed & Changed
- 复制 $\rightarrow$ Accessed & Modified & Changed & Birth
- 移动 $\rightarrow$ Changed

查看文件 timestamps:
- Windows: AccessData FTK Imager
- Linux: `stat xxx`

## png图片

（固定）八个字节89 50 4E 47 0D 0A 1A 0A为png的文件头  

（固定）四个字节00 00 00 0D（即为十进制的13）代表数据块的长度为13  

（固定）四个字节49 48 44 52（即为ASCII码的IHDR）是文件头数据块的标示（IDCH）  

（可变）13位数据块（IHDR)  

- 前四个字节代表该图片的宽  

- 后四个字节代表该图片的高  

- 后五个字节依次为Bit depth、ColorType、Compression method、Filter method、Interlace method
 
（可变）剩余四字节为该png的CRC (循环冗余校验)检验码，由从IDCH到IHDR的十七位字节进行crc计算得到。

> 可以使用python zlib库中的函数计算CRC校验值
> ```
> import zlib
> 
> with open('1.png','rb') as image_data:
>     bin_data=image_data.read()
> data = bytearray(bin_data[12:29])#截取待计算的字符串
> crc32key = zlib.crc32(data)#使用函数计算
> print(hex(crc32key))
> ```
> 根据CRC校验值暴力破解png图片正确的宽高
> ```
> import zlib
> import struct
> 
> filename = '2.png'
> with open(filename, 'rb') as f:
>    all_b = f.read()
>    crc32key = int(all_b[29:33].hex(), 16)
>    data = bytearray(all_b[12:29])
>    n = 4095  # 理论上0xffffffff,但考虑到屏幕实际/cpu，0x0fff就差不多了
>    for w in range(n):  # 高和宽一起爆破
>        width = bytearray(struct.pack('>i', w))  # q为8字节，i为4字节，h为2字节
>        for h in range(n):
>            height = bytearray(struct.pack('>i', h))
>            for x in range(4):
>                data[x + 4] = width[x]
>                data[x + 8] = height[x]
>            crc32result = zlib.crc32(data)
>            if crc32result == crc32key:
>                # 2021.7.20，有时候显示的宽高依然看不出具体的值，干脆输出data部分
>                print(data.hex())
>                print("宽为：", end="")
>                print(width)
>                print("高为：", end="")
>                print(height)
>                exit(0)
>```
IEND: `00 00 00 00 49 45 4E 44 AE 42 60 82`

### LSB隐写

LSB 全称 Least Significant Bit，最低有效位。PNG 文件中的图像像数一般是由 RGB 三原色（红绿蓝）组成，每一种颜色占用 8 位，取值范围为 0x00 至 0xFF，即有 256 种颜色，一共包含了 256 的 3 次方的颜色。

LSB 隐写就是修改 RGB 颜色分量的最低二进制位中的最低的 1 bit，而人类的眼睛不会注意到这前后的变化。

可以通过 Stegsolve 分别查看各个通道的信息

## 流量包 Wireshark

### PCAP 修复
使用[pcapfix](http://f00l.de/hacking/pcapfix.php)

### 过滤方法
- 显示过滤器表达式 Analyzez-Display Filter Expressions...
- 输入栏内输入
- 将数据包中某个属性作为过滤值

### 信息统计 Statistics

- 协议分级 Protocal Hierarchy

| 名称 | 含义 |
| -- | -- |
| Protocol | 协议名称 |
| % Packets | 含有该协议的包数目在捕捉文件所有包所占的比例 |
| Packets | 含有该协议的包的数目 |
| Bytes | 含有该协议的字节数 |
| Mbit/s | 抓包时间内的协议带宽 |
| End Packets | 该协议中的包的数目（作为文件中的最高协议层） |
| End Bytes | 该协议中的字节数（作为文件中的最高协议层） |
| End Mbit/s | 抓包时间内的协议带宽（作为文件中的最高协议层） |

- 对话 Conversations 

  发生于特定端点IP间的所有流量

- 端点 Endpoints

- HTTP

### 解密SSL流量
> SSL / TLS 握手详细过程
> 1. **"client hello"消息**：客户端通过发送"client hello"消息向服务器发起握手请求，该消息包含了客户端所支持的 TLS 版本和密码组合以供服务器进行选择，还有一个"client random"随机字符串。
> 2. **"server hello"消息**：服务器发送"server hello"消息对客户端进行回应，该消息包含了数字证书，服务器选择的密码组合和"server random"随机字符串。
> 3. **验证**：客户端对服务器发来的证书进行验证，确保对方的合法身份，验证过程可以细化为以下几个步骤：
>    - 检查数字签名
>    - 验证证书链 (这个概念下面会进行说明)
>    - 检查证书的有效期
>    - 检查证书的撤回状态 (撤回代表证书已失效)
> 4. **"premaster secret"字符串**：客户端向服务器发送另一个随机字符串"premaster secret (预主密钥)"，这个字符串是经过服务器的公钥加密过的，只有对应的私钥才能解密。
> 5. **使用私钥**：服务器使用私钥解密"premaster secret"。
> 6. **生成共享密钥**：客户端和服务器均使用 client random，server random 和 premaster secret，并通过相同的算法生成相同的共享密钥 KEY。
> 7. **客户端就绪**：客户端发送经过共享密钥 KEY加密过的"finished"信号。
> 8. **服务器就绪**：服务器发送经过共享密钥 KEY加密过的"finished"信号。
> 9. **达成安全通信**：握手完成，双方使用对称加密进行安全通信。

需要(其一即可)
- 共享密钥（
- client random，server random 和premaster secret，再生成KEY
- (如果使用了RSA) 暴力破解RSA得到解密后的premaster secret

Edit-Preferences-Protocol-SSL/TLS

## 磁盘取证

### 查看磁盘及分区
```
fdisk -l
```

获取整个磁盘镜像文件 `dd`
```
dd if=需要拷贝的磁盘 of=/存储目录/镜像文件 （确保存储目录有足够的空间）
```

### 远程取硬盘数据 `nc` `netcat`

建立端口连接
```
nc -nv 192.168.1.1 80
```
服务器开放端口
```
nc -l -p 1234
```

### 文件传输

客户端发送文件
```
nc -nv 192.168.2.121 1235 < target.doc
```

服务端接收文件
```
nc -lp 1234 > text.doc
```

### 远程克隆硬盘

客户端
```
dd if=/dev/sda2 | nc 192.168.10.11 4445
```

服务器
```
nc -l -p 4445 | dd of=/tmp/sda2.dd
```

挂载硬盘数据：AccessData FTK Imager 

## 内存取证 `Volatility`

### 基本命令
```
python vol.py -f IMAGE ‐-profile=PROFILE PLUGIN

-f 后面加的是要取证的文件
--profile 后加的是工具识别出的系统版本
plugin 是指使用的插件，其中默认存在一些插件，另外还可以自己下载一些插件扩充

可以使用 -h 或 --info 参数获取使用方法和插件介绍
```
### 常用命令

`imageinfo` 显示目标镜像的摘要信息，这常常是第一步，获取内存的操作系统类型及版本，之后可以在 –profile 中带上对应的操作系统，后续操作都要带上这一参数

`pslist` 该插件列举出系统进程，但它不能检测到隐藏或者解链的进程，psscan可以

`pstree` 以树的形式查看进程列表，和pslist一样，也无法检测隐藏或解链的进程

`psscan` 可以找到先前已终止(不活动)的进程以及被rootkit隐藏或解链的进程

`cmdscan` 可用于查看终端记录

`notepad` 查看当前展示的 notepad 文本（–profile=winxp啥的低版本可以，win7的不行，可以尝试使用editbox）

`filescan` 扫描所有的文件列表，linux配合 grep 命令进行相关字符定向扫描，如：`grep flag`、`grep -E ‘png|jpg|gif|zip|rar|7z|pdf|txt|doc’`

`dumpfiles` 导出某一文件(指定虚拟地址)，需要指定偏移量 `-Q` 和输出目录 `-D`

`mendump` 提取出指定进程，常用foremost 来分离里面的文件，需要指定进程`-p [pid]` 和输出目录 `-D`

`editbox` 显示有关编辑控件（曾经编辑过的内容）的信息

`screenshot` 保存基于GDI窗口的伪截屏

`clipboard` 查看剪贴板信息

`iehistory` 检索IE浏览器历史记录

`systeminfo` 显示关于计算机及其操作系统的详细配置信息（插件）

`hashdump` 查看当前操作系统中的 password hash，例如 Windows 的 SAM 文件内容(mimikatz插件可以获取系统明文密码)

`mftparser` 恢复被删除的文件

`svcscan` 扫描 Windows 的服务

`connscan` 查看网络连接

`envars` 查看环境变量

`dlllist` 列出某一进程加载的所有dll文件

`hivelist` 列出所有的注册表项及其虚拟地址和物理地址

`timeliner` 将所有操作系统事件以时间线的方式展开

### 其它命令

<https://github.com/volatilityfoundation/volatility/wiki/Command-Reference>

___

