# Ubuntu 安装部署 GoVPN，建立突破网络封锁和审查的 VPN 隧道

## 关于 GoVPN

GoVPN 是由俄罗斯 “快乐黑客” Sergey Matveev 使用 Go 语言开发的开源免费的 VPN 软件。

GoVPN 简单、安全，可以防止 DPI 深度包检测，可以对抗网络封锁和审查，GoVPN 使用 UDP 协议加密传输流量。

GoVPN 项目官方网站 http://www.govpn.info/

GoVPN 源代码由开发者自我托管，代码仓库 http://git.cypherpunks.ru/cgit.cgi/govpn.git/

GoVPN 最新版本 govpn-7.4 (August 27, 2017) 

（题外话，Sergey Matveev 因不认同 GitHub 的某些政策和做法，从 GitHub 仓库删除了 GoVPN 源代码。）

## 服务器端和本地客户端环境

服务器环境和客户端系统为 Ubuntu 17.04，服务器用户名为 bob，本地客户端用户名为 alice

服务器（VPS）可以自由访问国际互联网，有公网 IP，假设为 12.34.56.78，防火墙开放端口 1194（或自定义其他端口）

本地网关为 192.168.1.1，有无公网 IP 无所谓。

建立的 VPN 隧道 IP 段为 172.16.0.1/24，可以自定义为其他内网网段。

---
## 设置服务器端和本地客户端的系统环境
### 首先更新系统

```
[bob@server ~]$ sudo apt-get update
[bob@server ~]$ sudo apt-get -y upgrade
```

### 安装 Go 1.9

```
[bob@server ~]$ wget https://storage.googleapis.com/golang/go1.9.linux-amd64.tar.gz
[bob@server ~]$ sudo tar -xvf go1.9.linux-amd64.tar.gz
[bob@server ~]$ sudo mv go /usr/local
[bob@server ~]$ export GOROOT=/usr/local/go
[bob@server ~]$ export GOPATH=$HOME/work
[bob@server ~]$ export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```

#### 验证 Go 安装是否成功

```
[bob@server ~]$ go version
```
#### 打印输出内容如下，Go 安装成功

```
go version go1.9 linux/amd64
```
#### 检查 Go 的环境变量配置

```
[bob@server ~]$ go env
```
#### 打印输出内容如下，Go 环境变量配置正确

```
GOARCH="amd64"
GOBIN=""
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOOS="linux"
GOPATH="/home/bob/work"
GORACE=""
GOROOT="/usr/local/go"
GOTOOLDIR="/usr/local/go/pkg/tool/linux_amd64"
GCCGO="gccgo"
CC="gcc"
GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0"
CXX="g++"
CGO_ENABLED="1"
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
```

### Go 安装并配置完毕后，安装编译 GoVPN 所需的依赖包

```
[bob@server ~]$ sudo apt make make-guile
```
### 创建 work 目录，下载 GoVPN 源码并编译

```
[bob@server ~]$ mkdir ～/work
[bob@server ~]$ chmod a+x work
[bob@server ~]$ cd work
[bob@server ~]$ wget http://www.govpn.info/download/govpn-7.4.tar.xz
[bob@server ~]$ wget http://www.govpn.info/download/govpn-7.4.tar.xz.sig
[bob@server ~]$ gpg --verify govpn-7.4.tar.xz.sig govpn-7.4.tar.xz
[bob@server ~]$ tar xf govpn-7.4.tar.xz
[bob@server ~]$ make -C govpn-7.4 all
[bob@server ~]$ cd govpn-7.4
```
---
## 在本地客户端 

### 在 govpn-7.4 目录中创建客户端 Alice 连接服务器密码短语 key.txt

```
[alice@client ~]$ vi key.txt
```
#### 假定密码短语为 govpntest（也可以是长度为 32 个字符的 Base64 编码）

```
govpntest
```
### 在客户端，生成客户名为 Alice 的验证文件:

```
[alice@client ~]$ ./utils/newclient.sh Alice
Passphrase:
```

#### Passphrase: 提示输入密码短语，和上面创建的 key.txt 内容一致，即 govpntest，打印输出内容如下

