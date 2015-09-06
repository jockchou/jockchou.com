<!--
author: jockchou
date: 2015-09-06
title: netstat命令的基本用法
tags: Linux
category: Linux服务器
status: publish
summary: Netstat是一款命令行工具，可用于列出系统上所有的网络套接字连接情况，包括tcp, udp以及unix套接字，另外它还能列出处于监听状态（即等待接入请求）的套接字。
-->


## 1.Netstat简介 ##

Netstat是一款命令行工具，可用于列出系统上所有的网络套接字连接情况，包括 tcp，udp以及unix套接字，另外它还能列出处于监听状态（即等待接入请求）的套接字。如果你想确认系统上的Web服务有没有起来，你可以查看80端口有没有打开。以上功能使netstat成为网管和系统管理员的必备利器。在这篇教程中，我会列出几个例子，教大家如何使用netstat去查找网络连接信息和系统开启的端口号。

以下的简单介绍来自netstat 的man手册：

> netstat： 打印网络连接、路由表、连接的数据统计、伪装连接以及广播域成员。  


## 2.列出所有连接 ##

第一个要介绍的是最简单的命令：列出所有当前的连接。使用 -a 选项即可。

```
[root@i-7jyb6ynm data]# netstat -a
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address               Foreign Address             State      
tcp        0      0 *:sphinxapi                 *:*                         LISTEN      
tcp        0      0 *:ssh                       *:*                         LISTEN      
tcp        0      0 *:sphinxql                  *:*                         LISTEN      
tcp        0      0 192.168.200.200:ssh         192.168.200.1:9749          ESTABLISHED 
tcp        0      0 *:mysql                     *:*                         LISTEN      
tcp        0      0 *:ssh                       *:*                         LISTEN      
udp        0      0 *:bootpc                    *:*                                     
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node Path
unix  2      [ ACC ]     STREAM     LISTENING     453354 /tmp/mysql.sock
unix  2      [ ACC ]     STREAM     LISTENING     8188   /var/run/dbus/system_bus_socket
unix  2      [ ACC ]     STREAM     LISTENING     8326   /var/run/acpid.socket
unix  2      [ ACC ]     STREAM     LISTENING     6746   @/com/ubuntu/upstart
unix  2      [ ]         DGRAM                    7247   @/org/kernel/udev/udevd
unix  8      [ ]         DGRAM                    8159   /dev/log
unix  2      [ ]         DGRAM                    1897164 
unix  2      [ ]         DGRAM                    8479   
unix  2      [ ]         DGRAM                    8421   
unix  3      [ ]         STREAM     CONNECTED     8413   /var/run/dbus/system_bus_socket
unix  3      [ ]         STREAM     CONNECTED     8412   
unix  2      [ ]         DGRAM                    8328   

...省略...   
```

上述命令列出tcp，udp和unix协议下所有套接字的所有连接。然而这些信息还不够详细，管理员往往需要查看某个协议或端口的具体连接情况。


## 3.只列出TCP或UDP协议的连接 ##

使用 -t 选项列出TCP协议的连接：

```
[root@i-7jyb6ynm data]# netstat -at
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address               Foreign Address             State      
tcp        0      0 *:sphinxapi                 *:*                         LISTEN      
tcp        0      0 *:ssh                       *:*                         LISTEN      
tcp        0      0 *:sphinxql                  *:*                         LISTEN      
tcp        0      0 192.168.200.200:ssh         192.168.200.1:9749          ESTABLISHED 
tcp        0      0 *:mysql                     *:*                         LISTEN      
tcp        0      0 *:ssh                       *:*                         LISTEN    
```
使用 -u 选项列出UDP协议的连接：

```
[root@i-7jyb6ynm data]# netstat -au
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address               Foreign Address             State      
udp        0      0 *:bootpc                    *:*                               
```

## 4.禁用反向域名解析 ##

默认情况下netstat会通过反向域名解析技术查找每个IP地址对应的主机名。这会降低查找速度，如果你觉得IP地址已经足够，而没有必要知道主机名，就使用 -n 选项禁用域名解析功能。

```
[root@i-7jyb6ynm data]# netstat -atn
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address               Foreign Address             State      
tcp        0      0 0.0.0.0:9312                0.0.0.0:*                   LISTEN      
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      
tcp        0      0 0.0.0.0:9306                0.0.0.0:*                   LISTEN      
tcp        0      0 192.168.200.200:22          192.168.200.1:9749          ESTABLISHED 
tcp        0      0 :::3306                     :::*                        LISTEN      
tcp        0      0 :::22                       :::*                        LISTEN      
```

## 5.只列出监听中的连接 ##

任何网络服务的后台进程都会打开一个端口，用于监听接入的请求。这些正在监听的套接字也和连接的套接字一样，也能被netstat列出来。使用 -l 选项列出正在监听的套接字。

```
[root@i-7jyb6ynm data]# netstat -ntl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State      
tcp        0      0 0.0.0.0:9312                0.0.0.0:*                   LISTEN      
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      
tcp        0      0 0.0.0.0:9306                0.0.0.0:*                   LISTEN      
tcp        0      0 :::3306                     :::*                        LISTEN      
tcp        0      0 :::22                       :::*                        LISTEN      
```

现在我们可以看到处于监听状态的TCP端口和连接。如果你查看所有监听端口，去掉 -t 选项。如果你只想查看UDP端口，使用 -u 选项代替 -t 选项。

注意：不要使用 -a 选项，否则 netstat 会列出所有连接，而不仅仅是监听端口。

## 6.获取进程名、进程号以及用户ID ##

