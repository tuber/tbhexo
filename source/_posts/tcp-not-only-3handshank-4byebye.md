---
title: TCP除了3次握手四次挥手之外的基础知识
date: 2019-11-04 22:18:59
categories: NETWORK
tags:
 - TCP
---
#### tcp特点
1. 使用tcp双方的链接分配必要的内核资源，以管理连接的状态和连接上的数据的传输。全双工，即双方的数据读写可以通过一个连接进行。完成数据交换后，需要断开连接以释放系统资源。因为是一对一，所以基于广播和多播的协议不能使用tcp程序。tcp模块发送数据时，涉及到发送缓冲区可能被封装成一个或者多个tcp报文段发出。由此可见，tcp模块发送出的tcp报文段的个数和应用程序执行写的操作次数之间没有固定的数量的关系。
2.  当接收端收到一个或者多个tcp报文段后，tcp模块将他们携带的应用的程序数据按照tcp报文段的序号依次放入到tcp接收缓冲区中。并通知应用程序去读取数据，接收端应用程序可以一次性将tcp接受缓冲区的数据全部取出，也可以分多次读取。这个取决于用户指定的应用程序缓冲区的大小。由此可见，应用程序执行的读操作次数和tcp模块接收到tcp报文段个数之间也没有固定的数量关系。
3. 字节流的概念：发送端执行的写操作次数和接收端执行的读操作次数之间没有任何的数量关系，这就是字节流的概念。应用程序对数据的发送和接收是没有唯一边界限制的。而udp发送端应用程序每执行一次写，udp模块就将其封装成一个udp数据报并发送，接收端必须及时接收每个udp数据报进行读操作（recvfrom），否则就会丢包。此外，如果用户没有指定足够的应用程序缓冲区来读取udp数据，则udp数据将被截断。
4.  一些机制
    + 定时器
    + 未收到回应后重发（定时器）
    + RWND:receiver window：接收通告窗口
    + tcp对ip数据报进行重排，整理，再交付给上层


#### tcp头部结构
<!-- more -->

- 16位源端口|目的端口号，一般客户端选择临时，而服务端采取固定/etc/services
- 32位序列号 sequence number：一次tcp通信（从tcp连接到断开过程中某个一个传输方向上的某一个传输方向上的字节流的每个字节的每个子节点编号）

####  tcp四次关闭流程（假设客户端首先发起关闭）
- 客户端主动关闭，发送fin报文，客户端进入FIN_WAIT1
- 服务端收到客户端关闭连接后，会发送一个ack状态。进入CLOSE_WAIT状态,而此时客户端会进入FIN_WAIT2状态（此时wait2等待的是服务端发送的fin）。CLOSE_WAIT状态意思是等待服务器应用程序关闭连接，如果没有服务器没有阻塞，也会给客户端发送一个fin结束报文，然后服务端会进入到LAST_ACK状态，等待客户端对刚刚服务器发送的fin的最后的确认。如果服务器一直阻塞在某个程序（比如io阻塞），那么就一直会在close_wait状态。客户端给予最后的服务端的fin之后，会进入TIME_WAIT状态。
- 如果客户端发送最后一个ack给服务器。那么服务端就会进入close。
- 四次挥手是因为是全双工，A一方发送fin后，另一B方只能暂且发送ack，因为此方B不明确上层协议是否还有数据要发送给A。A只是说我不发了，B你有数据发给A，A还是可以接受的。

#### tcp三次握手过程
- 服务端listen，进入被动调用状态
- 服务端收到客户端connect调用发送的syn同步报文，服务端将该连接放入到内核等待队列中，此时客户端处于SYC_SENT状态
- 服务端向客户端发送带syn+ack标志的确认报文段，ack的值为客户端syn的值+1，此时服务端处于SYN_RCVD状态
- 客户端发送ack，服务端正确接收。双方为ESTABLISHED状态，
- 三次握手的本质是确认客户端的收发能力和服务端的收发能力完全正常。

