## 用 iPerf3 测量 WireGuard VPN 的网络吞吐量

#### 客户端为中国电信，服务器是 Vultr 东京机房，测量时间为 2018 年 6 月 4 日下午

服务器端参数

```
iperf3 -s -V
```

客户端参数

```
iperf3 -c 10.100.0.1 -u -i 10 -t 30 -b 20G --get-server-output
```

测量输入结果如下，已包含客户端和服务器端的结果：

```
$ iperf3 -c 10.100.0.1 -u -i 10 -t 30 -b 20G --get-server-output
Connecting to host 10.100.0.1, port 5201
[  5] local 10.100.0.10 port 51354 connected to 10.100.0.1 port 5201
[ ID] Interval           Transfer     Bitrate         Total Datagrams
[  5]   0.00-10.00  sec   662 MBytes   555 Mbits/sec  507450  
[  5]  10.00-20.00  sec   650 MBytes   545 Mbits/sec  497949  
[  5]  20.00-30.00  sec   608 MBytes   510 Mbits/sec  466127  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Jitter    Lost/Total Datagrams
[  5]   0.00-30.00  sec  1.87 GBytes   537 Mbits/sec  0.000 ms  0/1471526 (0%)  sender
[  5]   0.00-31.44  sec  54.5 MBytes  14.5 Mbits/sec  0.090 ms  1429719/1471481 (97%)  receiver

Server output:
Time: Mon, 04 Jun 2018 09:32:18 GMT
Accepted connection from 10.100.0.10, port 38484
      Cookie: ktzqr5uslulkc5vs37jblwiodcypbt5wnb75
[  5] local 10.100.0.1 port 5201 connected to 10.100.0.10 port 51354
Starting Test: protocol: UDP, 1 streams, 1368 byte blocks, omitting 0 seconds, 30 second test, tos 0
[ ID] Interval           Transfer     Bitrate         Jitter    Lost/Total Datagrams
[  5]   0.00-1.00   sec  1.81 MBytes  15.2 Mbits/sec  0.051 ms  6194/7583 (82%)  
[  5]   1.00-2.00   sec  2.53 MBytes  21.2 Mbits/sec  0.111 ms  50944/52884 (96%)  
[  5]   2.00-3.00   sec  1.75 MBytes  14.7 Mbits/sec  0.057 ms  50315/51659 (97%)  
[  5]   3.00-4.00   sec  1.62 MBytes  13.6 Mbits/sec  0.059 ms  50663/51907 (98%)  
[  5]   4.00-5.00   sec  1.79 MBytes  15.0 Mbits/sec  0.091 ms  50813/52186 (97%)  
[  5]   5.00-6.00   sec  1.59 MBytes  13.4 Mbits/sec  0.168 ms  51084/52304 (98%)  
[  5]   6.00-7.00   sec  1.78 MBytes  14.9 Mbits/sec  0.080 ms  49398/50759 (97%)  
[  5]   7.00-8.00   sec  1.64 MBytes  13.7 Mbits/sec  0.085 ms  38988/40242 (97%)  
[  5]   8.00-9.00   sec  1.83 MBytes  15.4 Mbits/sec  0.083 ms  47845/49251 (97%)  
[  5]   9.00-10.00  sec  1.64 MBytes  13.8 Mbits/sec  0.087 ms  50896/52152 (98%)  
[  5]  10.00-11.00  sec  1.79 MBytes  15.0 Mbits/sec  0.076 ms  49405/50774 (97%)  
[  5]  11.00-12.00  sec  1.64 MBytes  13.7 Mbits/sec  0.067 ms  48214/49470 (97%)  
[  5]  12.00-13.00  sec  1.71 MBytes  14.3 Mbits/sec  0.056 ms  48201/49508 (97%)  
[  5]  13.00-14.00  sec  1.65 MBytes  13.8 Mbits/sec  0.059 ms  47084/48348 (97%)  
[  5]  14.00-15.00  sec  1.77 MBytes  14.9 Mbits/sec  0.084 ms  39326/40683 (97%)  
[  5]  15.00-16.00  sec  1.62 MBytes  13.6 Mbits/sec  0.125 ms  47356/48598 (97%)  
[  5]  16.00-17.00  sec  1.76 MBytes  14.8 Mbits/sec  0.092 ms  49860/51209 (97%)  
[  5]  17.00-18.00  sec  1.60 MBytes  13.5 Mbits/sec  0.107 ms  50116/51346 (98%)  
[  5]  18.00-19.00  sec  1.83 MBytes  15.3 Mbits/sec  0.089 ms  50441/51840 (97%)  
[  5]  19.00-20.00  sec  1.59 MBytes  13.4 Mbits/sec  0.103 ms  51157/52377 (98%)  
[  5]  20.00-21.00  sec  1.75 MBytes  14.6 Mbits/sec  0.059 ms  50776/52114 (97%)  
[  5]  21.00-22.00  sec  1.64 MBytes  13.8 Mbits/sec  0.118 ms  51043/52301 (98%)  
[  5]  22.00-23.00  sec  1.77 MBytes  14.8 Mbits/sec  0.109 ms  50807/52160 (97%)  
[  5]  23.00-24.00  sec  1.69 MBytes  14.2 Mbits/sec  0.116 ms  50456/51753 (97%)  
[  5]  24.00-25.00  sec  1.79 MBytes  15.0 Mbits/sec  0.123 ms  48515/49886 (97%)  
[  5]  25.00-26.00  sec  1.80 MBytes  15.1 Mbits/sec  0.054 ms  42983/44360 (97%)  
[  5]  26.00-27.00  sec  1.90 MBytes  15.9 Mbits/sec  0.048 ms  38043/39499 (96%)  
[  5]  27.00-28.00  sec  1.70 MBytes  14.3 Mbits/sec  0.072 ms  41219/42525 (97%)  
[  5]  28.00-29.00  sec  1.86 MBytes  15.6 Mbits/sec  0.103 ms  37670/39097 (96%)  
[  5]  29.00-30.00  sec  1.78 MBytes  15.0 Mbits/sec  0.115 ms  42009/43376 (97%)  
[  5]  30.00-31.00  sec  1.83 MBytes  15.3 Mbits/sec  0.106 ms  47095/48496 (97%)  
[  5]  31.00-31.44  sec  41.4 KBytes   767 Kbits/sec  0.090 ms  803/834 (96%)  
- - - - - - - - - - - - - - - - - - - - - - - - -
Test Complete. Summary Results:
[ ID] Interval           Transfer     Bitrate         Jitter    Lost/Total Datagrams
[  5] (sender statistics not available)
[SUM]  0.0-31.4 sec  3 datagrams received out-of-order
[  5]   0.00-31.44  sec  54.5 MBytes  14.5 Mbits/sec  0.090 ms  1429719/1471481 (97%)  receiver
CPU Utilization: local/receiver 2.0% (0.4%u/1.6%s), remote/sender 0.0% (0.0%u/0.0%s)


iperf Done.
```
