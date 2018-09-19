## Shadowsocks-libev 服务器启用 GoQuiet 混淆插件及设置

#### Shadowsocks-libev 服务器系统为 Ubuntu，GoQuiet 版本为 ```gq-server-linux64-1.2.0```

```
cd /usr/local/bin/
wget https://github.com/cbeuw/GoQuiet/releases/download/1.2.0/gq-server-linux64-1.2.0
mv gq-server-linux64-1.2.0 gq-server
chmod +x gq-server
```

上面的步骤，将编译好的 ```gq-server-linux64-1.2.0``` 放在可执行程序目录中 ```/usr/local/bin/```

修改端口权限

```
setcap 'cap_net_bind_service=+ep' /usr/local/bin/gq-server
```

#### Shadowsocks-libev 配置参数, ```config.json``` 在 ```/etc/shadowsocks-libev``` 目录中

```
{
    "server": "0.0.0.0",
    "server_port": 443,
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "password": "mySSpassword",
    "timeout": 1200,
    "method": "chacha20-ietf-poly1305",
    "nameserver": "8.8.8.8",
    "mode": "tcp_and_udp",
    "reuse_port": true,
    "fast_open": true,
    "plugin": "/usr/local/bin/gq-server",
    "plugin_opts": "/etc/shadowsocks-libev/gqserver.json",
    "workers": 1
}
```

#### GoQuiet Server 配置参数, ```gqserver.json``` 在 ```/etc/shadowsocks-libev``` 目录中

```
{
        "WebServerAddr":"127.0.0.1:80",
        "Key":"myGQpassword!"
}
```