#### 半关闭状态指的是
- 客户端处于FIN_WAIT2
- 服务端处于CLOSE_WAIT
连接处在fin_wait2的状态情况可能发生在，客户端执行半关闭后，未等服务器关闭连接后就强行退出。此时客户端连接会由内核来接管。可称为孤儿连接。linux为了防止孤儿连接长时间再内核中，定义如下两个变量控制max数量及孤儿连接时间
    * root@udev:/home/tb/tbtmp# cat /proc/sys/net/ipv4/tcp_max_orphans  # orphans 是孤儿的意思
        4096
    * root@udev:/home/tb/tbtmp# cat /proc/sys/net/ipv4/tcp_fin_timeout 
        60
* 如何模拟半连接？
    半连接状态下，如果对端发送数据，对方将回应一个复位报文段。客户端可以通过拔掉网线，服务端可以通过停止服务。

#### TIME_WAIT状态存在的原因有以下两点
- 可靠的终止tcp连接
     为了保证客户端最后发送的ack如果中途丢失了，那么服务端会再次发送fin+ack，客户端需要在当前状态下再次回复当前连接的最后一个ack。否则会导致1. 客户端发送rst回应服务器，2.服务器莫名其妙（你不该给我ack吗）
- 保证让迟来的tcp报文段有足够的时间被识别并丢弃
    tcp端口在处于time_wait状态不能同时打开两次（可以用nc工具测试，断开并且马上用重复端口连接服务器）。如果断开连接后，客户端没有这个状态，则这个新连接可能收到和之前连接相同的tcp报文段（迟到的报文段）
- tcp 2msl的概念
    一个新的tcp连接应该在2msl之后安全建立。但是客户端本身由于临时端口的随机性，理论上可以关闭而无需time_wait（SO_REUSEADDR选项可以控制）状态。而考虑到服务端的固定端口就不大能接受了。

#### 复位报文段
- 访问不存在的端口,可以看到服务端复位报文段窗口大小win=0，seq=0，length=0.说明客户端也无法回复这个复位报文段。
- 
    ```
        root@php56:/home/tb# tcpdump -nt -i enp0s3 port 33765
        tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
        listening on enp0s3, link-type EN10MB (Ethernet), capture size 262144 bytes
        IP 10.70.30.73.43936 > 10.70.30.60.33765: Flags [S], seq 577022539, win 29200, options [mss 1460,sackOK,TS val 22768358 ecr 0,nop,wscale 7], length 0
        IP 10.70.30.60.33765 > 10.70.30.73.43936: Flags [R.], seq 0, ack 577022540, win 0, length 0
    ```
- 异常终止连接（socket选项SO_LINGER来发送）
    - 一旦发送了一个复位报文段，发送端所有排队等待的数据都会被丢弃
    - 这种不会进入到time_wait阶段，所以随之而来的问题可想而知（根据time_wait状态存在的意义）
    

#### tcp的交互数据和成块数据
- 交互数据可以理解为命令行下的交互 比如telnet，ssh。而成块数据涉及ftp等。以下为本机抓包23端口（登陆后输入ls的抓包细节）
    
