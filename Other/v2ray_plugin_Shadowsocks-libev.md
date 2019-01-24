## Shadowsocks-libev 服务器启用 v2ray-plugin 插件及设置

#### Shadowsocks-libev 服务器系统为 Ubuntu，v2ray-plugin 版本为 ```v2ray-plugin v1.0```

```
cd /usr/local/bin/
wget https://github.com/shadowsocks/v2ray-plugin/releases/download/v1.0/v2ray-plugin-linux-amd64-8cea1a3.tar.gz
mv v2ray-plugin-linux-amd64-8cea1a3.tar.gz v2ray-plugin
chmod +x v2ray-plugin
```

上面的步骤中，将编译好的二进制文件 ```v2ray-plugin``` 放在可执行程序目录中 ```/usr/local/bin/```

修改端口权限

```
setcap 'cap_net_bind_service=+ep' /usr/local/bin/v2ray-plugin
```

#### Shadowsocks-libev 配置参数, ```config.json``` 在 ```/etc/shadowsocks-libev``` 目录中

```
{
    "server": "0.0.0.0",
    "server_port": 443,
    "password": "mySSpassword!",
    "timeout": 360,
    "user":"nobody",
    "method": "chacha20-ietf-poly1305",
    "nameserver": "8.8.8.8",
    "fast_open":true,
    "reuse_port":true,
    "no_delay":true,
    "mode":"tcp_only",
    "plugin": "/usr/local/bin/v2ray-plugin",
    "plugin_opts":"server;tls;host=anti-gfw.io;cert=/home/ss/.acme.sh/anti-gfw.io/fullchain.cer;key=/home/ss/.
acme.sh/anti-gfw.io/anti-gfw.io.key"
}
``` 

```"plugin_opts"``` 参数设置可以参考 [v2ray-plugin's commands](https://github.com/LiveChief/v2ray-plugin/blob/c9f23a36d5e8a269a3d3714e260512ca50f8c65c/README.md)

在客户端，```"plugin_opts"``` 参数值为 ```"tls;host=anti-gfw.io"```
