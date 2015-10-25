---
layout: post
title: "INADDR_ANY的用法"
date: 2015-10-25
comments: true
category: Linux
tags: Linux
---

###INADDR_ANY的用法

网络程序服务器端socket结构,端口使用`INADDR_ANY`  

```
struct sockaddr_in name;
name.sin_addr.s_addr = htonl(INADDR_ANY);
```

linux下的socket INADDR_ANY表示的是一个服务器上所有的网卡（服务器可能不止一个网卡） 多个本地ip地址都进行绑定端口号，进行侦听。不光是多个网卡的问题. 见如下server listen:

```
80         0.0.0.0          //INADDR_ANY,外部的client ask 从哪个server的地址近来都可以连接到80端口.
8088       192.168.1.11     //外部的client ask 从server地址192.168.1.11进来才可以连接到8088端口.
8089       192.168.1.12     //外部的client ask 从server地址192.168.1.12进来才可以连接到8089端口.
```

也就是说`0.0.0.0`是指本地的地址(也就是代表了所有本地的地址,同一个网卡上也可能有多个地址). 这点上linux,windows系统都是相同的.

而对于在connect中指定了`INADDR_ANY`,那么:  
1. 在语义上一定是连接到本地地址,不可能是外部地址.  
2. INADDR_ANY在语义上有可能是对应了几个本地地址,因此有的系统会根据缺省规则连接本地指定的服务, 而有的系统则因为不能确定用户的任意本地地址是哪个而不能有效连接(如linux和windows不同).

`INADDR_ANY`就是指定地址为`0.0.0.0`的地址,这个地址事实上表示不确定地址,或“所有地址”、“任意地址”。 一般来说， 在各个系统中均定义成为0值。

例如在ubuntu的`/usr/include/netinet/in.h`定义为:  

```
/* Address to accept any incoming messages.  */
#define    INADDR_ANY        ((in_addr_t) 0x00000000)
```

一般情况下，如果你要建立网络服务器应用程序，则你要通知服务器操作系统：请在某地址 xxx.xxx.xxx.xxx上的某端口 yyyy上进行侦听， 并且把侦听到的数据包发送给我。这个过程，你是通过bind()系统调用完成的。  

也就是说，你的程序要绑定服务器的某地址， 或者说：把服务器的某地址上的某端口占为已用。服务器操作系统可以给你这个指定的地址，也可以不给你。如果你的服务器有 多个网卡（每个网卡上有不同的 IP地址），而你的服务（不管是在udp端口上侦听，还是在tcp端口上侦听）， 出于某种原因：可能是你的服务器操作系统可能随时增减IP地址，也有可能 是为了省去确定服务器上有什么网络端口（网卡）的麻烦 —— 可以要在调用bind()的时候，告诉操作系统：“我需要在 yyyy 端口上侦听，所以发送到服务器的这个端口， 不管是哪个网卡/哪个IP地址接收到的数据，都是我处理的。”这时候，服务器程序则在0.0.0.0这个地址上 进行侦听。

例如：

```
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)
……
udp4       0      0  *.7913                 *.*                    
udp4       0      0  *.7911                 *.*
tcp4       0      0  *.ftp                  *.*                    LISTEN
……
……
```

以上这些是网络侦听的情况，其中Local Address 为 “.ftp”、“.7911”等，代表了服务程序绑定了服务器的所有网卡。

1. 我的服务器有N个IP地址，会不会收到重复的数据包？收到数据包后，是不是会重复回复客户端呢？  
答案是：不会收到重复的数据包，也不会重复发送数据。 因为路由的关系，从客户端来的IP包只可能到达其中一个网卡。同时在服务器进程发送数据时，操作系统根据自身维护着的路由表， 决定IP数据包应该 c从哪一个outbound的gateway向目标端发送。根据gateway选择的不同，也就决定了从哪一个网卡／哪个IP地址发送。

2. 为什么不会接收到重复的数据包呢？  
答：因为客户端只向你的服务器上的唯一一个IP地址发送数据了。

3. 为什么不会重复发送数据包呢？  
答：因为发送数据包的路由（路径）是唯一的。如果服务器不知道在发送数据的时候应该向哪个地址发送数据，那么数据就会被发送到“默认网关”上。

4. 如何选择发送数据的路径呢？  
答：依照路由表的要求发送。

