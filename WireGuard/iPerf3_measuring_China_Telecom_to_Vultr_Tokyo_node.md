## 用 iPerf3 测量 WireGuard VPN 的网络吞吐量

#### 客户端为中国电信，服务器是 Vultr 东京机房，测量时间为 2018 年 6 月 4 日下午

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
[  5] local 10.100.0.10 port 44684 connected to 10.100.0.1 port 5201
[ ID] Interval           Transfer     Bitrate         Jitter    Lost/Total Datagrams
[  5]   0.00-10.00  sec   108 MBytes  90.4 Mbits/sec  0.061 ms  585019/667621 (88%)  
[  5]  10.00-20.00  sec   108 MBytes  90.3 Mbits/sec  0.054 ms  621063/703599 (88%)  
[  5]  20.00-30.00  sec   108 MBytes  90.4 Mbits/sec  0.051 ms  590182/672774 (88%)  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Jitter    Lost/Total Datagrams
[  5]   0.00-30.41  sec  2.64 GBytes   745 Mbits/sec  0.000 ms  0/2070730 (0%)  sender
[  5]   0.00-30.00  sec   323 MBytes  90.4 Mbits/sec  0.051 ms  1796264/2043994 (88%)  receiver

Server output:
iperf 3.5+
Linux cloud 4.15.0-20-generic #21-Ubuntu SMP Tue Apr 24 06:16:15 UTC 2018 x86_64
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------
Time: Mon, 04 Jun 2018 08:35:36 GMT
Accepted connection from 10.100.0.10, port 38472
      Cookie: vxjcts5s4cmk3c73lkhjrnweudbddai6d3z5
[  5] local 10.100.0.1 port 5201 connected to 10.100.0.10 port 44684
Starting Test: protocol: UDP, 1 streams, 1368 byte blocks, omitting 0 seconds, 30 second test, tos 0
[ ID] Interval           Transfer     Bitrate         Total Datagrams
[  5]   0.00-1.00   sec  48.9 MBytes   410 Mbits/sec  37484  
[  5]   1.00-2.00   sec  83.2 MBytes   698 Mbits/sec  63771  
[  5]   2.00-3.00   sec  91.7 MBytes   769 Mbits/sec  70304  
[  5]   3.00-4.00   sec  92.2 MBytes   773 Mbits/sec  70657  
[  5]   4.00-5.00   sec  95.1 MBytes   798 Mbits/sec  72888  
[  5]   5.00-6.00   sec  85.7 MBytes   719 Mbits/sec  65669  
[  5]   6.00-7.00   sec  95.5 MBytes   801 Mbits/sec  73206  
[  5]   7.00-8.00   sec  95.3 MBytes   799 Mbits/sec  73048  
[  5]   8.00-9.00   sec  94.2 MBytes   790 Mbits/sec  72205  
[  5]   9.00-10.00  sec  96.1 MBytes   806 Mbits/sec  73670  
[  5]  10.00-11.00  sec  95.5 MBytes   801 Mbits/sec  73193  
[  5]  11.00-12.00  sec  94.8 MBytes   795 Mbits/sec  72681  
[  5]  12.00-13.00  sec  96.6 MBytes   810 Mbits/sec  74053  
[  5]  13.00-14.00  sec  94.7 MBytes   795 Mbits/sec  72604  
[  5]  14.00-15.00  sec  86.2 MBytes   723 Mbits/sec  66072  
[  5]  15.00-16.00  sec  91.1 MBytes   764 Mbits/sec  69809  
[  5]  16.00-17.00  sec  95.4 MBytes   800 Mbits/sec  73095  
[  5]  17.00-18.00  sec  96.6 MBytes   811 Mbits/sec  74082  
[  5]  18.00-19.00  sec  72.6 MBytes   609 Mbits/sec  55645  
[  5]  19.00-20.00  sec  87.9 MBytes   738 Mbits/sec  67399  
[  5]  20.00-21.00  sec  93.7 MBytes   786 Mbits/sec  71805  
[  5]  21.00-22.00  sec  96.2 MBytes   807 Mbits/sec  73762  
[  5]  22.00-23.00  sec  95.9 MBytes   804 Mbits/sec  73472  
[  5]  23.00-24.00  sec  76.2 MBytes   639 Mbits/sec  58430  
[  5]  24.00-25.00  sec  69.4 MBytes   582 Mbits/sec  53216  
[  5]  25.00-26.00  sec  68.6 MBytes   575 Mbits/sec  52550  
[  5]  26.00-27.00  sec  91.1 MBytes   764 Mbits/sec  69829  
[  5]  27.00-28.00  sec  95.2 MBytes   796 Mbits/sec  73005  
[  5]  28.00-29.00  sec  95.5 MBytes   804 Mbits/sec  73180  
[  5]  29.00-30.00  sec  94.5 MBytes   793 Mbits/sec  72459  
[  5]  30.00-30.41  sec  35.9 MBytes   733 Mbits/sec  27487  
- - - - - - - - - - - - - - - - - - - - - - - - -
Test Complete. Summary Results:
[ ID] Interval           Transfer     Bitrate         Jitter    Lost/Total Datagrams
[  5]   0.00-30.41  sec  2.64 GBytes   745 Mbits/sec  0.000 ms  0/2070730 (0%)  sender
[  5] (receiver statistics not available)
CPU Utilization: local/sender 5.4% (0.4%u/5.0%s), remote/receiver 74.4% (10.4%u/64.0%s)

iperf Done.
```
