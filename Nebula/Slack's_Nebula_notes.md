# 搭建 Slack's_Nebula 笔记

### 系统及网络环境

VPS 服务器系统为 Ubuntu 19.10，有公网 IP，```11.22.33.44```，nebula name "vps"

本地客户端系统为 Ubuntu 19.10，内网，无公网 IP，nebula name "vms"

Nebula 目前的版本 v1.1.0，项目的代码仓库 https://github.com/slackhq/nebula

### 在 VPS 服务器

```
ubuntu@server:~# sudo -i
root@server:~# cd /usr/local/bin
root@server:/usr/local/bin# wget https://github.com/slackhq/nebula/releases/download/v1.1.0/nebula-linux-amd64.tar.gz
root@server:/usr/local/bin# tar xvf nebula-linux-amd64.tar.gz
root@server:/usr/local/bin# ls
  nebula  nebula-cert
root@server:/usr/local/bin# rm nebula-linux-amd64.tar.gz
root@server:/usr/local/bin# nebula-cert ca -name "nebulatest, Inc"
root@server:/usr/local/bin# nebula-cert sign -name "vps" -ip "192.168.200.1/24"
root@server:/usr/local/bin# nebula-cert sign -name "vms" -ip "192.168.200.100/24"
root@server:/usr/local/bin# ls
  ca.crt  ca.key  nebula  nebula-cert  vms.crt  vms.key  vps.crt  vps.key  
root@server:/usr/local/bin# mkdir -p /etc/nebula/
root@server:/usr/local/bin# cp ca.crt /etc/nebula/
root@server:/usr/local/bin# cp vps.crt /etc/nebula/
root@server:/usr/local/bin# cp vps.key /etc/nebula/
root@server:/usr/local/bin# cd /etc/nebula/
root@server:/etc/nebula# wget https://github.com/slackhq/nebula/raw/master/examples/config.yml
root@server:/etc/nebula# mv config.yml config.yaml
```
为了方便给客户端拷贝证书文件，将客户端证书文件拷贝到普通用户目录

```
root@server:/usr/local/bin# cp ca.crt /home/ubuntu/
root@server:/usr/local/bin# cp vms.crt /home/ubuntu/
root@server:/usr/local/bin# cp vms.key /home/ubuntu/
```

为了简洁明了输出修改后的 ```config.yaml``` 文件，过滤了空行和注释行，仅保留有效参数
```
root@server:/etc/nebula# cat config.yaml |grep '^\s*[^# \t].*$'
````

下面是修改后的配置文件 ```config.yaml``` 简明版

```
pki:
  ca: /etc/nebula/ca.crt
  cert: /etc/nebula/vps.crt
  key: /etc/nebula/vps.key
static_host_map:
  "192.168.200.1": ["11.22.33.44:4242"]
lighthouse:
  am_lighthouse: true
  interval: 60
listen:
  host: 0.0.0.0
  port: 4242
punchy: true
punch_back: true
  dev: nebula1
  drop_local_broadcast: false
  drop_multicast: false
  mtu: 1300
  routes:
  unsafe_routes:
logging:
  level: info
  format: text
firewall:
  conntrack:
    udp_timeout: 3m
    default_timeout: 10m
    max_connections: 100000
  outbound:
    - port: any
      proto: any
      host: any
  inbound:
    - port: any
      proto: icmp
      host: any
    - port: any
      proto: any
      cida: "192.168.200.1/24"
      groups:
        - laptop
        - home
```

可以用 ```nebula -config /etc/nebula/config.yaml``` 启动 nebula 服务

### 在本地客户端

```
ubuntu@vms:~# sudo -i
root@vms:~# cd /usr/local/bin
root@vms:/usr/local/bin# wget https://github.com/slackhq/nebula/releases/download/v1.1.0/nebula-linux-amd64.tar.gz
root@vms:/usr/local/bin# tar xvf nebula-linux-amd64.tar.gz
root@vms:/usr/local/bin# ls
  nebula  nebula-cert
root@vms:/usr/local/bin# rm nebula-linux-amd64.tar.gz
root@vms:/usr/local/bin# mkdir -p /etc/nebula/
root@vms:/etc/nebula# cd /etc/nebula/
root@vms:/etc/nebula# scp ubuntu@11.22.33.44:~/ca.crt ./
root@vms:/etc/nebula# scp ubuntu@11.22.33.44:~/vms.crt ./
root@vms:/etc/nebula# scp ubuntu@11.22.33.44:~/vms.key ./
root@vms:/etc/nebula# wget https://github.com/slackhq/nebula/raw/master/examples/config.yml
root@vms:/etc/nebula# mv config.yml config.yaml
```

输出一下客户端修改后的配置文件 ```config.yaml``` 简明版

```
pki:
  ca: /etc/nebula/ca.crt
  cert: /etc/nebula/vms.crt
  key: /etc/nebula/vms.key
static_host_map:
  "192.168.200.1": ["11.22.33.44：4242"]
lighthouse:
  am_lighthouse: false
  interval: 60
  hosts:
    - "192.168.200.1"
listen:
  host: 0.0.0.0
  port: 4242
punchy: true
punch_back: true
  dev: nebula1
  drop_local_broadcast: false
  drop_multicast: false
  mtu: 1300
  routes:
  unsafe_routes:
logging:
  level: info
  format: text
firewall:
  conntrack:
    udp_timeout: 3m
    default_timeout: 10m
    max_connections: 100000
  outbound:
    - port: any
      proto: any
      host: any
  inbound:
    - port: any
      proto: icmp
      host: any
    - port: any 
      proto: any 
      cidr: "192.168.100.1/24"
      groups:
        - laptop
        - home
```

用 ```nebula -config /etc/nebula/config.yaml``` 启动 nebula 客户端

### 连接情况，本地 vms Ping VPS

```
ubuntu@vms:~$ ping 192.168.200.1
PING 192.168.200.1 (192.168.200.1) 56(84) bytes of data.
64 bytes from 192.168.200.1: icmp_seq=1 ttl=64 time=320 ms
64 bytes from 192.168.200.1: icmp_seq=2 ttl=64 time=269 ms
64 bytes from 192.168.200.1: icmp_seq=3 ttl=64 time=285 ms
64 bytes from 192.168.200.1: icmp_seq=4 ttl=64 time=288 ms
64 bytes from 192.168.200.1: icmp_seq=5 ttl=64 time=285 ms
64 bytes from 192.168.200.1: icmp_seq=6 ttl=64 time=286 ms
64 bytes from 192.168.200.1: icmp_seq=7 ttl=64 time=300 ms
64 bytes from 192.168.200.1: icmp_seq=8 ttl=64 time=303 ms
64 bytes from 192.168.200.1: icmp_seq=9 ttl=64 time=257 ms
^C
--- 192.168.200.1 ping statistics ---
9 packets transmitted, 9 received, 0% packet loss, time 8016ms
rtt min/avg/max/mdev = 257.137/288.163/319.793/17.391 ms
```
隧道已打通，丢包情况时好时坏。

### 参考教程链接

[Nebula's getting started (quickly)](https://github.com/slackhq/nebula#getting-started-quickly)

[nebula 简单教程](https://runtime.pw/nebula-tutorials/)

[Nebula 简易手册](https://www.khow.me/blog/simple-tutorial-for-nubula.html)