```
Your client verifier is: $balloon$s=32768,t=16,p=2$pqrN42u1ruKOWDQUFlEMgg
Place the following YAML configuration entry on the server's side:
    Alice:
        up: /path/to/up.sh
        iface: or TUN/TAP interface name
        verifier: $balloon$s=32768,t=16,p=2$pqrN42u1ruKOWDQUFlEMgg$b0QwK15I7VanKqDAyATrE5VHyL5a+r6h4M8cevPNrxo
```

#### Alice 的验证文件值为:
```
$balloon$s=32768,t=16,p=2$pqrN42u1ruKOWDQUFlEMgg$b0QwK15I7VanKqDAyATrE5VHyL5a+r6h4M8cevPNrxo
```
---
## 在服务器端
### 在 govpn-7.4 目录中为客户端节点 Alice 创建配置文件，保存的文件名为 alice.yaml

```
[bob@server ~]$ vi alice.yaml
```
#### verifier: 之后的内容为客户段生成的参数值，和上面的内容一致

```
Alice:
    iface: tap10
    verifier: $balloon$s=32768,t=16,p=2$pqrN42u1ruKOWDQUFlEMgg$b0QwK15I7VanKqDAyATrE5VHyL5a+r6h4M8cevPNrxo
```

### 在服务器端添加虚拟网卡，设置流量转发和 iptables 规则
#### 虚拟网卡为 tap10，设置隧道 IP 段为 172.16.0.1/24

```
[bob@server ~]$ sudo ip tuntap add dev tap10 mode tap
[bob@server ~]$ sudo ip addr add 172.16.0.1/24 dev tap10
[bob@server ~]$ sudo ip link set up dev tap10
```

#### 设置服务器的 NAT 流量转发

```
[bob@server ~]$ sudo vi /etc/sysctl.conf
```
#### 找到这一行 #net.ipv4.ip_forward = 1 去掉注释符 “#”

```
net.ipv4.ip_forward = 1
```
```
[bob@server ~]$ sudo sysctl -p
```

#### 添加 iptables 规则，将 tap10 请求的流量全部通过 ens4（物理网卡）转发

```
[bob@server ~]$ sudo iptables -A FORWARD -i tap10 -j ACCEPT
[bob@server ~]$ sudo iptables -A FORWARD -o tap10 -j ACCEPT
[bob@server ~]$ sudo iptables -t nat -A POSTROUTING -o tap10 -j MASQUERADE
[bob@server ~]$ sudo iptables -t nat -A POSTROUTING -o ens4 -j MASQUERADE
```

#### 在服务器端启动 GoVPN 的守护进程

```
[bob@server ~]$ sudo ./govpn-server -conf alice.yaml -bind 0.0.0.0:1194
```
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
### 在客户端的 govpn-7.4 目录中启动 govpn-client
-key 值为启动客户端的密码短语
-verifier 值为验证密钥
-iface 值为虚拟网卡名称
-remote 值为远程服务器的公网 IP

```
[alice@client ~]$ ./govpn-client \
    -key /home/alice/work/govpn-7.4/key.txt
    -verifier '$balloon$s=32768,t=16,p=2$pqrN42u1ruKOWDQUFlEMgg' \
    -iface tap10 \
    -remote 12.34.56.78:1194
```

#### 或者不带换行符输的启动命令

```
[alice@client ~]$ ./govpn-client -key key.txt -verifier '$balloon$s=32768,t=16,p=2$pqrN42u1ruKOWDQUFlEMgg' -iface tap10 -remote 12.34.56.78:1194
```
---
## 至此，服务器端和本地客户端的 VPN 隧道已建立完成
### 测试隧道的连接状态
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
```
12.34.56.78
```

#### curl 获取到了服务器的公网 IP，流量转发成功
---
## Disable GoVPN，禁用 GoVPN 守护进程

删除添加的虚拟网卡、IP 和路由，或者重启系统，将恢复默认设置

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

GoVPN 目前不支持 Windows 系统和 Android 以及 iOS 系统。

GoVPN 在数字权利遭强权政府日益侵害的国家，能有效捍卫网络用户的数字权利。

GoVPN 在中国、俄罗斯、伊朗等极权国家，能有效突破网络封锁，突破 GFW 封锁，加密访问被封锁的网站，自由使用互联网。