-    
    ```
    tb@php56:~$ ls
    ...
    1.IP 127.0.0.1.50188 > 127.0.0.1.23: Flags [P.], seq 93:94, ack 503, win 350, options [nop,nop,TS val 21041646 ecr 21040271], length 1
    2.IP 127.0.0.1.23 > 127.0.0.1.50188: Flags [P.], seq 503:504, ack 94, win 342, options [nop,nop,TS val 21041646 ecr 21041646], length 1
    3.IP 127.0.0.1.50188 > 127.0.0.1.23: Flags [.], ack 504, win 350, options [nop,nop,TS val 21041646 ecr 21041646], length 0
    4.IP 127.0.0.1.50188 > 127.0.0.1.23: Flags [P.], seq 94:95, ack 504, win 350, options [nop,nop,TS val 21042291 ecr 21041646], length 1
    5.IP 127.0.0.1.23 > 127.0.0.1.50188: Flags [P.], seq 504:505, ack 95, win 342, options [nop,nop,TS val 21042291 ecr 21042291], length 1
    6.IP 127.0.0.1.50188 > 127.0.0.1.23: Flags [.], ack 505, win 350, options [nop,nop,TS val 21042291 ecr 21042291], length 0
    ...
    # ls回车敲下之后
    tb@php56:~$ ls
    anyconnect-linux64-4.6.03049                      data_mei     draveness.me         mount_all  tmptest           zikao_coding
    anyconnect-linux64-4.6.03049-predeploy-k9.tar.gz  dev_xin_tmp  login_lixinghang.sh  tb_down    vboxguestadition
    
    7.IP 127.0.0.1.50188 > 127.0.0.1.23: Flags [P.], seq 95:97, ack 505, win 350, options [nop,nop,TS val 21062968 ecr 21042291], length 2
    8.IP 127.0.0.1.23 > 127.0.0.1.50188: Flags [P.], seq 505:911, ack 97, win 342, options [nop,nop,TS val 21062968 ecr 21062968], length 406
    9.IP 127.0.0.1.50188 > 127.0.0.1.23: Flags [.], ack 911, win 359, options [nop,nop,TS val 21062968 ecr 21062968], length 0
    ```
- 其中123分半是客户端发送l，服务端对l确认，客户端对服务端l的确认，456同理为s
- 下面的7可以看到length为两个字节，应该为回车符和流结束符eof，为0x00
- 第8个length为406个字节，为具体的ls数据输出，包括文件名及其显示控制参数，第9个是客户端对第8个报文段的确认。
- 针对以上例子简单总结如下：
    - ack表示的是确认号，表示期待接收的下个序列号。
    - tcp全双工，一个连接上一个方向的tcp报文段都包括了相反方向上的报文段的ack。
    - 客户端对服务端的确认分组，不带任何应用数据 即length=0
    - 而服务端对客户端的确认数据，比如2,5,8既包含ack，又包括应用程序数据length。服务端的这种方式称为延迟确认，即不马上确认收到的客户端的ack，而是等等看是否本端有数据要给客户端，有的话一起在一个报文段里一并发出。即ack是累积的，一个确认字节号N的ack表示所有直到N的字节（不包括N）已经成功被接收。这样可以减少tcp报文段的数量。还有另外一个好处如果一个ack只接受一个，那么其中一个丢失了，那么后面的都得重传。而延迟确认可能将之前可能的ack的字节都一次都确认了。
    - 引申出nagle算法，主要目的就是减少大量小包的发送。Nagle算法的规则（可参考tcp_output.c文件里tcp_nagle_check函数注释），可以理解为他是针对每个包的最大报文段长度（mss）的停等协议。即只有一个未被ACK的包的包的包的包存在于网络。
        - 如果包长度达到MSS，则允许发送；
        - 如果该包含有FIN，则允许发送；
        - 设置了TCP_NODELAY选项，则允许发送；
        - 未设置TCP_CORK选项时，若所有发出去的小数据包（包长度小于MSS）均被确认，则允许发送；
        - 上述条件都未满足，但发生了超时（一般为200ms），则立即发送。

#### 成块数据vftp模拟

