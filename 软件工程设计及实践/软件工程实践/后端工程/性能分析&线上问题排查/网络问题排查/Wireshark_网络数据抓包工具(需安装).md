# Wireshark_网络数据抓包工具(需安装,底层使用tcpdump实现)

> 对于 IT 专业人员来说，很少有工具能像首选网络数据包捕获工具 Wireshark 那样有用。Wireshark 将帮助您捕获网络数据包并以粒度级别显示它们。一旦这些数据包被分解，您就可以使用它们进行实时或离线分析。该工具可让您将网络流量置于显微镜下，然后对其进行过滤和深入研究，放大问题的根本原因，协助网络分析并最终实现网络安全。

## 什么是 Wireshark？

Wireshark 是一种网络协议分析器，或者是一种从网络连接（例如从计算机到家庭办公室或互联网）捕获数据包的应用程序。数据包是典型以太网中离散数据单元的名称。

Wireshark 是世界上最常用的数据包嗅探器。与任何其他数据包嗅探器一样，Wireshark 执行三件事：

1. **数据包捕获：** Wireshark 实时侦听网络连接，然后捕获整个流量流 - 很可能一次捕获数万个数据包。
2. **过滤：** Wireshark 能够使用过滤器对所有这些随机实时数据进行切片和切分。通过应用过滤器，您可以仅获取您需要查看的信息。
3. **可视化：** Wireshark 与任何优秀的数据包嗅探器一样，允许您直接深入到网络数据包的中间。它还允许您可视化整个对话和网络流。

![显示 Wireshark 中数据包捕获的屏幕截图](./img/Wireshark_网络数据抓包工具(需安装)/packet-capture-in-wireshark.jpg)

图 1：在 Wireshark 中查看数据包捕获

数据包嗅探可以比作洞穴探险——进入洞穴并四处徒步旅行。在网络上使用 Wireshark 的人有点像那些使用手电筒来看看他们能找到什么很酷的东西的人。毕竟，当在网络连接上使用 Wireshark（或在洞穴中使用手电筒）时，您实际上是在使用一种工具来搜索隧道和管道以查看您能看到的内容。

## Wireshark 有何用途？

