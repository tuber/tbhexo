---
title: tcpdump 工具查看分析arp协议
date: 2019-05-13 22:26:44
categories: NETWORK
tags:
    - ARP
    - tcpdump
---
##### 环境准备机器1 udev的mac及ip地址
```
root@udev:/home/tb# ifconfig
enp0s3    Link encap:Ethernet  HWaddr 08:00:27:63:49:66  
          inet addr:10.70.30.73  Bcast:10.70.31.255  Mask:255.255.254.0
          inet6 addr: fe80::a00:27ff:fe63:4966/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1992020 errors:0 dropped:0 overruns:0 frame:0
          TX packets:569243 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:235878919 (235.8 MB)  TX bytes:149889975 (149.8 MB)
```

 <!-- more -->
##### 环境准备机器2 php56当前的mac及ip地址及 arp缓存

```
tb@php56:~$ ifconfig
docker0   Link encap:Ethernet  HWaddr 02:42:c6:68:73:96  
          inet addr:172.17.0.1  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

enp0s3    Link encap:Ethernet  HWaddr 08:00:27:ce:14:39  
          inet addr:10.70.30.60  Bcast:10.70.31.255  Mask:255.255.254.0
          inet6 addr: fe80::a00:27ff:fece:1439/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1636533 errors:0 dropped:0 overruns:0 frame:0
          TX packets:149265 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:219865638 (219.8 MB)  TX bytes:123084741 (123.0 MB)
root@php56:/home/tb# arp -a
? (10.70.30.79) at 08:62:66:4d:f1:09 [ether] on enp0s3
? (10.70.30.32) at 64:00:6a:20:ae:c6 [ether] on enp0s3
? (10.70.30.47) at 8c:ec:4b:5f:e9:49 [ether] on enp0s3
? (10.70.30.73) at 08:00:27:63:49:66 [ether] on enp0s3
? (10.70.30.1) at 84:b2:61:8f:98:00 [ether] on enp0s3
? (10.70.30.72) at 8c:ec:4b:a1:49:3f [ether] on enp0s3
? (10.70.30.40) at 74:ea:c8:e3:17:ab [ether] on enp0s3
? (10.70.31.191) at <incomplete> on enp0s3
```

##### 删除php56上的10.70.30.73的arp缓存
```
root@php56:/home/tb# arp -d 10.70.30.73

#抓php56(10.70.30.66 )上 telnet 到10.70.30.73的包
root@php56:/home/tb# tcpdump -i enp0s3 -ent '(dst 10.70.30.73 and src 10.70.30.60) or (dst 10.70.30.60 and src 10.70.30.73)'
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s3, link-type EN10MB (Ethernet), capture size 262144 bytes

# -e选项代表开启以太网帧头部信息显示
```
##### 新开一个窗口在php56上  telnet，失败不要紧，因为在链路层，arp在tcp连接建立前就已经完成，不关心成功与否
```
root@php56:/home/tb# telnet 10.70.30.73
Trying 10.70.30.73...
telnet: Unable to connect to remote host: Connection refused
```
##### 抓包结果
```
root@php56:/home/tb# tcpdump -i enp0s3 -ent '(dst 10.70.30.73 and src 10.70.30.60) or (dst 10.70.30.60 and src 10.70.30.73)'
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s3, link-type EN10MB (Ethernet), capture size 262144 bytes
08:00:27:ce:14:39 > ff:ff:ff:ff:ff:ff, ethertype ARP (0x0806), length 42: Request who-has 10.70.30.73 tell 10.70.30.60, length 28
08:00:27:63:49:66 > 08:00:27:ce:14:39, ethertype ARP (0x0806), length 60: Reply 10.70.30.73 is-at 08:00:27:63:49:66, length 46
08:00:27:ce:14:39 > 08:00:27:63:49:66, ethertype IPv4 (0x0800), length 74: 10.70.30.60.42366 > 10.70.30.73.23: Flags [S], seq 803077829, win 29200, options [mss 1460,sackOK,TS val 173958745 ecr 0,nop,wscale 7], length 0
08:00:27:63:49:66 > 08:00:27:ce:14:39, ethertype IPv4 (0x0800), length 60: 10.70.30.73.23 > 10.70.30.60.42366: Flags [R.], seq 0, ack 803077830, win 0, length 0
08:00:27:63:49:66 > 08:00:27:ce:14:39, ethertype ARP (0x0806), length 60: Request who-has 10.70.30.60 tell 10.70.30.73, length 46
08:00:27:ce:14:39 > 08:00:27:63:49:66, ethertype ARP (0x0806), length 42: Reply 10.70.30.60 is-at 08:00:27:ce:14:39, length 28
08:00:27:ce:14:39 > 08:00:27:63:49:66, ethertype IPv4 (0x0800), length 74: 10.70.30.60.42368 > 10.70.30.73.23: Flags [S], seq 3070062063, win 29200, options [mss 1460,sackOK,TS val 173961995 ecr 0,nop,wscale 7], length 0
08:00:27:63:49:66 > 08:00:27:ce:14:39, ethertype IPv4 (0x0800), length 60: 10.70.30.73.23 > 10.70.30.60.42368: Flags [R.], seq 0, ack 3070062064, win 0, length 0
08:00:27:ce:14:39 > 08:00:27:63:49:66, ethertype IPv4 (0x0800), length 74: 10.70.30.60.52718 > 10.70.30.73.7: Flags [S], seq 4237197441, win 29200, options [mss 1460,sackOK,TS val 173965580 ecr 0,nop,wscale 7], length 0
08:00:27:63:49:66 > 08:00:27:ce:14:39, ethertype IPv4 (0x0800), length 60: 10.70.30.73.7 > 10.70.30.60.52718: Flags [R.], seq 0, ack 4237197442, win 0, length 0
08:00:27:ce:14:39 > 08:00:27:63:49:66, ethertype IPv4 (0x0800), length 74: 10.70.30.60.52720 > 10.70.30.73.7: Flags [S], seq 3993979182, win 29200, options [mss 1460,sackOK,TS val 173969570 ecr 0,nop,wscale 7], length 0
08:00:27:63:49:66 > 08:00:27:ce:14:39, ethertype IPv4 (0x0800), length 60: 10.70.30.73.7 > 10.70.30.60.52720: Flags [R.], seq 0, ack 3993979183, win 0, length 0
08:00:27:63:49:66 > 08:00:27:ce:14:39, ethertype ARP (0x0806), length 60: Request who-has 10.70.30.60 tell 10.70.30.73, length 46
08:00:27:ce:14:39 > 08:00:27:63:49:66, ethertype ARP (0x0806), length 42: Reply 10.70.30.60 is-at 08:00:27:ce:14:39, length 28

```
##### 包内容简短解释
ff:ff:ff:ff:ff:ff 代表lan内广播地址，所有机器都会收到并处理这样的帧。Ox086代表是以太网帧arp类型（注意分用思想）。length 42字节，实际为46，由于tcpdump不关心以太网帧尾部的crc校验字段。最后的length 28|46 字节代表数据长度。request reply为arp请求 应答 固定标识，最后路由器并不响应arp请求。

##### 参考自下图

![image.png](https://image-static.segmentfault.com/169/921/1699212970-5dbb9b59ac52c_articlex)


