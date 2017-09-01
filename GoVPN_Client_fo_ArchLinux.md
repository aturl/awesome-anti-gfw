# Arch Linux 安装部署 GoVPN 客户端，建立突破网络封锁和审查的 VPN 隧道
## 关于 GoVPN

GoVPN 是由俄罗斯 “快乐黑客” Sergey Matveev 使用 Go 语言开发的开源免费的 VPN 软件。

GoVPN 简单、安全，可以防止 DPI 深度包检测，可以对抗网络封锁和审查，GoVPN 使用 UDP 协议加密传输流量。

GoVPN 项目官方网站 http://www.govpn.info/

GoVPN 源代码由开发者自我托管，代码仓库 http://git.cypherpunks.ru/cgit.cgi/govpn.git/

GoVPN 最新版本 govpn-7.4 (August 27, 2017)，AUR 仓库为 <a href="https://aur.archlinux.org/packages/govpn">govpn 7.3-1</a>，由 <a href="https://twitter.com/wzyboy">wzyboy</a> 维护。

（题外话，Sergey Matveev 因不认同 GitHub 的某些政策和做法，从 GitHub 仓库删除了 GoVPN 源代码。）

## 服务器端和本地客户端环境

服务器系统如果是 Ubuntu，参见 <a href="Ubuntu_Install_Setup_GoVPN.md">Ubuntu 安装部署 GoVPN</a>

Arch Linux 系统可以按照下面内容设置。Arch Linux 已安装 AUR 软件仓库，已安装 Go

服务器用户名为 bob，本地客户端用户名为 alice

服务器（VPS）可以自由访问国际互联网，有公网 IP，假设为 12.34.56.78，防火墙开放端口 1194（或自定义其他端口）

本地网关为 192.168.1.1，有无公网 IP 无所谓。

建立的 VPN 隧道 IP 段为 172.16.0.1/24，可以自定义为其他内网网段。

---
## 设置服务器端和本地客户端的系统环境
### 服务器端设置略过，参见 <a href="Ubuntu_Install_Setup_GoVPN.md">Ubuntu 安装部署 GoVPN</a>
### 本地客户端操作系统为 Arch Linux，首先更新系统并安装 GoVPN

```
[alice@client ~]$ sudo pacman -Syyuu
[alice@client ~]$ yaourt -Syu --aur
[alice@client ~]$ yaourt -S govpn
```

## 在本地客户端

### 创建客户端 Alice 连接服务器密码短语 key.txt

```
[alice@client ~]$ vi key.txt
```

#### 假定密码短语为 govpntest（也可以是长度为 32 个字符的 Base64 编码）

```
govpntest
```

### 在客户端，生成客户名为 Alice 的验证文件:

```
[alice@client ~]$ govpn-verifier Alice
Passphrase:
```

#### Passphrase: 提示输入密码短语，和上面创建的 key.txt 内容一致，即 govpntest，打印输出内容如下

```
$balloon$s=32768,t=16,p=2$pqrN42u1ruKOWDQUFlEMgg$b0QwK15I7VanKqDAyATrE5VHyL5a+r6h4M8cevPNrxo
$balloon$s=32768,t=16,p=2$pqrN42u1ruKOWDQUFlEMgg
```

#### Alice 的验证文件值为:

```
$balloon$s=32768,t=16,p=2$pqrN42u1ruKOWDQUFlEMgg$b0QwK15I7VanKqDAyATrE5VHyL5a+r6h4M8cevPNrxo
```

---
## 在服务器端
服务器端设置参见：<a href="Ubuntu_Install_Setup_GoVPN.md">Ubuntu 安装部署 GoVPN</a>

---
## 在本地客户端
### 在客户端，为虚拟网卡 tap10 设置 IP 和路由

