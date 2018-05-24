# 在 Ubuntu 部署 VPN 隧道 WireGuard
## 关于 WireGuard

WireGuard 是简单、快速、高效并且安全的开源 VPN 软件，它采用最先进的加密协议，基于 Linux 内核实现。

WireGuard 项目官方网站 https://www.wireguard.com/

WireGuard 源代码由开发者自我托管，代码仓库 https://git.zx2c4.com/WireGuard/

WireGuard 源代码在 GitHub 的镜像仓库 https://github.com/WireGuard

## 在服务器端部署 WireGuard
WireGuard 跨平台，参照官方给出的快速安装指南 https://www.wireguard.com/install/

---
### 服务器环境以 Ubuntu 系统为例
#### 生成公钥、私钥、共享密钥

```
sudo mkdir -p /etc/wireguard && sudo chmod 0777 /etc/wireguard && cd /etc/wireguard
umask 077
wg genkey | tee privatekey | wg pubkey > publickey | wg genpsk > presharedkey
```

打印输出私钥
```
cat privatekey
+Cr59JzbCKz9rESqimHGi5C2QfIRYZ5xVMssiTAEqV4=
```

打印输出公钥
```
cat publickey
bco6xIgfp++iFBj6vmDr27tAXfgYsppn/wyUJRcFgUc=
```

打印输出共享密钥（可以不生成，配置文件中不是必须的）
```
cat presharedkey
Vv0MdBNolqbnsBPQPf0ttJecOw2QC8QqWBVieNtvoIo=
```

#### 编辑并保存 wg0 配置文件 wg0.conf

```
sudo vi wg0.conf
```
#### wg0.conf 配置文件内容如下：
ListenPort = 监听端口 1194

PrivateKey = 服务器私钥

PrivateKey = 连接节点公钥（由客户端生成）

AllowedIPs = VPN 隧道的内网 IP 段

```
[Interface]
ListenPort = 1194
PrivateKey = +Cr59JzbCKz9rESqimHGi5C2QfIRYZ5xVMssiTAEqV4=

[Peer]
PublicKey = 1lq23n/oNwYosnf0xMadtrIcC+droND9fg/wiS0aEhw=
AllowedIPs = 10.100.0.1/24
```

#### 设置服务器的 NAT 流量转发

```
sudo vi /etc/sysctl.conf
```

#### 找到这一行 #net.ipv4.ip_forward = 1 去掉注释符 “#”

```
net.ipv4.ip_forward = 1
```

#### 执行 sysctl 并生效

```
sudo sysctl -p
```

#### 在服务器端添加虚拟网卡 wg0，设置隧道 IP 和 iptables 规则
```
sudo ip link add dev wg0 type wireguard
sudo ip address add dev wg0 10.100.0.1/24
sudo ip link set wg0 up
sudo wg setconf wg0 /etc/wireguard/wg0.conf
sudo iptables -A FORWARD -i wg0 -j ACCEPT
sudo iptables -A FORWARD -o wg0 -j ACCEPT
sudo iptables -t nat -A POSTROUTING -o wg0 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -o ens4 -j MASQUERADE
```

检查 wg 设置是否正常
```
sudo wg show
```

#### 正常情况应该输出以下内容：
```
interface: wg0
  public key: 1lq23n/oNwYosnf0xMadtrIcC+droND9fg/wiS0aEhw=
  private key: (hidden)
  listening port: 1194

peer: 8+9jGlxuwyGUWtUk8/NZMAl1Ax577KAvnXJY0EeuNkQ=
  endpoint: 12.34.56.78:61353
  allowed ips: 10.100.0.0/24
  latest handshake: 0 days, 8 minutes, 19 seconds ago
  transfer: 0.60 GiB received, 0.93 GiB sent
```

服务器端设置完成。

---
### 设置客户端
#### 客户端系统 Ubuntu/Debian/ArchLinux 参照官方给出的快速安装指南 https://www.wireguard.com/install/

```
sudo mkdir -p /etc/wireguard && sudo chmod 077 /etc/wireguard && cd /etc/wireguard
umask 077
wg genkey | tee privatekey | wg pubkey > publickey
```
打印输出私钥
```
cat privatekey
SHBjWA3uAYAZc+TUvr6PcTA5SVQnt+aSVkdlZhlg1Hk=
```

