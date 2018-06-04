## 用 iPerf3 测量 WireGuard VPN 的网络吞吐量

#### 客户端为中国电信，服务器是 Google Cloud Platform (GCE) 台湾数据中心，测量时间为 2018 年 6 月 4 日下午

服务器端参数

```
iperf3 -s -V
```

客户端参数

```
iperf3 -c 10.100.0.1 -u -i 10 -t 30 -b 10G -R --get-server-output
```

测量输入结果如下，已包含客户端和服务器端的结果：

```
iperf3 -c 10.100.0.1 -u -i 10 -t 30 -b 10G -R --get-server-output
Connecting to host 10.100.0.1, port 5201
Reverse mode, remote host 10.100.0.1 is sending
[  5] local 10.100.0.10 port 51880 connected to 10.100.0.1 port 5201
[ ID] Interval           Transfer     Bitrate         Jitter    Lost/Total Datagrams
[  5]   0.00-10.00  sec   108 MBytes  90.7 Mbits/sec  0.043 ms  790044/875377 (90%)  
[  5]  10.00-20.00  sec   108 MBytes  90.4 Mbits/sec  0.032 ms  778739/863846 (90%)  
[  5]  20.00-30.00  sec   108 MBytes  90.8 Mbits/sec  0.035 ms  781755/867180 (90%)  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Jitter    Lost/Total Datagrams
[  5]   0.00-30.20  sec  3.24 GBytes   923 Mbits/sec  0.000 ms  0/2622734 (0%)  sender
[SUM]  0.0-30.2 sec  2 datagrams received out-of-order
[  5]   0.00-30.00  sec   324 MBytes  90.6 Mbits/sec  0.035 ms  2350538/2606403 (90%)  receiver

Server output:
Time: Mon, 04 Jun 2018 09:12:40 GMT
Accepted connection from 10.100.0.10, port 38476
      Cookie: uorfa4ap2nwcp42othpkp6xd2ekfsljumc2g
[  5] local 10.100.0.1 port 5201 connected to 10.100.0.10 port 51880
Starting Test: protocol: UDP, 1 streams, 1328 byte blocks, omitting 0 seconds, 30 second test, tos 0
[ ID] Interval           Transfer     Bitrate         Total Datagrams
[  5]   0.00-1.00   sec   117 MBytes   985 Mbits/sec  92704  
[  5]   1.00-2.00   sec   109 MBytes   916 Mbits/sec  86243  
[  5]   2.00-3.00   sec   110 MBytes   919 Mbits/sec  86521  
[  5]   3.00-4.00   sec   110 MBytes   924 Mbits/sec  86945  
[  5]   4.00-5.00   sec   109 MBytes   916 Mbits/sec  86205  
[  5]   5.00-6.00   sec   110 MBytes   919 Mbits/sec  86547  
[  5]   6.00-7.00   sec   111 MBytes   929 Mbits/sec  87480  
[  5]   7.00-8.00   sec   111 MBytes   928 Mbits/sec  87363  
[  5]   8.00-9.00   sec   110 MBytes   923 Mbits/sec  86882  
[  5]   9.00-10.00  sec   109 MBytes   913 Mbits/sec  85940  
[  5]  10.00-11.00  sec   107 MBytes   898 Mbits/sec  84543  
[  5]  11.00-12.00  sec   111 MBytes   932 Mbits/sec  87724  
[  5]  12.00-13.00  sec   110 MBytes   923 Mbits/sec  86864  
[  5]  13.00-14.00  sec   111 MBytes   928 Mbits/sec  87390  
[  5]  14.00-15.00  sec   110 MBytes   925 Mbits/sec  87101  
[  5]  15.00-16.00  sec   113 MBytes   945 Mbits/sec  88993  
[  5]  16.00-17.00  sec   111 MBytes   929 Mbits/sec  87426  
[  5]  17.00-18.00  sec   116 MBytes   974 Mbits/sec  91656  
[  5]  18.00-19.00  sec   103 MBytes   868 Mbits/sec  81678  
[  5]  19.00-20.00  sec   103 MBytes   861 Mbits/sec  80998  
[  5]  20.00-21.00  sec   109 MBytes   917 Mbits/sec  86334  
[  5]  21.00-22.00  sec   110 MBytes   923 Mbits/sec  86839  
[  5]  22.00-23.00  sec   110 MBytes   923 Mbits/sec  86920  
[  5]  23.00-24.00  sec   109 MBytes   914 Mbits/sec  86076  
[  5]  24.00-25.00  sec   110 MBytes   919 Mbits/sec  86471  
[  5]  25.00-26.00  sec   110 MBytes   921 Mbits/sec  86728  
[  5]  26.00-27.00  sec   110 MBytes   927 Mbits/sec  87237  
[  5]  27.00-28.00  sec   111 MBytes   927 Mbits/sec  87299  
[  5]  28.00-29.00  sec   109 MBytes   918 Mbits/sec  86443  
[  5]  29.00-30.00  sec   111 MBytes   931 Mbits/sec  87643  
[  5]  30.00-30.20  sec  22.2 MBytes   922 Mbits/sec  17541  
- - - - - - - - - - - - - - - - - - - - - - - - -
Test Complete. Summary Results:
[ ID] Interval           Transfer     Bitrate         Jitter    Lost/Total Datagrams
[  5]   0.00-30.20  sec  3.24 GBytes   923 Mbits/sec  0.000 ms  0/2622734 (0%)  sender
[  5] (receiver statistics not available)
CPU Utilization: local/sender 33.9% (3.7%u/30.2%s), remote/receiver 0.0% (0.0%u/0.0%s)


iperf Done.
```
