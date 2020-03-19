# 在 Raspberry Pi 2/3/4 B+ 安装部署 WireGuard，建立 VPN 隧道

## Raspberry Pi 版本和系统情况

Raspberry Pi 2/3/4 B+ 操作系统（ ≥ RASPBIAN STRETCH LITE, 2017-09-07, Kernel 版本 4.9）

### 服务器端安装 WireGuard

可以通过各种 Linux 发行版的软件仓库安装 [WireGuard Installation](https://www.wireguard.com/install/)

编译安装文档可以参考 WireGuard 官方文档 [Compiling the Kernel Module from Source](https://www.wireguard.com/compilation/)

### Raspberry Pi 安装 WireGuard

编译安装文档可以参考 WireGuard 官方的 [Compiling the Kernel Module from Source](https://www.wireguard.com/compilation/)

```
pi@raspberrypi:~$ sudo apt-get install libelf-dev raspberrypi-kernel-headers build-essential pkg-config
pi@raspberrypi:~$ git clone https://git.zx2c4.com/wireguard-linux-compat
pi@raspberrypi:~$ git clone https://git.zx2c4.com/wireguard-tools
pi@raspberrypi:~$ sudo make -C wireguard-linux-compat/src -j$(nproc)
pi@raspberrypi:~$ sudo make -C wireguard-linux-compat/src install
pi@raspberrypi:~$ sudo make -C wireguard-tools/src -j$(nproc)
pi@raspberrypi:~$ sudo make -C wireguard-tools/src install
```

#### 在 Raspberry Pi 上 wg0.conf 配置文件示例以及启动服务

```
pi@raspberrypi:~$ sudo vi /etc/wireguard/wg0.conf
```

编辑 ```wg0.conf``` 对应服务器的配置内容

```
[Interface]
PrivateKey = SHBjWA3uAYAZc+TUvr8RCTA5SVQnt+aSVkdlPKhD1Hk=
Address = 10.10.0.5/24
DNS = 8.8.8.8
MTU = 1472

[Peer]
PublicKey = mh+9HFTbMJKF8UGFEQpoJG1G81AMQ5+/tHAUWLIjHHU=
Endpoint = 12.34.56.78:943
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

启动或停止 Raspberry Pi 值守进程

```
sudo wg-quick up wg0
sudo wg-quick down wg0
```

```
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```

如果没什么错误提示就完成了 WireGuard 安装。用 ```sudo wg``` 命令显示连接状态。