```
## 客户端交互数据
tb@php56:~$ ftp 127.0.0.1
Connected to 127.0.0.1.
220 (vsFTPd 3.0.3)
Name (127.0.0.1:tb): tb
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 




## 服务监听端抓包明细
apt-get install vsftpd
root@php56:/home/tb# tcpdump -nt -i lo port 21
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on lo, link-type EN10MB (Ethernet), capture size 262144 bytes
IP 127.0.0.1.34426 > 127.0.0.1.21: Flags [S], seq 3720077491, win 43690, options [mss 65495,sackOK,TS val 21572543 ecr 0,nop,wscale 7], length 0
IP 127.0.0.1.21 > 127.0.0.1.34426: Flags [S.], seq 3214185420, ack 3720077492, win 43690, options [mss 65495,sackOK,TS val 21572543 ecr 21572543,nop,wscale 7], length 0
IP 127.0.0.1.34426 > 127.0.0.1.21: Flags [.], ack 1, win 342, options [nop,nop,TS val 21572543 ecr 21572543], length 0
IP 127.0.0.1.21 > 127.0.0.1.34426: Flags [P.], seq 1:21, ack 1, win 342, options [nop,nop,TS val 21572543 ecr 21572543], length 20: FTP: 220 (vsFTPd 3.0.3)
IP 127.0.0.1.34426 > 127.0.0.1.21: Flags [.], ack 21, win 342, options [nop,nop,TS val 21572543 ecr 21572543], length 0
IP 127.0.0.1.34426 > 127.0.0.1.21: Flags [P.], seq 1:10, ack 21, win 342, options [nop,nop,TS val 21573111 ecr 21572543], length 9: FTP: USER tb
IP 127.0.0.1.21 > 127.0.0.1.34426: Flags [.], ack 10, win 342, options [nop,nop,TS val 21573111 ecr 21573111], length 0
IP 127.0.0.1.21 > 127.0.0.1.34426: Flags [P.], seq 21:55, ack 10, win 342, options [nop,nop,TS val 21573111 ecr 21573111], length 34: FTP: 331 Please specify the password.
IP 127.0.0.1.34426 > 127.0.0.1.21: Flags [.], ack 55, win 342, options [nop,nop,TS val 21573111 ecr 21573111], length 0
IP 127.0.0.1.34426 > 127.0.0.1.21: Flags [P.], seq 10:19, ack 55, win 342, options [nop,nop,TS val 21573327 ecr 21573111], length 9: FTP: PASS tb
IP 127.0.0.1.21 > 127.0.0.1.34426: Flags [P.], seq 55:78, ack 19, win 342, options [nop,nop,TS val 21573334 ecr 21573327], length 23: FTP: 230 Login successful.
IP 127.0.0.1.34426 > 127.0.0.1.21: Flags [.], ack 78, win 342, options [nop,nop,TS val 21573334 ecr 21573334], length 0
IP 127.0.0.1.34426 > 127.0.0.1.21: Flags [P.], seq 19:25, ack 78, win 342, options [nop,nop,TS val 21573334 ecr 21573334], length 6: FTP: SYST
IP 127.0.0.1.21 > 127.0.0.1.34426: Flags [P.], seq 78:97, ack 25, win 342, options [nop,nop,TS val 21573334 ecr 21573334], length 19: FTP: 215 UNIX Type: L8
IP 127.0.0.1.34426 > 127.0.0.1.21: Flags [.], ack 97, win 342, options [nop,nop,TS val 21573344 ecr 21573334], length 0

```
#### 带外数据out of band
- 用于迅速通告对方本端的重要事件，优先级更高
- 涉及到紧急指针标志和紧急指针所指向的位置
- 带外缓存只有一个字节
- SS_OOBINLINE选项

####  tcp超时重传
 