```
[alice@client ~]$ sudo ip tuntap add dev tap10 mode tap
[alice@client ~]$ sudo ip addr add 172.16.0.2/24 dev tap10
[alice@client ~]$ sudo ip link set up dev tap10
[alice@client ~]$ sudo ip route add 0/1 via 172.16.0.1
[alice@client ~]$ sudo ip route add 128/1 via 172.16.0.1
[alice@client ~]$ sudo ip route add 12.34.56.78 via 192.168.1.1
[alice@client ~]$ sudo ip route del default
[alice@client ~]$ sudo ip route add default dev tap10
```

### 在客户端启动 govpn-client
-key 值为启动客户端的密码短语
-verifier 值为验证密钥
-iface 值为虚拟网卡名称
-remote 值为远程服务器的公网 IP

```
[alice@client ~]$ sudo govpn-client \
    -key /home/alice/work/key.txt
    -verifier '$balloon$s=32768,t=16,p=2$pqrN42u1ruKOWDQUFlEMgg' \
    -iface tap10 \
    -remote 12.34.56.78:1194
```

#### 或者不带换行符的启动命令

```
[alice@client ~]$ sudo govpn-client -key key.txt -verifier '$balloon$s=32768,t=16,p=2$pqrN42u1ruKOWDQUFlEMgg' -iface tap10 -remote 12.34.56.78:1194
```

至此，服务器端和本地客户端的 VPN 隧道已建立完成

---
## 在本地客户端，测试 VPN 隧道的连接状态

#### Ping 服务器端地址 172.16.0.1

```
[alice@client ~]$ ping 172.16.0.1
```
#### ping 输出如下

````
PING 172.16.0.1 (172.16.0.1) 56(84) bytes of data.
64 bytes from 172.16.0.1: icmp_seq=1 ttl=44 time=83.5 ms
64 bytes from 172.16.0.1: icmp_seq=2 ttl=44 time=83.0 ms
64 bytes from 172.16.0.1: icmp_seq=3 ttl=44 time=83.0 ms
64 bytes from 172.16.0.1: icmp_seq=4 ttl=44 time=83.1 ms
64 bytes from 172.16.0.1: icmp_seq=5 ttl=44 time=83.2 ms
64 bytes from 172.16.0.1: icmp_seq=6 ttl=44 time=83.3 ms
64 bytes from 172.16.0.1: icmp_seq=7 ttl=44 time=83.1 ms
64 bytes from 172.16.0.1: icmp_seq=8 ttl=44 time=83.1 ms
^C
--- 172.16.0.1 ping statistics ---
8 packets transmitted, 8 received, 0% packet loss, time 7007ms
rtt min/avg/max/mdev = 83.038/83.212/83.592/0.261 ms
````

#### 说明 VPN 隧道已通

### 用 curl 命令测试隧道的流量转发状态

```
[alice@client ~]$ curl ifconfig.me
```

### 显示 IP 为服务器的公网 IP

```
12.34.56.78
```

curl 获取到了服务器的公网 IP，流量转发成功

---
## Disable GoVPN，禁用 GoVPN 守护进程

本地客户端删除虚拟网卡 tap10、IP 和路由，执行以下命令或者重启系统，将恢复默认设置

```
sudo ip link del dev tap10
sudo ip route del 12.34.56.78 via 192.168.1.1
sudo ip route del default
sudo ip route add default via 192.168.1.1
```

---
## 其他

GoVPN 设计原理和使用方法参考 http://www.cypherpunks.ru/govpn/index.html

如果拥有 IPv4 和 IPv6，设置方法参考开发者给出的示例 http://www.cypherpunks.ru/govpn/Example.html#Example

GoVPN 目前不支持 Windows，Android 以及 iOS 系统。

GoVPN 在数字权利遭强权政府日益侵害的国家，能有效捍卫网络用户的数字权利。

GoVPN 在中国、俄罗斯、伊朗等极权国家，能有效突破网络封锁，突破 GFW 封锁，加密访问被封锁的网站，自由使用互联网。

原创内容，转载请注明出处。
