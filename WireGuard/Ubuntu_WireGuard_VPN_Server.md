# 在 Ubuntu 部署 WireGuard VPN 服务器

## 关于 WireGuard

WireGuard 是简单、快速、高效并且安全的开源 VPN 软件，它采用最先进的加密协议，基于 Linux 内核实现。

WireGuard 项目官方网站 https://www.wireguard.com/

WireGuard 源代码由开发者自我托管，代码仓库 https://git.zx2c4.com/WireGuard/

WireGuard 源代码在 GitHub 的镜像仓库 https://github.com/WireGuard

## 在服务器端部署 WireGuard

服务器系统为 Ubuntu 18.04

#### 启用服务器 Linux 内核数据转发

查看当前配置，运行以下命令：

```
sudo sysctl net.ipv4.ip_forward
```

输出 ```net.ipv4.ip_forward = 0``` 为未启用，输出 ```net.ipv4.ip_forward = 1``` 为已启用。

假如未启用，需要设置为启用，运行以下命令：

```
sudo vi /etc/sysctl.conf
```

在 ```/etc/sysctl.conf``` 文件底部添加一行

```
net.ipv4.ip_forward = 1
```

```:wq!``` 保存并退出编辑状态

用 ```sysctl -p``` 命令触发新的内核配置，使其生效。

```
sudo sysctl -p
```

再次查看当前配置，运行以下命令：

```
sudo sysctl net.ipv4.ip_forward
```

输出显示已经启用数据转发：

```
net.ipv4.ip_forward = 1
```

官方提供的快速安装指南 https://www.wireguard.com/install/

### WireGuard 安装

#### 从 PPA 库安装

```
sudo add-apt-repository ppa:wireguard/wireguard
sudo apt-get update
sudo apt-get install wireguard
```

#### 或 Clone 源代码编译安装

```
sudo apt-get install -y git libmnl-dev libelf-dev linux-headers-$(uname -r) build-essential pkg-config
git clone https://git.zx2c4.com/WireGuard
cd WireGuard/src
make
sudo make install
```

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

ListenPort = 监听端口 51820

PrivateKey = 服务器私钥

AllowedIPs = VPN 隧道的内网 IP 段

PostUp 和 PostDown 是 iptables 规则

下面是典型的服务器端 wg0 配置文件（wg0 是网络接口名称）

```
[Interface]
Address = 10.100.0.1/24
SaveConfig = true
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
ListenPort = 51820
PrivateKey = +Cr59JzbCKz9rESqimHGi5C2QfIRYZ5xVMssiTAEqV4=
```

将 ```WireGuard``` 添加到系统服务中，由 Systemctl 管理，分别为重载守护进程、启用服务、禁用服务、启动服务、重启服务、服务状态。

```sudo systemctl daemon-reload``` 

```sudo systemctl enable wg-quick@wg0```

```sudo systemctl disable wg-quick@wg0```

```sudo systemctl start wg-quick@wg0```

```sudo systemctl restart wg-quick@wg0```

```sudo systemctl status wg-quick@wg0```

手动启动或停止 ```WireGuard``` 守护进程：

```sudo wg-quick up wg0```

```sudo wg-quick down wg0```

### 添加客户端

如果已在客户端安装了 WireGuard 并生成了公钥、私钥，服务器端添加命令如下：

```
sudo wg set wg0 peer 8K8h4+xWpAXfL9bJSmhbcFHlvMr2/2ITpNI4Ua998kh= allowed-ips 10.100.0.2
```

```peer 8K8h4+xWpAXfL9bJSmhbcFHlvMr2/2ITpNI4Ua998kh=``` 为客户端公钥

```allowed-ips 10.100.0.2``` 为客户端 IP 地址

检查服务器端设置是否正常，与客户端的 VPN 隧道是否建立成功

```
sudo wg show
```

正常情况应该输出以下内容：

```
interface: wg0
  public key: bco6xIgfp++iFBj6vmDr27tAXfgYsppn/wyUJRcFgUc=
  private key: (hidden)
  listening port: 51820

peer: 8K8h4+xWpAXfL9bJSmhbcFHlvMr2/2ITpNI4Ua998kh=
  endpoint: 12.34.56.78:51820
  allowed ips: 10.100.0.2/32
  latest handshake: 27 seconds ago
  transfer: 8.71 KiB received, 94.77 KiB sent
```

如果有类似上面的输出内容，说明 VPN 隧道已建立，并且有数据传输。

在服务器端可以 ping 客户端，数据包正常也说明隧道成功

```
ping 10.100.0.2
```

服务器端每添加一个客户端节点，便会缓存在服务器端的 ```wg0.conf``` 配置文件中，有过时或无效的客户端节点，用下面命令移除：

```
sudo wg set wg0 peer 88K8h4+xWpAXfL9bJSmhbcFHlvMr2/2ITpNI4Ua998kh= remove
```