```
# 服务端开启服务
apt install iperf3
root@php56:/home/tb# iperf3 -s
-----------------------------------------------------------
Server listening on 5201
-----------------------------------------------------------

## 在客户端执行参数
root@udev:/home/tb# telnet 10.70.30.60 5201
Trying 10.70.30.60...
Connected to 10.70.30.60.
Escape character is '^]'.
1234
12

## 客户端抓包结果
root@udev:/home/tb# tcpdump -n -i enp0s3 port 5201
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on enp0s3, link-type EN10MB (Ethernet), capture size 262144 bytes
16:06:59.057034 IP 10.70.30.73.55102 > 10.70.30.60.5201: Flags [S], seq 654898028, win 29200, options [mss 1460,sackOK,TS val 26731779 ecr 0,nop,wscale 7], length 0
16:06:59.057257 IP 10.70.30.60.5201 > 10.70.30.73.55102: Flags [S.], seq 2973035393, ack 654898029, win 28960, options [mss 1460,sackOK,TS val 22374774 ecr 26731779,nop,wscale 7], length 0
16:06:59.057270 IP 10.70.30.73.55102 > 10.70.30.60.5201: Flags [.], ack 1, win 229, options [nop,nop,TS val 26731779 ecr 22374774], length 0

16:07:09.416168 IP 10.70.30.73.55102 > 10.70.30.60.5201: Flags [P.], seq 1:7, ack 1, win 229, options [nop,nop,TS val 26734369 ecr 22374774], length 6
16:07:09.416390 IP 10.70.30.60.5201 > 10.70.30.73.55102: Flags [.], ack 7, win 227, options [nop,nop,TS val 22377364 ecr 26734369], length 0


16:07:28.984171 IP 10.70.30.73.55102 > 10.70.30.60.5201: Flags [P.], seq 7:11, ack 1, win 229, options [nop,nop,TS val 26739261 ecr 22377364], length 4
16:07:29.188603 IP 10.70.30.73.55102 > 10.70.30.60.5201: Flags [P.], seq 7:11, ack 1, win 229, options [nop,nop,TS val 26739312 ecr 22377364], length 4
16:07:29.392513 IP 10.70.30.73.55102 > 10.70.30.60.5201: Flags [P.], seq 7:11, ack 1, win 229, options [nop,nop,TS val 26739363 ecr 22377364], length 4
16:07:29.799504 IP 10.70.30.73.55102 > 10.70.30.60.5201: Flags [P.], seq 7:11, ack 1, win 229, options [nop,nop,TS val 26739465 ecr 22377364], length 4
16:07:30.616060 IP 10.70.30.73.55102 > 10.70.30.60.5201: Flags [P.], seq 7:11, ack 1, win 229, options [nop,nop,TS val 26739669 ecr 22377364], length 4
16:07:32.251692 IP 10.70.30.73.55102 > 10.70.30.60.5201: Flags [P.], seq 7:11, ack 1, win 229, options [nop,nop,TS val 26740078 ecr 22377364], length 4
16:07:35.524165 IP 10.70.30.73.55102 > 10.70.30.60.5201: Flags [P.], seq 7:11, ack 1, win 229, options [nop,nop,TS val 26740896 ecr 22377364], length 4

```
- 简单解释
    - 第一段为三次握手
    - 第二段为1234+回车+eof以及服务端的确认
    - 第三段为12+回车+eof的七次重传
- 相关内核参数
    - root@udev:/proc/sys/net/ipv4# cat tcp_retries1 3
    - root@udev:/proc/sys/net/ipv4# cat tcp_retries2 15

####  拥塞控制
- 四个部分
    - 慢启动 slow start
    - 拥塞避免 congestion avoidance
    - 快速重传 fast retransmit
    - 快速恢复 fast recovery
- 涉及算法
    - reno算法
    - vegas算法
    - cubic算法
    - root@udev:/proc/sys/net/ipv4# cat /proc/sys/net/ipv4/tcp_congestion_control  =>cubic
- 拥塞控制的最终受控变量是发送端向网络一次连续写入（收到其中第一个数据的确认之前）的数据量。称为send window SWND.请注意swnd是限制的tcp报文段的数量，而tcp报文段的最大长度（数据部分）和mss （maximum segment size）有关系。其中这个发送最大长度即称为SMSS.而接收方是通过rwnd（receive window）来告知控制发送端的swnd。值得注意的是，发送端引入了一个称为拥塞窗口 congestion window的状态的变量，实际的cwnd是 min（swnd，rwnd）
- 慢启动和拥塞避免
    - tcp建立连接时，cwnd的初始值（initial window），大小一般为2-4个smss，代表发送端最多能发送IW字节的数据。
    - 此后发送端每接收一个确认，cwnd按照以下公式增加：CWND+=MIN(N,SMSS).N代表此次确认中包含的之前的未被确认的字节数，这样CWND将按照指数形式扩大，这就是所谓的慢启动。这个思想是开始发送数据时，并不知道网络的实际情况，需要用一种试探的方式平滑的增加CWND的大小。
    - 慢启动门限 SLOW START THRESHOLD SIZE 缩写为ssthresh ，当cwnd的大小超过该值时，tcp拥塞控制将进入拥塞避免阶段。这样避免了cwnd一直按照线性方式增加，从而避免其扩大。主要通过以下两种方式
        - 每个rtt时间内按照CWND+=MIN(N,SMSS)计算新的cwnd，而不论该rtt时间内发送端接收到多少个确认。
        - 每收到一个对新数据的确认报文段，就按照CWND+=SMSS*SMSS/CWND
