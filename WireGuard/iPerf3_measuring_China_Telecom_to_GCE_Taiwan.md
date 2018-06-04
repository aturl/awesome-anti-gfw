## 用 iPerf3 测量 WireGuard VPN 的网络吞吐量

#### 客户端为中国电信，服务器是 Google Cloud Platform (GCE) 台湾数据中心，测量时间为 2018 年 6 月 4 日下午

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
[  5] local 10.100.0.10 port 59833 connected to 10.100.0.1 port 5201
[ ID] Interval           Transfer     Bitrate         Total Datagrams
[  5]   0.00-10.00  sec   644 MBytes   540 Mbits/sec  508408  
[  5]  10.00-20.00  sec   655 MBytes   549 Mbits/sec  517060  
[  5]  20.00-30.00  sec   668 MBytes   561 Mbits/sec  527805  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Jitter    Lost/Total Datagrams
[  5]   0.00-30.00  sec  1.92 GBytes   550 Mbits/sec  0.000 ms  0/1553273 (0%)  sender
[  5]   0.00-30.61  sec  81.5 MBytes  22.3 Mbits/sec  0.051 ms  1488907/1553252 (96%)  receiver

Server output:
Time: Mon, 04 Jun 2018 09:29:05 GMT
Accepted connection from 10.100.0.10, port 38482
      Cookie: fw2muf2ew73ihswymxiuuzhkc7b4463d7ct7
[  5] local 10.100.0.1 port 5201 connected to 10.100.0.10 port 59833
Starting Test: protocol: UDP, 1 streams, 1328 byte blocks, omitting 0 seconds, 30 second test, tos 0
[ ID] Interval           Transfer     Bitrate         Jitter    Lost/Total Datagrams
[  5]   0.00-1.00   sec  6.16 MBytes  51.6 Mbits/sec  0.051 ms  32061/36922 (87%)  
[  5]   1.00-2.00   sec  2.56 MBytes  21.5 Mbits/sec  0.072 ms  48614/50638 (96%)  
[  5]   2.00-3.00   sec  2.56 MBytes  21.5 Mbits/sec  0.077 ms  49589/51613 (96%)  
[  5]   3.00-4.00   sec  2.56 MBytes  21.5 Mbits/sec  0.055 ms  49105/51130 (96%)  
[  5]   4.00-5.00   sec  2.56 MBytes  21.5 Mbits/sec  0.052 ms  48631/50655 (96%)  
[  5]   5.00-6.00   sec  2.56 MBytes  21.5 Mbits/sec  0.112 ms  48835/50860 (96%)  
[  5]   6.00-7.00   sec  2.56 MBytes  21.5 Mbits/sec  0.060 ms  48968/50992 (96%)  
[  5]   7.00-8.00   sec  2.56 MBytes  21.5 Mbits/sec  0.065 ms  48426/50451 (96%)  
[  5]   8.00-9.00   sec  2.56 MBytes  21.5 Mbits/sec  0.095 ms  48706/50730 (96%)  
[  5]   9.00-10.00  sec  2.56 MBytes  21.5 Mbits/sec  0.061 ms  48507/50531 (96%)  
[  5]  10.00-11.00  sec  2.56 MBytes  21.5 Mbits/sec  0.075 ms  48678/50702 (96%)  
[  5]  11.00-12.00  sec  2.56 MBytes  21.5 Mbits/sec  0.080 ms  48818/50843 (96%)  
[  5]  12.00-13.00  sec  2.56 MBytes  21.5 Mbits/sec  0.067 ms  48723/50747 (96%)  
[  5]  13.00-14.00  sec  2.56 MBytes  21.5 Mbits/sec  0.060 ms  48583/50607 (96%)  
[  5]  14.00-15.00  sec  2.56 MBytes  21.5 Mbits/sec  0.076 ms  48579/50604 (96%)  
[  5]  15.00-16.00  sec  2.56 MBytes  21.5 Mbits/sec  0.078 ms  48572/50596 (96%)  
[  5]  16.00-17.00  sec  2.56 MBytes  21.5 Mbits/sec  0.059 ms  49664/51689 (96%)  
[  5]  17.00-18.00  sec  2.56 MBytes  21.5 Mbits/sec  0.066 ms  50305/52329 (96%)  
[  5]  18.00-19.00  sec  2.56 MBytes  21.5 Mbits/sec  0.086 ms  50655/52680 (96%)  
[  5]  19.00-20.00  sec  2.56 MBytes  21.5 Mbits/sec  0.068 ms  50704/52728 (96%)  
[  5]  20.00-21.00  sec  2.56 MBytes  21.5 Mbits/sec  0.061 ms  50740/52764 (96%)  
[  5]  21.00-22.00  sec  2.56 MBytes  21.5 Mbits/sec  0.064 ms  50714/52739 (96%)  
[  5]  22.00-23.00  sec  2.56 MBytes  21.5 Mbits/sec  0.070 ms  50817/52841 (96%)  
[  5]  23.00-24.00  sec  2.56 MBytes  21.5 Mbits/sec  0.061 ms  50724/52749 (96%)  
[  5]  24.00-25.00  sec  2.56 MBytes  21.5 Mbits/sec  0.051 ms  50550/52574 (96%)  
[  5]  25.00-26.00  sec  2.56 MBytes  21.5 Mbits/sec  0.043 ms  50564/52588 (96%)  
[  5]  26.00-27.00  sec  2.56 MBytes  21.5 Mbits/sec  0.042 ms  50580/52605 (96%)  
[  5]  27.00-28.00  sec  2.56 MBytes  21.5 Mbits/sec  0.058 ms  50732/52756 (96%)  
[  5]  28.00-29.00  sec  2.56 MBytes  21.5 Mbits/sec  0.201 ms  50796/52821 (96%)  
[  5]  29.00-30.00  sec  2.56 MBytes  21.5 Mbits/sec  0.065 ms  49016/51040 (96%)  
[  5]  30.00-30.61  sec  1008 KBytes  13.5 Mbits/sec  0.051 ms  18951/19728 (96%)  
- - - - - - - - - - - - - - - - - - - - - - - - -
Test Complete. Summary Results:
[ ID] Interval           Transfer     Bitrate         Jitter    Lost/Total Datagrams
[  5] (sender statistics not available)
[  5]   0.00-30.61  sec  81.5 MBytes  22.3 Mbits/sec  0.051 ms  1488907/1553252 (96%)  receiver
CPU Utilization: local/receiver 1.6% (0.5%u/1.1%s), remote/sender 0.0% (0.0%u/0.0%s)


iperf Done.
```
