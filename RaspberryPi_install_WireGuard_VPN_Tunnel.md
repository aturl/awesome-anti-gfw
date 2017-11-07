# 在 Raspberry Pi 2/3 B+ 安装部署 WireGuard，建立 VPN 隧道

## 两端环境

服务器系统为 Ubuntu 17.10

Raspberry Pi 2/3 B+ 操作系统（RASPBIAN STRETCH LITE, 2017-09-07, Kernel 版本 4.9）

### 服务器端安装 WireGuard

服务器端安装 WireGuard 可以参考之前的方法：<a href="WireGuard_VPN_Tunnel.md">在 Ubuntu 部署 VPN 隧道 WireGuard</a>

（将会另写一篇用 wg-quick 快速脚本启动 WireGuard 的内容）

### Raspberry Pi 安装 WireGuard

在 Raspberry Pi 安装 WireGuard 可以参考链接中的方法，详细有效。 [Install WireGuard on Raspberry Pi](https://github.com/adrianmihalko/raspberrypiWireGuard)

另外，还要给 Raspberry Pi 安装 Linux 内核头文件，安装 WireGuard 需要编译内核模块。参考链接：[Raspberry Pi kernel source installer](https://github.com/notro/rpi-source/wiki)

为了方便安装，将内容摘录如下，如有注意事项会注释说明，涉及未知问题请自行研究。

```
sudo apt-get install bc libncurses5-dev
```

#### 安装内核头文件并编译内核模块，用 rpi-source 脚本搞定。

```
sudo wget https://raw.githubusercontent.com/notro/rpi-source/master/rpi-source -O /usr/bin/rpi-source && sudo chmod +x /usr/bin/rpi-source && /usr/bin/rpi-source -q --tag-update
```

```
sudo rpi-source
```

rpi-source 脚本安装时会检查 gcc 版本，如果 gcc 版本低于内核构建的版本，需要升级，如果高于内核构建的版本，跳过就行。
（详细方法参见 [gcc version check](https://github.com/notro/rpi-source/wiki#gcc-version-check)）

检查 Raspberry Pi 已安装的 gcc 版本

```
$ cat /proc/version
Linux version 4.9.58-v7+ (dc4@dc4-XPS13-9333) (gcc version 4.9.3 (crosstool-NG crosstool-ng-1.22.0-88-g8460611) ) #1046 SMP Tue Oct 24 17:07:15 BST 2017
```

```
$ gcc --version | grep gcc
gcc (Raspbian 6.3.0-18+rpi1) 6.3.0 20170516
```

我的 gcc 版本没问题，不用升级，如果有错误提示，给 rpi-source 加上跳过参数

```
sudo rpi-source --skip-gcc
```

### 继续安装 WireGuard

安装编译 WireGuard 所需依赖包

```
sudo apt-get install libmnl-dev build-essential git
```

git WireGuard 源码并编译安装

```
git clone https://git.zx2c4.com/WireGuard
cd WireGuard/
cd src/
make
sudo make install
```

如果没什么错误提示就完成了 WireGuard 安装。

### 两端 VPN 隧道连接情况
（假设服务器端公网 IPv4 是 12.34.56.78 ，客户端映射的内网 IPv4 是 111.222.1.1）

#### 显示服务器端的连接状态

```
$ sudo wg show
interface: wg0
  public key: mh+9HFTbMJKF6tU2sQpoJG1G89OihD+/tHAUWEjjHHU=
  private key: (hidden)
  listening port: 943

peer: 0PvXwI7H0S+CcDMimq/MPbNqSyGDUA5FYjjS2gCy5S0=
  endpoint: 111.222.1.1:51770
  allowed ips: 10.10.0.252/32
  latest handshake: 24 seconds ago
  transfer: 2.91 MiB received, 3.13 MiB sent
```

#### 显示 Raspberry Pi 的连接状态 

```
$ sudo wg show
interface: wg0
  public key: 0PvXwI7H0S+CcrfdBV/MPbNSgCGAUA5FYjjS6Iyy5S0=
  private key: (hidden)
  listening port: 51770
  fwmark: 0xd3d

peer: mh+9HFTbMJKF8UGFEQpoJG1G81AMQ5+/tHAUWLIjHHU=
  endpoint: 12.34.56.78:943
  allowed ips: 0.0.0.0/0
  latest handshake: 56 seconds ago
  transfer: 3.05 MiB received, 3.72 MiB sent
  persistent keepalive: every 25 seconds
```

## 其他

#### 服务器端 wg0.conf 配置文件示例

```
[Interface]
PrivateKey = GBKJGDPE5KDQA9TFvQHqlOeEbJoxs1IH9/WUmo26Unw=
Address = 10.10.0.1/24
ListenPort = 943
PostUp = echo nameserver 8.8.8.8; iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
SaveConfig = true
```
服务器端添加客户端的命令

```
sudo wg set wg0 peer 0PvXwI7H0S+CcDMimq/MPbNqSyGDUA5FYjjS2gCy5S0= allowed-ips 10.10.0.5
```

启动服务器端值守进程

```
sudo wg-quick up wg0
```

停止服务器端值守进程

```
sudo wg-quick down wg0
```

#### 在 Raspberry Pi 上 wg0.conf 配置文件示例

```
[Interface]
PrivateKey = SHBjWA3uAYAZc+TUvr8RCTA5SVQnt+aSVkdlPKhD1Hk=
Address = 10.10.0.5/24
PostUp = echo nameserver 8.8.8.8 | resolvconf -a tun.%i -m 0 -x
PostDown = resolvconf -d tun.%i

[Peer]
PublicKey = mh+9HFTbMJKF8UGFEQpoJG1G81AMQ5+/tHAUWLIjHHU=
Endpoint = 12.34.56.78:943
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

启动 Raspberry Pi 值守进程

```
sudo wg-quick up wg0
```

停止 Raspberry Pi 值守进程

```
sudo wg-quick down wg0
```

#### 关于 WireGuard 默认端口
WireGuard 默认端口是 51820 ，对于 GFW 来说那一个端口都一样，不过换一个也无妨。

服务器端的端口在 wg0.conf 配置文件中指定监听端口就可以。 Raspberry Pi 通过修改 wg-quick 文件约第 152 行

```
sudo vi /usr/bin/wg-quick
```

```
DEFAULT_TABLE=
add_default() {
	if [[ -z $DEFAULT_TABLE ]]; then
		DEFAULT_TABLE=51820 // 默认端口 51820 可以改为其他端口，如 943
		while [[ -n $(ip -4 route show table $DEFAULT_TABLE) || -n $(ip -6 route show table $DEFAULT_TABLE) ]]; do
			((DEFAULT_TABLE++))
		done
```

（用 wg-quick 脚本启动 WireGuard 服务会另写一篇。）