- 发送端判断拥塞发生的依据
    1. 传输超时，或者说tcp定时器溢出。应对策略为慢启动和拥塞避免
    2. 接收到重复的确认报文段。应对策略为快速重传和快速恢复。
- 快速重传和快速恢复
    - 发送端如果连续收到3个重复的确认报文段，就认为是拥塞发生了。然后将启用快速重传和快速恢复。按照以下格式：CWND=SSTHRESH+3*SMSS
    
####  关于ISN 初始序列号
也是绝对序列号，后期的序列号都是这个绝对序列号++。syn消耗序列号，而ack不会消耗。对于发送方和接收方都有自己的ISN的生成规则。ISN,包括对等端的+1操作，都可能涉及到重传操作。
    - ISN算法：ISN = M + F(localhost, localport, remotehost, remoteport)，m代表计时器，每隔4微秒+1；F是一个hash算法
    - 代码回绕问题 将无符号转为有符号。

####  关于SYN FOLLD攻击
- 基本思想：恶意像某个服务器端口发送大量SYN包，服务器会分配一个transmission control block，并返回ack。然后服务端转为syn-recv状态。系统保持这个资源
- 常见防攻击方法
    - 入门级:系统监视半连开连接和不活动的连接。一视同仁，超过一个阈值后全部rst这些连接。
    - 延缓tcp分配：当三次握手后再分配tcb，这样可以有效的减轻对服务器资源消耗（常见方法是使用syn cache，syn cookie）
        - syn cache
            - 保存对应的序列号和报文（这里特指半连接）到hash表中，直到收到正确的ack报文再分配tcp
        - syn cookie
            - 使用特殊算法生成sequence number，主要涉及到对方无法了解到的己方固定的一些信息（比如己方mss 时间等）。这样如果对方真的发送过来了ack报文，对其ack-1.如果相等，那么就分配这个tcb。没有或者不相等，那就不分配。
    - syn proxy防火墙：我简单理解为是一层代理，中间有验证，或者涉及到序列号的修改。具体的不了解。。

####  syn quene and accept quene
- 如果这两个队列满了，就会丢包。具体怎么个丢法。。不尽相同
- syns quene
    - 半连接队列，这时候服务端处理syn_rcvd 状态
    - linux 默认会进行指数退避算法，重发syn+ack。这也是给syn攻击者的一个机会。1+2+4+8+16+32=63s，tcp才会把这个连接断开。
    - tcp_syncookies:将连接信息编码在isn中返回给客户端。这时server不需要将半连接报错在队列中，而是利用客户端随后发来的ack带回的isn还原连接信息
    - tcp_max_syn_backlog
    - tcp_abort_on_overflow：0表示直接丢弃该ACK，1表示发送RST通知client，而客户端则分别返回read timeout 和connection reset by peer
    - tcp_syncookies
    - 以上这些参数遇到问题时可以适时调整参数
- accept quene
    - 全连接队列，这时候状态为established，但是未被应用程序accept

####  这张图对具体的发送时机和状态先后顺序非常清晰。值得保存]！！！![下面这个图对具体的发送时机和状态先后顺序非常清晰。值得保存][1]
[1]: /img/network/tcp-3-4.png
