# 从源代码编译安装 Shadowsocks-libev

### 编译安装 Shadowsocks-libev, libsodium, MbedTLS

详细内容见 [Shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev) 在 GitHub 代码仓库，讨论或提问前往 [Shadowsocks-libev's Issues](https://github.com/shadowsocks/shadowsocks-libev/issues) 

```
# Update && Upgrade OS
sudo apt update && sudo apt -y upgrade
sudo apt install shadowsocks-libev

# Installation of basic build dependencies
sudo apt-get install --no-install-recommends gettext build-essential autoconf libtool libpcre3-dev asciidoc xmlto libev-dev libc-ares-dev automake

# Clone source code
git clone https://github.com/shadowsocks/shadowsocks-libev.git
cd shadowsocks-libev
git submodule update --init --recursive

# Installation of libsodium
export LIBSODIUM_VER=1.0.18
wget https://download.libsodium.org/libsodium/releases/libsodium-$LIBSODIUM_VER.tar.gz
tar xvf libsodium-$LIBSODIUM_VER.tar.gz
pushd libsodium-$LIBSODIUM_VER
./configure --prefix=/usr && make
sudo make install
popd
sudo ldconfig

# Installation of MbedTLS
export MBEDTLS_VER=2.16.5
wget https://tls.mbed.org/download/mbedtls-$MBEDTLS_VER-gpl.tgz
tar xvf mbedtls-$MBEDTLS_VER-gpl.tgz
pushd mbedtls-$MBEDTLS_VER
make SHARED=1 CFLAGS="-O2 -fPIC"
sudo make DESTDIR=/usr install
popd
sudo ldconfig

# Start building
./autogen.sh && ./configure && make
sudo make install

cd ..
rm -rf shadowsocks-libev
```

### 安装和设置混淆插件 Cloak

从 Cloak 项目库下载编译好的二进制文件，也可以按项目说明手动编译

Cloak 项目说和使用见 [Cloak](https://github.com/cbeuw/Cloak) 在 GitHub 代码仓库，讨论或提问前往 [Cloak's Issues](https://github.com/cbeuw/Cloak/issues) 

```
wget https://github.com/cbeuw/Cloak/releases/download/v2.1.3/ck-server-linux-amd64-2.1.3
mv ck-server-linux-amd64-2.1.3 ck-server
chmod +x ck-server
```

执行 ```./ck-server -u``` 生成 ```UID``` 和 ```BypassUID```，需要填在插件参数中，参数如下

```
sudo vi /etc/shadowsocks-libev/ckserver.json

{
  "ProxyBook": {
    "shadowsocks": [
      "tcp",
      "127.0.0.1:443"
    ]
  },
  "BindAddr": [
    ":443",
    ":80"
  ],
  "BypassUID": [
    "1rmq6Ag1jZJCImLBIL5wzQ=="
  ],
  "RedirAddr": "204.79.197.200",
  "PrivateKey": "EN5aPEpNBO+vw+BtFQY2OnK9bQU7rvEj5qmnmgwEtUc=",
  "AdminUID": "5nneblJy6lniPJfr81LuYQ==",
  "DatabasePath": "userinfo.db",
  "StreamTimeout": 300
}
```
根据 ```ck-server -u``` 生成的值修改 ```BypassUID```, ```PrivateKey```, ```AdminUID```

### 创建 ```/etc/shadowsocks-libev/config.json```

```
sudo vi /etc/shadowsocks-libev/config.json

{
    "server":"0.0.0.0",
    "server_port":443,
    "local_port":1080,
    "password":"mysspass",
    "timeout":120,
    "method":"chacha20-ietf-poly1305",
    "mode": "tcp_and_udp",
    "nameserver": "8.8.8.8",
    "reuse_port": true,
    "fast_open": true,
    "plugin": "/usr/local/bin/ck-server",
    "plugin_opts": "/etc/shadowsocks-libev/ckserver.json"
}
```

### 修改 ```ss-server``` 和 ```cloak``` 可执行程序的路径

```
sudo -i
cd /usr/bin/
pkill ss-server
cp /usr/local/bin/ss-* /usr/bin/
cp /home/guest/ck-server /usr/local/bin/
chmod +x ck-server
setcap 'cap_net_bind_service=+ep' /usr/local/bin/ck-server
exit
```

### 启用、启动 Shadowsocks-libev 系统服务

```
sudo systemctl enable shadowsocks-libev
sudo systemctl start shadowsocks-libev
sudo systemctl status shadowsocks-libev
```

如果服务状态出错，修改 ```/lib/systemd/system/shadowsocks-libev.service```

```
sudo vi /lib/systemd/system/shadowsocks-libev.service

[Unit]
Description=Shadowsocks-libev Default Server Service
Documentation=man:shadowsocks-libev(8)
After=network-online.target

[Service]
Type=simple
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE
EnvironmentFile=/etc/default/shadowsocks-libev
User=root
Group=nogroup
LimitNOFILE=32768
ExecStart=/usr/bin/ss-server -c $CONFFILE $DAEMON_ARGS

[Install]
WantedBy=multi-user.target
```

将 ```User=nobody``` 修改为 ```User=root```，重启 ```shadowsocks-libev``` 服务

```
sudo systemctl daemon-reload
sudo systemctl restart shadowsocks-libev
sudo systemctl status shadowsocks-libev
```

如果状态正常，客户端便可以连接到这台 Shadowsocks-libev 服务器。
