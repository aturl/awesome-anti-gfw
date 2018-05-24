## ZeroTier 与 WireGuard 墙外 VPS 和本地客户端都在同一网络环境下 ping 测试

通过 WireGuard VPN 从本地 ping 墙外 VPS

```
--- 192.168.168.1 ping statistics ---
691 packets transmitted, 689 received, 0% packet loss, time 691041ms
rtt min/avg/max/mdev = 47.580/56.199/165.841/9.941 ms
```

通过 WireGuard VPN 从墙外 VPS ping 本地

```
--- 192.168.100.100 ping statistics ---
698 packets transmitted, 694 received, 0% packet loss, time 698189ms
rtt min/avg/max/mdev = 45.652/52.565/334.185/15.399 ms
```

通过 ZeroTier VPN 从本地 ping 墙外 VPS

```
--- 192.168.192.1 ping statistics ---
682 packets transmitted, 576 received, 15% packet loss, time 685726ms
rtt min/avg/max/mdev = 41.699/44.726/111.154/7.264 ms
```

通过 ZeroTier VPN 从墙外 VPS ping 本地

```
--- 192.168.192.100 ping statistics ---
688 packets transmitted, 578 received, +108 errors, 15% packet loss, time 690275ms
rtt min/avg/max/mdev = 41.786/44.935/109.396/7.906 ms, pipe 4
```
