# Linux 命令行中使用 tcpdump

## 1. 介绍

Tcpdump 是一个命令行实用程序，可让您捕获和分析通过系统的网络流量。它通常用于帮助解决网络问题以及安全工具。tcpdump 是一个功能强大且多功能的工具，包含许多选项和过滤器，可用于多种情况。由于它是一个命令行工具，因此非常适合在没有 GUI 的远程服务器或设备中运行，以收集稍后可以分析的数据。它还可以在后台启动或使用 crontab 等工具作为计划作业启动。

## 2. 安装

Tcpdump 包含在多个 Linux 发行版中，因此您很可能已经安装了它。使用以下命令检查系统上是否安装了 tcpdump：

```bash
$ which tcpdump
/usr/sbin/tcpdump
```

如果未安装 tcpdump，您可以使用发行版的包管理器来安装它。例如，在 CentOS 或 Red Hat Enterprise Linux 上，如下所示：

```bash
$ sudo dnf install -y tcpdump
```

Tcpdump 需要`libpcap`，这是一个用于网络数据包捕获的库。如果未安装，它将自动添加为依赖项。

## 3. 使用 tcpdump 抓包

1. ##### 查看哪些接口可用于捕获: `tcpdump --list-interfaces `（或`-D`简称）

```bash
$ sudo tcpdump -D
1.eth0
2.virbr0
3.eth1
4.any (Pseudo-device that captures on all interfaces)
5.lo [Loopback]
```

##### 2. 抓包指令：`tcpdump -i any -c 1 -nn`

```bash
$ sudo tcpdump -i any -c 1 -nn
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes
23:56:24.292206 IP 192.168.64.28.22 > 192.168.64.1.35110: Flags [P.], seq 166198580:166198776, ack 
1 packets captured
1 packets received by filter
0 packets dropped by kernel
```

指令解析：

- `-i any`: "-i" 选项用于指定要监听的网络接口。这里的 "any" 表示监听所有可用的网络接口，包括以太网接口、无线接口等。
- `-c 5`: "-c" 选项用于指定要捕获的数据包数量的限制。这里的 "5" 表示只捕获 5 个数据包后就停止捕获。
- `-nn`: "-n" 选项用于禁用将网络地址和端口转换为主机名和服务名的功能。"-nn" 组合表示禁用这种转换，显示原始的 IP 地址和端口号。

##### 3. 网络数据包格式解析

tcpdump 捕获的典型 TCP 数据包如下所示：

```text
08:41:13.729687 IP 192.168.64.28.22 > 192.168.64.1.41916: Flags [P.], seq 196:568, ack 1, win 6379, options [nop,nop,TS val 2575629829 ecr 4020648114], length 0
```

- `08:41:13.729687`: 这是捕获该数据包的时间戳，格式为时:分:秒.毫秒。

- `IP`: 表示该数据包是一个 IPv4 数据包。对于`IPv6`数据包，该值为`IP6`

- `192.168.64.28.22`: 这是源 IP 地址和端口号。IP 地址为 192.168.64.28，端口号为 22。这意味着该数据包是从 IP 地址为 192.168.64.28、端口号为 22 的主机发送出去的。

- `192.168.64.1.41916`: 这是目标 IP 地址和端口号。IP 地址为 192.168.64.1，端口号为 41916。这表示该数据包的目标是 IP 地址为 192.168.64.1、端口号为 41916 的主机。

- `Flags [P.]`: 这是 TCP 数据包的标志位。在这个示例中，"[P.]" 表示数据包中有数据（P），并且是 PUSH 标志位被设置。

  | Value | Flag Type | 描述                          |
  | ----- | --------- | ----------------------------- |
  | S     | SYN       | Connection Start（链接开始）  |
  | F     | FIN       | Connection Finish（链接完成） |
  | P     | PUSH      | Data push（数据推送）         |
  | R     | RST       | Connection reset（链接重置）  |
  | .     | ACK       | Acknowledgment（确认）        |

  该字段也可以是这些值的组合，例如`[S.]`对于`SYN-ACK`数据包。

