# 建立自定义 ZeroTier Moons 服务器

#### Moon 的介绍和以下记录的内容参考来源：

ZeroTier 官方手册 [ZeroTier Manual: Creating Your Own Roots](https://www.zerotier.com/manual.shtml#4_4)

好运博客 | 创建 zerotier moon 其实很简单 www.lucktu.com/archives/766.html

___
***ZeroTier 最新版本是 1.8.6 （即将发布1.8.7）可以将操作系统更新至新版本安装  ZeroTier，或者用官方一键安装脚本安装 ```curl -s https://install.zerotier.com | sudo bash```***

***以下记录仅作为个人建立 ZeroTier Moon 服务器的记录，尚未再次验证，如有错误会纠正并更新***

### 服务器环境

服务器系统为 Ubuntu 18.04，已安装高于 GCC/G++ 4.9.3 或 CLANG/CLANG++ 3.4.2 的版本。

### 安装

强烈建议使用 ZeroTier 官方提供的安装方法：

https://www.zerotier.com/download.shtml

快速安装可以使用 ZeroTier 官方提供已编译好的二进制安装包：

https://download.zerotier.com/

```
wget https://download.zerotier.com/RELEASES/1.2.10/dist/debian/bionic/pool/main/z/zerotier-one/zerotier-one_1.2.10_amd64.deb
sudo dpkg -i zerotier-one_1.2.10_amd64.deb
```

也可以从源代码编译安装最新版的 ZeroTier，目前版本 1.2.10 (20180521)。

```
wget https://github.com/zerotier/ZeroTierOne/archive/1.2.10.zip
unzip 1.2.10.zip
cd ZeroTierOne-1.2.10
make && sudo make install
```

执行 ```make``` 结束后打印输出

```
...
strip --strip-all zerotier-one
ln -sf zerotier-one zerotier-idtool
ln -sf zerotier-one zerotier-cli
```

执行 ```sudo make install``` 结束后打印输出

```
mkdir -p /usr/sbin
rm -f /usr/sbin/zerotier-one
cp -f zerotier-one /usr/sbin/zerotier-one
rm -f /usr/sbin/zerotier-cli
rm -f /usr/sbin/zerotier-idtool
ln -s zerotier-one /usr/sbin/zerotier-cli
ln -s zerotier-one /usr/sbin/zerotier-idtool
mkdir -p /var/lib/zerotier-one
rm -f /var/lib/zerotier-one/zerotier-one
rm -f /var/lib/zerotier-one/zerotier-cli
rm -f /var/lib/zerotier-one/zerotier-idtool
ln -s ../../../usr/sbin/zerotier-one /var/lib/zerotier-one/zerotier-one
ln -s ../../../usr/sbin/zerotier-one /var/lib/zerotier-one/zerotier-cli
ln -s ../../../usr/sbin/zerotier-one /var/lib/zerotier-one/zerotier-idtool
mkdir -p /usr/share/man/man8
rm -f /usr/share/man/man8/zerotier-one.8.gz
cat doc/zerotier-one.8 | gzip -9 >/usr/share/man/man8/zerotier-one.8.gz
mkdir -p /usr/share/man/man1
rm -f /usr/share/man/man1/zerotier-idtool.1.gz
rm -f /usr/share/man/man1/zerotier-cli.1.gz
cat doc/zerotier-cli.1 | gzip -9 >/usr/share/man/man1/zerotier-cli.1.gz
cat doc/zerotier-idtool.1 | gzip -9 >/usr/share/man/man1/zerotier-idtool.1.gz
```

将 ```zerotier-one``` 添加到系统服务中，由 Systemctl 管理

```
cd ~/ZeroTierOne-1.2.10
sudo cp debian/zerotier-one.service /etc/systemd/system
sudo systemctl daemon-reload
sudo systemctl enable zerotier-one
sudo systemctl start zerotier-one
```

完成安装后，需要用到 ```zerotier-one``` ```zerotier-idtool``` ```zerotier-cli``` 几个编译好的二进制文件。

### ZeroTier Moon 设置

在 ```/var/lib/zerotier-one``` 目录创建 ```moons.d``` 文件夹，并进入 ```zerotier-one``` 目录 ```/var/lib/zerotier-one```

```
sudo mkdir -p /var/lib/zerotier-one/moons.d && cd /var/lib/zerotier-one/
```

使用 zerotier-idtool，生成 identity.public 文件

```
sudo zerotier-idtool generate identity.public
```

生成后输出内容

```
identity.public written
```

```cat identity.public``` 文件是一串带有 ID 的密钥

```
sudo cat identity.public
```

```
198c26n0dq:0:65875992a5e5a5a6b40fee2428caa459c622c611b70cbfa8503926e4eed63a62a8e4bc9297932eed744629d14e9a47129b343cff9f44wr5f01cf19665td5e6e6:4d9f085b289ff91b036235d4320dc8ae809b616e34fe2cb44092fbae1d5c90b7c85ebcc229564cad722ea75e4b701594d8e2f52c262c7cb1fd8e1ffb48c0f517
```

用 ```zerotier-idtool``` 工具根据 ```identity.public``` 生成 moon 的配置文件 ```moon.json```

```
sudo -s
zerotier-idtool initmoon identity.public >> moon.json
```

```cat moon.json``` 文件内容

```
{
 "id": "198c26n0dq",
 "objtype": "world",
 "roots": [
  {
   "identity": "198c26n0dq:0:65875992a5e5a5a6b40fee2428caa459c622c611b70cbfa8503926e4eed63a62a8e4bc9297932eed744629d14e9a47129b343cff9f44wr5f01cf19665td5e6e6",
   "stableEndpoints": []
  }
 ],
 "signingKey": "65rt56526016017ad16334dacdb2d9ebf93ffab4b2a022be4921195c9c6ace65602bce568a021357867547rf60k8d1a8ecb0503705723fd554c0bbb33efd32ce",
 "signingKey_SECRET": "w54f65fad55d9ec17c0d0c5c1fbe03c5a8bcc80179bbbc52715251a1d04e9fd7f688031692129e4e748ba24867e747b2de4a80b75d71c3c7784j7fd9kpc66399",
 "updatesMustBeSignedBy": "65rt56526016017ad16334dacdb2d9ebf93ffab4b2a022be4921195c9c6ace65602bce568a021357867547rf60k8d1a8ecb0503705723fd554c0bbb33efd32ce",
 "worldType": "moon"
}
```

上面的各项参数中，```198c26n0dq``` 是 Moon 服务器节点的 ID，字符长度 10 位，

```"stableEndpoints": []``` 是空值，需要在此定义 Moon 服务器的 IP 地址和端口，并且需要公网 IP。

如果仅设置 IPv4,是这样 ```"stableEndpoints": ["1.2.3.4/9993"]```，也可以设置 IPv46

完整的内容如下：

```
{
 "id": "198c26n0dq",
 "objtype": "world",
 "roots": [
  {
   "identity": "198c26n0dq:0:65875992a5e5a5a6b40fee2428caa459c622c611b70cbfa8503926e4eed63a62a8e4bc9297932eed744629d14e9a47129b343cff9f44wr5f01cf19665td5e6e6",
   "stableEndpoints": ["1.2.3.4/9993"]
  }
 ],
 "signingKey": "65rt56526016017ad16334dacdb2d9ebf93ffab4b2a022be4921195c9c6ace65602bce568a021357867547rf60k8d1a8ecb0503705723fd554c0bbb33efd32ce",
 "signingKey_SECRET": "w54f65fad55d9ec17c0d0c5c1fbe03c5a8bcc80179bbbc52715251a1d04e9fd7f688031692129e4e748ba24867e747b2de4a80b75d71c3c7784j7fd9kpc66399",
 "updatesMustBeSignedBy": "65rt56526016017ad16334dacdb2d9ebf93ffab4b2a022be4921195c9c6ace65602bce568a021357867547rf60k8d1a8ecb0503705723fd554c0bbb33efd32ce",
 "worldType": "moon"
}
```

然后生成 Moon 服务器的配置文件，生成配置文件需要 ```moon.json``` 中的参数，

```
sudo zerotier-idtool genmoon moon.json
```

执行命令后会输入 ```wrote 000000198c26n0dq.moon (signed world with timestamp 1527057680039)```

上面生成的文件 ```000000198c26n0dq.moon``` 是 Moon 的网络 ID，000000 + Node ID 198c26n0dq 共 16 个字符的长度。

由系统服务启动 moon 将生成的 ```000000198c26n0dq.moon``` 文件拷贝到 moons.d 目录

```
sudo cp 000000198c26n0dq.moon ./moons.d
```

杀死 ```zerotier-one``` 当前进程并重新启动 ```zerotier-one```

```
sudo killall -9 zerotier-one && sudo zerotier-one -d
```

或

```
sudo systemctl restart zerotier-one
```

查看 ZeroTier 状态 ```zerotier-cli info``` 状态输出信息为在线

```
sudo zerotier-cli info
200 info 54y24dho95 1.2.10 ONLINE
```

至此，ZeroTier Moon 服务器暂时建立完成。

### 其他信息

自定义的 ZeroTier Moons 服务器还不是纯粹的根服务器，以 ZeroTier 目前的设计路线，自定义的 ZeroTier Moons 还不能脱离 ZeroTier Root Network (PLANET)，必须依赖根服务，开发者说预计在 1.4 版本才可能实现脱离根服务。

可以在 Raspberry Pi 运行 ZeroTier Moons。

可以建立多个 ZeroTier Moons，将每个服务器的 ```identity``` 和 ```stableEndpoints``` 参数添加在 moon.json 文件， 生成类似的 ```000000198c26n0dq.moon``` 文件并同步到 ```/var/lib/zerotier-one/moons.d``` 目录。

就像 ZeroTier 官方教程中给出的这个示例：

```
{
  "id": "deadbeef00",
  "objtype": "world",
  "roots": [
    {
      "identity": "deadbeef00:0:34031483094...",
      "stableEndpoints": [ "10.0.0.2/9993","2001:abcd:abcd::1/9993" ]
    },
    {
      "identity": "feedbeef11:0:83588158384...",
      "stableEndpoints": [ "10.0.0.3/9993","2001:abcd:abcd::3/9993" ]
    }
  ],
  "signingKey": "b324d84cec708d1b51d5ac03e75afba501a12e2124705ec34a614bf8f9b2c800f44d9824ad3ab2e3da1ac52ecb39ac052ce3f54e58d8944b52632eb6d671d0e0",
  "signingKey_SECRET": "ffc5dd0b2baf1c9b220d1c9cb39633f9e2151cf350a6d0e67c913f8952bafaf3671d2226388e1406e7670dc645851bf7d3643da701fd4599fedb9914c3918db3",
  "updatesMustBeSignedBy": "b324d84cec708d1b51d5ac03e75afba501a12e2124705ec34a614bf8f9b2c800f44d9824ad3ab2e3da1ac52ecb39ac052ce3f54e58d8944b52632eb6d671d0e0",
  "worldType": "moon"
}
```

默认 moon 端口 是 9993，自定义比较麻烦，如果要自定义端口，如下：

```
cd /var/lib/zerotier-one
echo "943" > zerotier-one.port
```

### 桌面客户端连接到自定义的 moon 服务器（以后会移到客户端设置的相关内容中）

下载服务器端生成的 ```000000198c26n0dq.moon``` 文件到本地，拷贝到 ```/var/lib/zerotier-one/moons.d``` 目录

```
sudo mkdir -p /var/lib/zerotier-one/moons.d
sudo cp 000000198c26n0dq.moon /var/lib/zerotier-one/moons.d
sudo systemctl restart zerotier-one
```

```000000198c26n0dq.moon``` 由系统服务加载启动，也可以用 ```zerotier-cli orbit``` 命令手动启动。

```
sudo zerotier-cli orbit 198c26n0dq 8841408a2e
```

```erotier-cli orbit``` 后面的两个 ID，第一个 ID ```198c26n0dq``` 是自己的 Moon ID，即 World ID，

第二个 ID ```8841408a2e``` 是 ZeroTier 的 Root ID。

或者忽略 ID ```8841408a2e```

```
sudo zerotier-cli orbit 198c26n0dq 198c26n0dq
```

列出节点连接信息

```
sudo zerotier-cli listpeers
```

会显示当前的连接状态

```
200 listpeers <ztaddr> <path> <latency> <version> <role>
200 listpeers 198c26n0dq``` - -1 - MOON
200 listpeers 8841408a2e 159.203.2.154/9993;7783;7595 200 1.1.5 PLANET
200 listpeers 9d219039f3 159.203.97.171/9993;7783;2524 241 1.1.5 PLANET
```

上面的，```198c26n0dq``` 为自己的 ID，身份是 ```MOON```，另两个 ```PLANET``` 是 ZeroTier 官方的根服务器。

查看 moons 信息

```
sudo zerotier-cli  listpeers | grep "MOON"
200 listpeers 198c26n0dq - -1 - MOON
```
