+++
title= "netcat：一款网络测试的瑞士军刀"
description= "文章简介"
date= 2023-04-08T17:14:54+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= ["app"]

tags=  [ " 网络"," "]

+++

# 一、基本使用简介

~~~
usage: nc [-46CDdFhklNnrStUuvZz] [-I length] [-i interval] [-M ttl]
          [-m minttl] [-O length] [-P proxy_username] [-p source_port]
          [-q seconds] [-s source] [-T keyword] [-V rtable] [-W recvlimit] [-w timeout]
          [-X proxy_protocol] [-x proxy_address[:port]]           [destination] [port]
        Command Summary:
                -4              Use IPv4
                -6              Use IPv6
                -b              Allow broadcast
                -C              Send CRLF as line-ending
                -D              Enable the debug socket option
                -d              Detach from stdin
                -F              Pass socket fd
                -h              This help text
                -I length       TCP receive buffer length
                -i interval     Delay interval for lines sent, ports scanned
                -k              Keep inbound sockets open for multiple connects（配合 -l 选项使用，可以重复接受客户端连接。）
                -l              Listen mode, for inbound connects（开启“监听模式”，nc 作为【服务端】注：如不加该选项，nc 默认作为客户端）
                -M ttl          Outgoing TTL / Hop Limit
                -m minttl       Minimum incoming TTL / Hop Limit
                -N              Shutdown the network socket after EOF on stdin
                -n              Suppress name/port resolutions
                -O length       TCP send buffer length
                -P proxyuser    Username for proxy authentication
                -p port         Specify local port for remote connects（指定“端口号”）
                -q secs         quit after EOF on stdin and delay of secs（让 nc 延时（N 秒）再退出）
                -r              Randomize remote ports
                -S              Enable the TCP MD5 signature option
                -s source       Local source address
                -T keyword      TOS value
                -t              Answer TELNET negotiation
                -U              Use UNIX domain socket
                -u              UDP mode（使用 UDP 协议 注：如不加该选项，默认是 TCP 协议）
                -V rtable       Specify alternate routing table
                -v              Verbose （显示详细信息）
                -W recvlimit    Terminate after receiving a number of packets
                -w timeout      Timeout for connects and final net reads（设置连接的超时间隔（N 秒））
                -X proto        Proxy protocol: "4", "5" (SOCKS) or "connect"（指定代理的类型）
                -x addr[:port]  Specify proxy address and port（以 IP:port 的格式指定代理的位置。）
                -Z              DCCP mode
                -z              Zero-I/O mode [used for scanning]
        Port numbers can be individual or ranges: lo-hi [inclusive]
        
        
-g <网关> # 设置路由器跃程通信网关，最多可设置8个。
-G<指向器数目> # 设置来源路由指向器，其数值为4的倍数。
-h 在线帮助。
-i<延迟秒数> 设置时间间隔，以便传送信息及扫描通信端口。
-l 使用监听模式，管控传入的资料。
-n 直接使用IP地址，而不通过域名服务器。
-o<输出文件> # 指定文件名称，把往来传输的数据以16进制字码倾倒成该文件保存。
-p<通信端口> # 设置本地主机使用的通信端口。
-r 乱数指定本地与远端主机的通信端口。
-s<来源位址> # 设置本地主机送出数据包的IP地址。
-u 使用UDP传输协议。
-v 显示指令执行过程。
-w<超时秒数> # 设置等待连线的时间。
-z 使用0输入/输出模式，只在扫描通信端口时使用
~~~

# 二、netcat的使用

1. ## 渗透测试（端口扫描）

~~~
nc -znv 127.0.0.1 1-1024 2>&1 | grep succeeded

选项 -z
意思是：开启“zero-I/O 模式”。该模式指的是：nc 只判断某个监听端口是否能连上，连上后【不】与对端进行数据通讯。

选项 -n
由于测试的是【IP 地址】，用该选项告诉 nc，【无须】进行域名（DNS）解析；
反之，如果你要测试的主机是基于【域名】，就【不能】用“选项 -n”

选项 -v
-v 选项前面也聊过，这里要特地强调一下。
对 nc 的其它用法，-v 选项是可加可不加滴；但对于“端口扫描”而言，一定要有这个选项——否则你【看不到】扫描结果

2>&1 | grep succeeded
过滤掉不成功的信息
~~~

2. ## 要判断某个主机的监听端口是否能连上

   ~~~
   nc -nv 127.0.0.1 80 
   
   选项 -v
   如果你是 nc 的新手，建议总是带上这个选项——通过更详细的输出，能帮你搞明白状况。
   
   选项 -n
   由于测试的是【IP 地址】，用该选项告诉 nc，【无须】进行域名（DNS）解析；
   反之，如果你要测试的主机是基于【域名】，就【不能】用“选项 -n”
   
   选项 -w
   超时设置 在测试链接的时候，如果你没使用 -w 这个超时选项，默认情况下 nc 会等待很久，然后才告诉你连接失败。如果你所处的网络环境稳定且高速（比如：局域网内），那么，你可以追加“-w 选项”，设置一个比较小的超时值。在下面的例子中，超时值设为3秒。
   nc -nv -w 3 x.x.x.x xx
   
   UDP 通常情况下，要测试的端口都是 TCP 协议的端口；如果你碰到特殊情况，需要测试某个 UDP 的端口是否可达。nc 同样能胜任。只需要追加 -u 选项。
   ~~~

   ## 3.监听服务器

   ~~~
   nc -lv -port 7000
   ~~~

   ## 4.传输文件

   4.1 单个文件

   ~~~
   服务器
   nc -l -p 7000 > file2
   
   客户端
   nc 127.0.0.1 7000 < file1
   
   性能优势 
   
   用 nc 传输文件，相当于是：直接在【裸 TCP】层面传输。你可以通俗理解为：【没有】应用层。如果你传输的文件【超级大】或者文件数
   量【超级多】，用 nc 传输文件的性能优势会很明显（相比“FTP、SSH、共享目录…”而言）
   ~~~

   4.2 目录

   ~~~
   服务器
    nc -lv -p 7000 | tar xvf -
   
   客户端
   tar cvf - * | nc -nv 127.0.0.1 7000
   
   管道前面表示把当前目录的所有文件打包为 - 
   ~~~

   

   ## 5. 网速吞吐量测试

   ~~~
   服务器
   nc -nvv -l -p 7000 | pv
   
   客户端
   time nc -n 127.0.0.1 7000 < /dev/zero
   
   -n是不要解析域名，避免解析域名造成时间误差
   其实上面两种方法都把建立连接的握手时间以及 TCP 窗口慢启动的时间给计算进去了，不是特别精确，最精确的方式是搭配 pv 命令（监控统计管道数据的速度）
   ~~~

   