打印输出公钥
```
cat publickey
8+9jGlxuwyGUWtUk8/NZMAl1Ax577KAvnXJY0EeuNkQ=
```

#### 编辑并保存 wg0 配置文件 wg0.conf

```
sudo vi wg0.conf
```
#### wg0.conf 配置文件内容如下：
ListenPort = 监听端口 1194

PrivateKey = 本地客户端私钥

PrivateKey = 服务器端公钥（由服务器端生成）

AllowedIPs = VPN 隧道的内网 IP 段

Endpoint = 远程服务器公网 IP 和端口

PersistentKeepalive = 连接心跳包的唤醒间隔

```
[Interface]
ListenPort = 1194
PrivateKey = SHBjWA3uAYAZc+TUvr6PcTA5SVQnt+aSVkdlZhlg1Hk=

[Peer]
PublicKey = bco6xIgfp++iFBj6vmDr27tAXfgYsppn/wyUJRcFgUc=
AllowedIPs = 0.0.0.0/0
Endpoint = 12.34.56.78:1194
PersistentKeepalive = 25
```

### 在本地客户端
### 在客户端，添加虚拟网卡 wg0 并设置 IP 和路由

```
sudo ip link add dev wg0 type wireguard
sudo ip address add dev wg0 10.100.0.101/24
sudo ip link set wg0 up
sudo wg setconf wg0 /etc/wireguard/wg0.conf
sudo ip route add 12.34.56.78 via 192.168.1.1
sudo ip route del default
sudo ip route add default dev wg0
```
#### 客户端设置完成，Ping 测试 VPN 隧道

```
ping 10.100.0.1
```

#### ping 输出如下

````
PING 10.100.0.1 (10.100.0.1) 56(84) bytes of data.
64 bytes from 10.100.0.1: icmp_seq=1 ttl=44 time=103.50 ms
64 bytes from 10.100.0.1: icmp_seq=2 ttl=44 time=103.50 ms
64 bytes from 10.100.0.1: icmp_seq=3 ttl=44 time=103.50 ms
64 bytes from 10.100.0.1: icmp_seq=4 ttl=44 time=103.50 ms
64 bytes from 10.100.0.1: icmp_seq=5 ttl=44 time=103.50 ms
64 bytes from 10.100.0.1: icmp_seq=6 ttl=44 time=103.50 ms
64 bytes from 10.100.0.1: icmp_seq=7 ttl=44 time=103.50 ms
64 bytes from 10.100.0.1: icmp_seq=8 ttl=44 time=103.50 ms
^C
--- 10.100.0.1 ping statistics ---
8 packets transmitted, 8 received, 0% packet loss, time 828ms
rtt min/avg/max/mdev = 103.50/103.50/103.50/0.00 ms
````

说明 VPN 隧道已通

用 curl 命令测试隧道的流量转发状态

```
curl ifconfig.me
```

显示 IP 为服务器的公网 IP

```
12.34.56.78
```

curl 获取到了服务器的公网 IP，流量转发成功

---
### isable WireGuard，禁用 WireGuard

删除添加的虚拟网卡、IP 和路由，或者重启系统，将恢复默认设置

#### 服务器端
```
sudo ip link del dev wg0
```

#### 本地客户端
```
sudo ip link del dev wg0
sudo ip route del 12.34.56.78 via 192.168.1.1
sudo ip route del default
sudo ip route add default via 192.168.1.1
```

## 其他

WireGuard 设计原理和使用方法参考 https://www.wireguard.com/

WireGuard 跨平台，支持各种 Linux 发行版，目前 Windows 和 macOS 客户端（Go 实现）以及 Android 应用都在开发中。

WireGuard 在数字权利遭受强权政府日益侵害的国家，能有效捍卫网络用户的数字权利。

WireGuard GoVPN 在中国、俄罗斯、伊朗等网络审查严重的国家，能有效突破网络封锁，突破 GFW 封锁，加密访问被封锁的网站，自由使用互联网。

原创内容，转载请注明出处。