- `seq 196:568`: 表示该数据包包含该流的字节 196 到 568。

- `ack 1`: 这表示该数据包是一个确认数据包，并且确认了序列号为 1 的数据。

- `win 6379`: 这是窗口大小（Window Size），表示发送方可以接收的字节数量。

- `options [nop,nop,TS val 2575629829 ecr 4020648114]`: 这是 TCP 选项，请参阅[传输控制协议 (TCP) 参数](https://www.iana.org/assignments/tcp-parameters/tcp-parameters.xhtml)。。在这个示例中，选项包括两个 "nop"（空操作）以及时间戳选项。"TS val" 表示时间戳值，这里的值为 2575629829。"ecr" 表示确认应答时间戳，这里的值为 4020648114。

- `length 0`: 这表示数据包中的数据长度为 0，即没有有效负载数据。

## 4. 网络数据包过滤指令

###### 1. Protocol：根据协议过滤数据包，在命令行中指定协议。

例如，只使用以下命令捕获 ICMP 数据包:

```bash
$ sudo tcpdump -i any -c5 icmp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes
```

###### 2. Host：通过使用 `host` 将捕获限制为仅与特定主机相关的数据包:

```bash
$ sudo tcpdump -i any -c5 -nn host 54.204.39.132
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes
09:54:20.042023 IP 192.168.122.98.39326 > 54.204.39.132.80: Flags [S], seq 1375157070, win 29200, options [mss 1460,sackOK,TS val 122350391 ecr 0,nop,wscale 7], length 0
09:54:20.088127 IP 54.204.39.132.80 > 192.168.122.98.39326: Flags [S.], seq 1935542841, ack 1375157071, win 28960, options [mss 1460,sackOK,TS val 522713542 ecr 122350391,nop,wscale 9], length 0
09:54:20.088204 IP 192.168.122.98.39326 > 54.204.39.132.80: Flags [.], ack 1, win 229, options [nop,nop,TS val 122350437 ecr 522713542], length 0
09:54:20.088734 IP 192.168.122.98.39326 > 54.204.39.132.80: Flags [P.], seq 1:113, ack 1, win 229, options [nop,nop,TS val 122350438 ecr 522713542], length 112: HTTP: GET / HTTP/1.1
09:54:20.129733 IP 54.204.39.132.80 > 192.168.122.98.39326: Flags [.], ack 113, win 57, options [nop,nop,TS val 522713552 ecr 122350438], length 0
5 packets captured
5 packets received by filter
0 packets dropped by kernel
```

###### 3. Port：若要基于所需的服务或端口的数据包

例如，使用以下命令捕获与 Web (HTTP)服务相关的数据包:

```bash
$ sudo tcpdump -i any -c5 -nn port 80
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes
09:58:28.790548 IP 192.168.122.98.39330 > 54.204.39.132.80: Flags [S], seq 1745665159, win 29200, options [mss 1460,sackOK,TS val 122599140 ecr 0,nop,wscale 7], length 0
09:58:28.834026 IP 54.204.39.132.80 > 192.168.122.98.39330: Flags [S.], seq 4063583040, ack 1745665160, win 28960, options [mss 1460,sackOK,TS val 522775728 ecr 122599140,nop,wscale 9], length 0
09:58:28.834093 IP 192.168.122.98.39330 > 54.204.39.132.80: Flags [.], ack 1, win 229, options [nop,nop,TS val 122599183 ecr 522775728], length 0
09:58:28.834588 IP 192.168.122.98.39330 > 54.204.39.132.80: Flags [P.], seq 1:113, ack 1, win 229, options [nop,nop,TS val 122599184 ecr 522775728], length 112: HTTP: GET / HTTP/1.1
09:58:28.878445 IP 54.204.39.132.80 > 192.168.122.98.39330: Flags [.], ack 113, win 57, options [nop,nop,TS val 522775739 ecr 122599184], length 0
5 packets captured
5 packets received by filter
0 packets dropped by kernel
```

###### 4. Source IP/hostname：可以根据源或目标 IP 地址或主机名过滤数据包。

例如，要从主机192.168.122.98捕获数据包:

```bash
$ sudo tcpdump -i any -c5 -nn src 192.168.122.98
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes
10:02:15.220824 IP 192.168.122.98.39436 > 192.168.122.1.53: 59332+ A? opensource.com. (32)
10:02:15.220862 IP 192.168.122.98.39436 > 192.168.122.1.53: 20749+ AAAA? opensource.com. (32)
10:02:15.364062 IP 192.168.122.98.39334 > 54.204.39.132.80: Flags [S], seq 1108640533, win 29200, options [mss 1460,sackOK,TS val 122825713 ecr 0,nop,wscale 7], length 0
10:02:15.409229 IP 192.168.122.98.39334 > 54.204.39.132.80: Flags [.], ack 669337581, win 229, options [nop,nop,TS val 122825758 ecr 522832372], length 0
10:02:15.409667 IP 192.168.122.98.39334 > 54.204.39.132.80: Flags [P.], seq 0:112, ack 1, win 229, options [nop,nop,TS val 122825759 ecr 522832372], length 112: HTTP: GET / HTTP/1.1
5 packets captured
5 packets received by filter
0 packets dropped by kernel
```

Note：tcpdump 为多个服务(如名称解析(端口53)和 HTTP (端口80))捕获源 IP 地址为192.168.122.98的数据包。由于响应数据包的源 IP 不同，因此不显示响应数据包。

相反，可以使用 `dst`  筛选器根据目标 IP/主机名进行筛选:

```bash
$ sudo tcpdump -i any -c5 -nn dst 192.168.122.98
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes
10:05:03.572931 IP 192.168.122.1.53 > 192.168.122.98.47049: 2248 1/0/0 A 54.204.39.132 (48)
10:05:03.572944 IP 192.168.122.1.53 > 192.168.122.98.47049: 33770 0/0/0 (32)
10:05:03.621833 IP 54.204.39.132.80 > 192.168.122.98.39338: Flags [S.], seq 3474204576, ack 3256851264, win 28960, options [mss 1460,sackOK,TS val 522874425 ecr 122993922,nop,wscale 9], length 0
10:05:03.667767 IP 54.204.39.132.80 > 192.168.122.98.39338: Flags [.], ack 113, win 57, options [nop,nop,TS val 522874436 ecr 122993972], length 0
10:05:03.672221 IP 54.204.39.132.80 > 192.168.122.98.39338: Flags [P.], seq 1:643, ack 113, win 57, options [nop,nop,TS val 522874437 ecr 122993972], length 642: HTTP: HTTP/1.1 302 Found
5 packets captured
5 packets received by filter
0 packets dropped by kernel
```

###### 5. Complex expressions：可以通过使用逻辑运算符创建更复杂的表达式来组合筛选器。

例如，我们只过滤 HTTP 服务(端口80)和源 IP 地址192.168.122.98或54.204.39.132的数据包。这是检查同一流程的两端的快速方法，请使用以下命令:

```bash
$ sudo tcpdump -i any -c5 -nn "port 80 and (src 192.168.122.98 or src 54.204.39.132)"
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes
10:10:37.602214 IP 192.168.122.98.39346 > 54.204.39.132.80: Flags [S], seq 871108679, win 29200, options [mss 1460,sackOK,TS val 123327951 ecr 0,nop,wscale 7], length 0
10:10:37.650651 IP 54.204.39.132.80 > 192.168.122.98.39346: Flags [S.], seq 854753193, ack 871108680, win 28960, options [mss 1460,sackOK,TS val 522957932 ecr 123327951,nop,wscale 9], length 0
10:10:37.650708 IP 192.168.122.98.39346 > 54.204.39.132.80: Flags [.], ack 1, win 229, options [nop,nop,TS val 123328000 ecr 522957932], length 0
10:10:37.651097 IP 192.168.122.98.39346 > 54.204.39.132.80: Flags [P.], seq 1:113, ack 1, win 229, options [nop,nop,TS val 123328000 ecr 522957932], length 112: HTTP: GET / HTTP/1.1
10:10:37.692900 IP 54.204.39.132.80 > 192.168.122.98.39346: Flags [.], ack 113, win 57, options [nop,nop,TS val 522957942 ecr 123328000], length 0
5 packets captured
5 packets received by filter
0 packets dropped by kernel
```

## 5. 检查数据包内容

在前面的示例中，我们只检查数据包的头信息，如源、目的地、端口等。有时，这就是我们解决网络连接问题所需要的全部内容。

但是，有时候我们需要检查数据包的内容，以确保我们发送的消息包含我们需要的内容或者我们收到了预期的响应。

为了查看数据包内容，tcpdump 提供了两个附加标志:

* `-X` 用于以十六进制打印内容
* `-A` 用于以 ASCII 打印内容。

例如，像下面这样检查 Web 请求的 HTTP 内容:

```bash
$ sudo tcpdump -i any -c10 -nn -A port 80
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes
13:02:14.871803 IP 192.168.122.98.39366 > 54.204.39.132.80: Flags [S], seq 2546602048, win 29200, options [mss 1460,sackOK,TS val 133625221 ecr 0,nop,wscale 7], length 0
E..<..@.@.....zb6.'....P...@......r............
............................
13:02:14.910734 IP 54.204.39.132.80 > 192.168.122.98.39366: Flags [S.], seq 1877348646, ack 2546602049, win 28960, options [mss 1460,sackOK,TS val 525532247 ecr 133625221,nop,wscale 9], length 0
E..<..@./..a6.'...zb.P..o..&...A..q a..........
.R.W.......	................
```

## 6. 将抓取的数据包保存到文件中

tcpdump 提供的另一个有用特性是能够将捕获保存到文件中，以便稍后分析结果。

例如，这允许您在一夜之间以批处理模式捕获数据包，并在早上验证结果。当有太多数据包需要分析时，这也很有帮助，因为实时捕获可能发生得太快。

* 要将数据包保存到文件中而不是显示在屏幕上，可以使用选项 `-w` (表示写) :

  ```bash
  $ sudo tcpdump -i any -c10 -nn -w webserver.pcap port 80
  [sudo] password for ricardo: 
  tcpdump: listening on any, link-type LINUX_SLL (Linux cooked), capture size 262144 bytes
  10 packets captured
  10 packets received by filter
  0 packets dropped by kernel
  ```

  此命令将输出保存在名为 webserver.pcap 的文件中。那个`.Pcap` 扩展代表“包捕获”，是这种文件格式的约定。

* tcpdump 创建二进制格式的文件，因此不能简单地用文本编辑器打开它。要读取文件的内容，使用 `-r`  (for read)选项执行:

  Note: 可以使用我们讨论过的任何过滤器来过滤文件中的内容，就像对待实时数据一样。

  ```bash
  $ tcpdump -nn -r webserver.pcap src 54.204.39.132
  reading from file webserver.pcap, link-type LINUX_SLL (Linux cooked)
  13:36:57.718932 IP 54.204.39.132.80 > 192.168.122.98.39378: Flags [S.], seq 1999298316, ack 3709732620, win 28960, options [mss 1460,sackOK,TS val 526052949 ecr 135708029,nop,wscale 9], length 0
  13:36:57.756979 IP 54.204.39.132.80 > 192.168.122.98.39378: Flags [.], ack 113, win 57, options [nop,nop,TS val 526052959 ecr 135708068], length 0
  13:36:57.760122 IP 54.204.39.132.80 > 192.168.122.98.39378: Flags [P.], seq 1:643, ack 113, win 57, options [nop,nop,TS val 526052959 ecr 135708068], length 642: HTTP: HTTP/1.1 302 Found
  13:36:58.022089 IP 54.204.39.132.80 > 192.168.122.98.39378: Flags [F.], seq 643, ack 114, win 57, options [nop,nop,TS val 526053025 ecr 135708327], length 0
  ```


## 参考链接：https://opensource.com/article/18/10/introduction-tcpdump