查看端口和连接的信息时，能查看到它们对应的进程名和进程号对系统管理员来说是非常有帮助的。举个例，Apache的http 服务开启80端口，如果你要查看http服务是否已经启动，或者http服务是由apache还是nginx启动的，这时候你可以看看进程名。

使用 -p 选项查看进程信息。

```
[root@i-7jyb6ynm data]# netstat -nltp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name   
tcp        0      0 0.0.0.0:9312                0.0.0.0:*                   LISTEN      23635/searchd       
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      915/sshd            
tcp        0      0 0.0.0.0:9306                0.0.0.0:*                   LISTEN      23635/searchd       
tcp        0      0 :::3306                     :::*                        LISTEN      3228/mysqld         
tcp        0      0 :::22                       :::*                        LISTEN      915/sshd
```

使用 -p 选项时，netstat必须运行在root权限之下，不然它就不能得到运行在root权限下的进程名，而很多服务包括http和ftp都运行在root权限之下。

相比进程名和进程号而言，查看进程的拥有者会更有用。使用 -ep 选项可以同时查看进程名和用户名。

```
[root@i-7jyb6ynm data]# netstat -ltpe
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       User       Inode      PID/Program name   
tcp        0      0 *:sphinxapi                 *:*                         LISTEN      root       544723     23635/searchd       
tcp        0      0 *:ssh                       *:*                         LISTEN      root       8443       915/sshd            
tcp        0      0 *:sphinxql                  *:*                         LISTEN      root       544724     23635/searchd       
tcp        0      0 *:mysql                     *:*                         LISTEN      mysql      453353     3228/mysqld         
tcp        0      0 *:ssh                       *:*                         LISTEN      root       8445       915/sshd
```

上面列出TCP协议下的监听套接字，同时显示进程信息和一些额外信息。这些额外的信息包括用户名和进程的索引节点号。这个命令对网管来说很有用。

注意：假如你将 -n 和 -e 选项一起使用，User列的属性就是用户的ID号，而不是用户名。


## 7.打印统计数据 ##

netstat可以打印出网络统计数据，包括某个协议下的收发包数量，下面列出所有网络包的统计情况：

```
[root@i-7jyb6ynm data]# netstat -s
Ip:
    278001 total packets received
    2 with invalid addresses
    0 forwarded
    0 incoming packets discarded
    277999 incoming packets delivered
    147657 requests sent out
Icmp:
    0 ICMP messages received
    0 input ICMP message failed.
    ICMP input histogram:
    0 ICMP messages sent
    0 ICMP messages failed
    ICMP output histogram:
Tcp:
    294 active connections openings
    28 passive connection openings
    1 failed connection attempts
    9 connection resets received
    1 connections established
    275760 segments received
    145373 segments send out
    47 segments retransmited
    3 bad segments received.
    12 resets sent


...省略...
```

如果想只打印出TCP或UDP协议的统计数据，只要加上对应的选项（-t 和 -u）即可。


## 8.显示内核路由信息 ##

使用 -r 选项打印内核路由信息。打印出来的信息与rout 命令输出的信息一样，我们也可以使用 -n 选项禁止域名解析。

```
[root@i-7jyb6ynm data]# netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
192.168.200.0   0.0.0.0         255.255.255.0   U         0 0          0 eth0
0.0.0.0         192.168.200.1   0.0.0.0         UG        0 0          0 eth0
```

## 9.打印网络接口 ##

netstat也能打印网络接口信息，-i 选项就是为这个功能而生。

```
[root@i-7jyb6ynm data]# netstat -i
Kernel Interface table
Iface       MTU Met    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
eth0       1500   0   282575      0      0      0   152201      0      0      0 BMRU
lo        65536   0        0      0      0      0        0      0      0      0 LRU
```
上面输出的信息比较原始。我们将 -e 选项和 -i 选项搭配使用，可以输出用户友好的信息，输出信息与ifconfig输出的信息一样。

```
[root@i-7jyb6ynm data]# netstat -ie
Kernel Interface table
eth0      Link encap:Ethernet  HWaddr 52:54:55:ED:3C:89  
          inet addr:192.168.200.200  Bcast:192.168.200.255  Mask:255.255.255.0
          inet6 addr: fe80::5054:55ff:feed:3c89/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:282587 errors:0 dropped:0 overruns:0 frame:0
          TX packets:152208 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:302446766 (288.4 MiB)  TX bytes:30868787 (29.4 MiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)

```

## 10.netstat持续输出 ##

我们可以使用netstat的 -c 选项持续输出信息。

```
[root@i-7jyb6ynm data]# netstat -ct
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State      
tcp        0      0 192.168.200.200:ssh         192.168.200.1:9749          ESTABLISHED 
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State      
tcp        0      0 192.168.200.200:ssh         192.168.200.1:9749          ESTABLISHED 
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State      
tcp        0      0 192.168.200.200:ssh         192.168.200.1:9749          ESTABLISHED
```

## 11.打印active状态的连接 ##

active状态的套接字连接用 "ESTABLISHED" 字段表示，所以我们可以使用grep命令获得active状态的连接：
```
[root@i-7jyb6ynm data]# netstat -antp | grep ESTA
tcp        0      0 192.168.200.200:22          192.168.200.1:9749          ESTABLISHED 14455/sshd
```

配合 watch 命令监视 active 状态的连接：
```
watch -d -n0 "netstat -atnp | grep ESTA"
```

## 12.查看服务是否在运行 ##

如果你想看看http，smtp或mysql服务是否在运行，使用grep：

```
[root@i-7jyb6ynm data]# netstat -atple | grep mysql
tcp        0      0 *:mysql		*:*      LISTEN      mysql      453353     3228/mysqld
```

好了，netstat的大部分功能都介绍过了，如果你想知道netstat更高级的功能，阅读它的手册吧（man netstat）。