Wireshark 有很多用途，包括对存在性能问题的[网络进行故障排除](https://www.comptia.org/content/guides/a-guide-to-network-troubleshooting)。网络安全专业人员经常使用 Wireshark 来跟踪连接、查看可疑网络事务的内容并识别网络流量突发。它是任何 IT 专业人员工具包的主要组成部分 - 希望 IT 专业人员具备使用它的知识。

## 什么时候应该使用 Wireshark？

Wireshark 是政府机构、教育机构、企业、小型企业和非营利组织等用来解决网络问题的安全工具。此外，Wireshark 还可以用作学习工具。

信息安全新手可以使用 Wireshark 作为工具来了解网络流量分析、涉及特定协议时通信如何进行以及发生某些问题时哪里出错。

当然，Wireshark 并不能做所有事情。

* 首先，它无法帮助对[网络协议了解甚少的用户。](https://www.comptia.org/content/guides/what-is-a-network-protocol)没有任何工具，无论多么酷，都可以很好地取代知识。换句话说，要正确使用 Wireshark，您需要准确了解网络的运行方式。这意味着，您需要了解诸如三向 TCP 握手和各种协议（包括 TCP、UDP、DHCP 和 ICMP）等内容。

* 其次，正常情况下，Wireshark 无法从网络上的所有其他系统获取流量。在使用称为交换机的设备的现代网络上，Wireshark（或任何其他标准数据包捕获工具）只能嗅探本地计算机与其正在通信的远程系统之间的流量。

* 第三，虽然 Wireshark 可以显示格式错误的数据包并应用颜色编码，但它没有实际的警报；Wireshark 不是入侵检测系统 (IDS)。

* 第四，Wireshark 无法帮助解密加密流量。

* 最后，欺骗 IPv4 数据包非常容易。Wireshark 无法真正告诉您它在捕获的数据包中找到的特定 IP 地址是否真实。这需要 IT 专业人员具备更多的专业知识以及额外的软件。

## 常见 Wireshark 用例

以下是 Wireshark 捕获如何帮助识别问题的常见示例。下图显示了家庭网络的问题，其中互联网连接速度非常慢。

如图所示，路由器认为公共目的地无法到达。这是通过深入研究 IPv6 互联网消息控制协议 (ICMP) 流量（标记为黑色）发现的。在 Wireshark 中，任何标记为黑色的数据包都被认为反映了某种问题。

![显示如何使用 Wireshark 深入分析数据包以识别网络问题的屏幕截图](./img/Wireshark_网络数据抓包工具(需安装)/network-problem-wireshark.jpg)

<center>图 2：使用 Wireshark 深入分析数据包以识别网络问题</center>

在这种情况下，Wireshark 帮助确定路由器工作不正常，无法轻松找到 YouTube。通过重新启动电缆调制解调器解决了该问题。当然，虽然这个特定问题不需要使用 Wireshark，但权威地解决这个问题还是很酷的。

当您再看一下图 2 的底部时，您可以看到一个特定的数据包突出显示。这显示了 TCP 数据包的内部结构，该数据包是传输层安全 (TLS) 对话的一部分。这是一个很好的例子，说明了如何深入了解捕获的数据包。

使用 Wireshark 不允许您读取数据包的加密内容，但您可以识别浏览器和 YouTube 用于加密内容的 TLS 版本。有趣的是，在监听过程中加密转变为 TLS 1.2 版本。

Wireshark 通常用于识别更复杂的网络问题。例如，如果网络经历过多的重传，则可能会发生拥塞。通过使用 Wireshark，您可以识别特定的重传问题，如下图 3 所示。

![显示如何在 Wireshark 中查看数据包流统计信息的屏幕截图](./img/Wireshark_网络数据抓包工具(需安装)/packet-flow-statistics-in-wireshark.jpg)

图 3：使用 Wireshark 查看数据包流统计信息以识别重传

通过确认此类问题，您可以重新配置路由器或交换机以加快流量。

## 如何使用 Wireshark

您可以从 [www.wireshark.org](https://www.wireshark.org/) 免费下载 Wireshark 。它还可以作为[GNU 通用公共许可证](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)版本 2 下的开源应用程序免费提供。

### 如何在 Windows 上安装 Wireshark

如果您是 Windows 操作系统用户，请下载适合您的特定版本的版本。例如，如果您使用 Windows 10，则可以获取 64 位 Windows 安装程序并按照向导进行安装。要安装，您需要管理员权限。

### 如何在 Linux 上安装 Wireshark

如果您有 [Linux 系统](https://www.comptia.org/blog/all-about-linux-and-linux) ，则可以使用以下顺序安装 Wireshark（请注意，您需要拥有 root 权限）：

```bash
$ sudo apt-get install wireshark
$ sudo dpkg-reconfigure wireshark-common
$ sudo usermod -a -G wireshark $USER
$ newgrp wireshark
```

完成上述步骤后，注销并重新登录，然后启动 Wireshark：

```bash
$ wireshark &
```

### 如何使用 Wireshark 抓包

安装 Wireshark 后，您就可以开始抓取网络流量。但请记住：要捕获任何数据包，您需要在计算机上拥有适当的权限才能将 Wireshark 置于混杂模式。

- 在 Windows 系统中，这通常意味着您拥有管理员访问权限。
- 在 Linux 系统中，这通常意味着您拥有 root 访问权限。

只要您拥有正确的权限，您就有多种选择来实际开始捕获。也许最好的方法是从主窗口中选择“捕获>>选项”。这将打开“Capture Interfaces”窗口，如下图 4 所示。

![显示 Wireshark 中的捕获接口对话框的屏幕截图](./img/Wireshark_网络数据抓包工具(需安装)/capture-interfaces-dialog-wireshark.jpg)

图 4：Wireshark 中的“捕获接口”对话框

该窗口将列出所有可用的接口。在这种情况下，Wireshark提供了几种可供选择。

在此示例中，我们将选择以太网 3 接口，它是最活跃的接口。Wireshark 通过显示一条移动线来可视化流量，该线代表网络上的数据包。

选择网络接口后，您只需单击“开始”按钮即可开始捕获。捕获开始后，可以查看屏幕上显示的数据包，如下图 5 所示。

![显示 Wireshark 捕获数据包的屏幕截图](./img/Wireshark_网络数据抓包工具(需安装)/wireshark-capturing-packets.jpg)

图5：Wireshark抓包

捕获所有所需的数据包后，只需单击顶部的红色方形按钮即可。现在您有一个静态数据包捕获可供研究。

### Wireshark 中颜色编码的含义

现在您有了一些数据包，是时候弄清楚它们的含义了。Wireshark 尝试通过应用常识性颜色编码来帮助您识别数据包类型。下表描述了主要数据包类型的默认颜色。

|                        |                                                     |
  | :--------------------- | :-------------------------------------------------- |
  | **Wireshark 中的颜色** | **数据包类型**                                      |
  | 浅紫色                 | 传输控制协议                                        |
  | 浅蓝色                 | UDP协议                                             |
  | 黑色的                 | 有错误的数据包                                      |
  | 浅绿色                 | HTTP 流量                                           |
  | 浅黄色                 | Windows 特定流量，包括服务器消息块 (SMB) 和 NetBIOS |
  | 暗黄色                 | 路由                                                |
  | 深灰色                 | TCP SYN、FIN 和 ACK 流量                            |



默认的着色方案如下图 6 所示。您可以通过转到查看 >> 着色规则来查看此方案。

![显示 Wireshark 中默认着色规则的屏幕截图。](./img/Wireshark_网络数据抓包工具(需安装)/default-coloring-rules-in-wireshark.jpg)

图 6：默认着色规则

您甚至可以更改默认值或应用自定义规则。如果您根本不需要任何着色，请转至“查看”，然后单击“对数据包列表进行着色”。这是一个切换开关，因此如果您想要恢复着色，只需返回并再次单击“为数据包列表着色”即可。甚至可以对计算机之间的特定对话进行着色。

在下面的图 7 中，您可以看到标准 UDP（浅蓝色）、TCP（浅紫色）、TCP 握手（深灰色）和路由流量（黄色）。

![显示 Wireshark 中彩色数据包的屏幕截图。](./img/Wireshark_网络数据抓包工具(需安装)/colorized-packets-in-wireshark.jpg)

图 7：在 Wireshark 中查看彩色数据包

但是，您不仅限于仅通过颜色进行解释。可以查看整个数据包捕获的输入/输出 (I/O) 统计信息。

在 Wireshark 中，只需转到“统计”>>“I/O 图表”，您就会看到类似于图 8 所示的图表。

![显示 Wireshark 中输入/输出流量图表的屏幕截图。](./img/Wireshark_网络数据抓包工具(需安装)/input-output-traffic-graph-wireshark.jpg)

图 8：在 Wireshark 中查看输入/输出流量图

该特定图表显示了家庭办公室产生的典型流量。图中的峰值是由于使用一些 Linux 系统生成 [分布式拒绝服务 (DDoS) 攻击而导致的流量突发。](https://www.comptia.org/content/guides/what-is-ddos-protection-tools-stopping/)

在这种情况下，产生了三个主要的流量突发。很多时候，网络安全专家使用 Wireshark 作为一种快速但肮脏的方法来识别攻击期间的流量爆发。

还可以捕获一个系统与另一个系统之间生成的流量。如果您转到“统计”，然后选择“对话”，您将看到端点之间的对话摘要，如下图 9 所示。

![显示 Wireshark 中端点对话的屏幕截图。](./img/Wireshark_网络数据抓包工具(需安装)/endpoint-conversations-in-wireshark.jpg)

图 9：在 Wireshark 中查看端点对话

在上述案例中，Wireshark 用于查看是否可以跟踪在客户端网络上运行的 MCI 通信中的旧设备。

事实证明，客户端甚至不知道该设备在网络上。因此，它被删除，有助于[使网络更加安全。](https://www.comptia.org/content/guides/network-security-basics-definition-threats-and-solutions)另请注意，此网络连接正在经历大量流向 Amazon（当时管理 AWS 中的服务器）和 Box.com（当时使用 Box 进行系统备份）的流量。

在某些情况下，甚至可以使用 Wireshark 来识别源和目标流量的地理位置。如果单击屏幕底部的“地图”按钮（如上图 9 所示），Wireshark 将向您显示一张地图（图 10），提供对您已识别的 IP 地址位置的最佳猜测。

![显示 Wireshark 中地理估计的屏幕截图。](./img/Wireshark_网络数据抓包工具(需安装)/geographic-estimations-in-wireshark.jpg)

图 10：在 Wireshark 中查看地理估计

由于 IPv4 地址很容易被欺骗，因此您不能完全依赖此地理信息。但它可以相当准确。

### 如何在 Wireshark 中过滤和检查数据包

您可以通过两种方式应用 Wireshark 过滤器：

1. 在“显示过滤器”窗口中，位于屏幕顶部
  2. 通过突出显示数据包（或数据包的一部分）并右键单击该数据包

Wireshark 过滤器使用关键短语，如下所示：

|           |                           |
| --------- | ------------------------- |
| ip.addr   | 指定 IPv4 地址            |
| ipv6.addr | 指定 IPv6 地址            |
| src       | 来源 - 数据包来自哪里     |
| dst       | 目的地 - 数据包要去的地方 |

您还可以使用以下值：

|      |                                                          |
  | ---- | -------------------------------------------------------- |
  | &&   | 表示“和”，如“选择 192.168.2.1 和 192.168.2.2 的 IP 地址” |
  | ==   | 表示“等于”，如“仅选择 IP 地址 192.168.2.1”               |
  | ！   | 意思是“不”，不显示特定的 IP 地址或源端口                 |


有效的过滤规则始终为绿色。如果您在过滤规则上犯了错误，该框将变成鲜艳的粉红色。

让我们从一些基本规则开始。例如，假设您想查看内部某处仅包含 IP 地址 18.224.161.65 的数据包。您将创建以下命令行，并将其放入“过滤器”窗口中：

ip地址==18.224.161.65 

图 11 显示了添加该过滤器的结果：

![显示应用于 Wireshark 中捕获的过滤器的屏幕截图](./img/Wireshark_网络数据抓包工具(需安装)/filter-to-capture-in-wireshark.jpg)

图 11：在 Wireshark 中对捕获应用过滤器

或者，您可以突出显示数据包的 IP 地址，然后为其创建过滤器。选择 IP 地址后，单击鼠标右键，然后选择“应用为过滤器”选项。

然后您将看到一个包含其他选项的菜单。其中之一称为“选定”。如果您选择“选定”，则 Wireshark 将创建一个过滤器，仅显示包含该 IP 地址的数据包。

您还可以决定使用以下过滤器过滤掉特定的 IP 地址，如图 12 所示：

!ip.addr==18.224.161.65 

![显示如何在 Wireshark 中过滤特定 IP 地址的屏幕截图](./img/Wireshark_网络数据抓包工具(需安装)/filter-ip-addresses-wireshark.jpg)

图 12：在 Wireshark 中过滤掉特定 IP 地址

您不仅限于 IPv4 地址。例如，如果您想查看网络上某台特定计算机是否处于活动状态并使用 IPv6 地址，您可以打开 Wireshark 的副本并应用以下规则：

ipv6.dst == 2607:f8b0:400a:15::b 

图 13 显示了相同的规则。

![显示 Wireshark 中 IPv6 过滤器的屏幕截图](./img/Wireshark_网络数据抓包工具(需安装)/ipv6-filter-in-wireshark.jpg)

图 13：在 Wireshark 中应用 IPv6 过滤器

显然，这个系统运行良好，可以在网络上进行通信。有很多可能性。

其他过滤器包括：

|                                                  |                                                              |
| ------------------------------------------------ | ------------------------------------------------------------ |
| tcp.port==8080                                   | 过滤数据包以显示您自己选择的端口 - 在本例中为端口 8080       |
| !(ip.src == 162.248.16.53)                       | 显示除源自 162.248.16.53 的数据包之外的所有数据包            |
| !(ipv6.dst ==2607:f8b0:400a:15::b)               | 显示除前往 IPv6 地址 2607:f8b0:400a:15::b 的数据包之外的所有数据包 |
| ip.addr == 192.168.4.1 && ip.addr == 192.168.4.2 | 显示 192.168.4.1 和 192.168.4.2                              |
| http.request                                     | 仅显示 http 请求 – 在故障排除或可视化网络流量时非常有用      |

## 参考链接：https://www.comptia.org/content/articles/what-is-wireshark-and-how-to-use-it