5. 如果路由表的记录有重复/有冲突呢，这时候如何选择路径呢？  
答：路由表记录有优先级别。一般来说，Windows操作系统的路由表记录，如果是重复的话，以后来加入的记录为准， 而某些操作系统，象linux/FreeBSD是不允许加入重复的路由表记录的；如果是专用的路由器，有路由选择算法，一般来说， 到达网络上的某一点的路径是可以有很多条的。路由选择算法可以确定“最好的一条路径”，这条路径要么是延时最小的， 要么是通讯费用最低的，要么是带宽最高的，要么是跳点最小的——究竟是如何选择，就看路由器的管理员如何配置了。

`INADDR_ANY`的具体含义是，绑定到`0.0.0.0`。此时，对所有的地址都将是有效的，如果系统考虑冗余，采用多个网卡的话，那么使用此种bind，将在所有网卡上进行绑定。在这种情况下，你可以收到发送到所有有效地址上数据包。

例如：

```
SOCKADDR_IN Local;
Local.sin_addr.s_addr = htonl(INADDR_ANY);
```

另外一种方式如下：

```
SOCKADDR_IN Local;
hostent* thisHost = gethostbyname("");
char* ip = inet_ntoa((struct in_addr *)thisHost->h_addr_list);
Local.sin_addr.s_addr = inet_addr(ip);
```

在这种方式下，将在系统中当前第一个可用地址上进行绑定。在多网卡的环境下，可能会出问题。
最常见的方式：

```
const char LocalIP[] = "192.168.0.100";
SOCKADDR_IN Local;
Local.sin_addr.s_addr = inet_addr(LocalIP);
bind(socket, (LPSOCKADDR)&Local, sizeof(SOCKADDR_IN);
```

bind的安全问题：
如果你在bind时，使用了INADDR_ANY那么，你将可以在所有有效的地址上进行监听，但是Socket有一个特性：可在同一端口上绑定多个Socket。
让我们看看下面的情况：假设你的系统只有一个IP：192.168.0.100，你希望bind到4096端口。对于下面的两种bind，当数据包到达时，谁会接收到呢？

```
Local.sin_addr.s_addr = htonl(INADDR_ANY);
Local.sin_addr.s_addr = inet_addr(“192.168.0.100”);
```

WinSocke库是这样处理的：谁绑定的最明确，谁获取数据包。显然，第二种bind将获取到达的数据包。如果避免这种情况呢？使用 SO_EXECLUSINEADDRUSE选项。需要注意的是，此选项在Windows NT 4 Service Pack 4以后（包括SP4）才可以使用。

示例代码： 

```
#ifndef SO_EXECLUSINEADDRUSE
#define SO_EXECLUSINEADDRUSE ((int)(~SO_REUSEADDR))
#endif 　　
SOCKADDR_IN Local;
BOOL val = TRUE;
Local. sin_family = AF_INET;
Local. sin_port = htons(4096);
Local.sin_addr.s_addr = htonl(INADDR_ANY);
setsocketopt(socket,
SOL_SOCKET,
SO_EXECLUSINEADDRUSE,
(char*)&val,
sizeof(val));
bind(socket, (LPSOCKADDR)&Local, sizeof(SOCKADDR_IN)
```

对于客户端如果绑定`INADDR_ANY`，情况类似。对于TCP而言，在connect()系统调用时将其绑顶到一具体的IP地址。选择的依据是该地址所在 子网到目标地址是可达的（reachable).   

这时通过getsockname（）系统调用就能得知具体使用哪一个地址。对于UDP而言, 情况比较特殊。即使使用connect()系统调用也不会绑定到一具体地址。  

这是因为对UDP使用connect()并不会真正向目标地址发送任何建立连 接的数据，也不会验证到目标地址的可达性。它只是将目标地址的信息记录在内部的socket数据结构之中，共以后使用。只有当调用 sendto()/send()时,由系统内核根据路由表决定由哪一个地址（网卡）发送UDP packet.

###INADDR_LOOPBACK和INADDR_ANY的区别

INADDR_ANY是ANY，是绑定地址0.0.0.0上的监听, 能收到任意一块网卡的连接； INADDR_LOOPBACK, 也就是绑定地址LOOPBAC, 往往是127.0.0.1, 只能收到127.0.0.1上面的连接请求