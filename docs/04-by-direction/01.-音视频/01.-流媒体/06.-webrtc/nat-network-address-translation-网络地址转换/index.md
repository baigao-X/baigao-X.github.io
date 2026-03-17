# Nat (Network Address Translation) 网络地址转换<no value>

[wiki](https://en.wikipedia.org/wiki/Network_address_translation)

---

RFC 1631

# 作用
- 最初用于绕过在网络 移动或者更换上游互联网服务提供商时为每个主机分配地址的需要。
- 解决IPV4地址不够的“短期解决方案”

# NAT类型

RFC 3489-2003-NAT上的简单遍历协议 已经废弃
将 NAT实现分类为：
- 完全圆锥型NAT "Full Cone" NAT
	- 一旦内部地址 (iAddr:iPort) 映射到外部地址 (eAddr:ePort)，来自 iAddr:iPort 的任何数据包都将通过 eAddr:ePort 发送。
	- 任何外部主机都_可以通过将数据包发送到 eAddr:ePort 来将数据包发送到 iAddr:iPort。
	 举例: 
		 1.A先通过私有IP端口$10.0.0.1:4321$ 发向公共服务器 $18.181.0.31:1234$,NAT选择的映射A的公网IP和端口为 $155.99.25.11:62000$后
		 2.且所有外部主机发送的数据包都能通过$155.99.25.11:62000$ 到达A的内部IP端口$10.0.0.1:4321$
		 3.A再通过私有IP端口 $10.0.0.1:4321$ 发向B, NAT选择的映射A的公网IP和端口依然为 $155.99.25.11:62000$
- 地址受限圆锥型NAT 
	- 一旦内部地址 (iAddr:iPort) 映射到外部地址 (eAddr:ePort)，来自 iAddr:iPort 的任何数据包都将通过 eAddr:ePort 发送。
	- 仅当 iAddr:iPort 先前已将数据包发送到 hAddr :any_时，外部主机 ( hAddr:any ) 才可以通过将数据包发送到 eAddr:ePort 来将数据包发送到 iAddr: _iPort_。“任意”意味着端口号无关紧要。
	 举例: 
		 1.A先通过私有IP端口$10.0.0.1:4321$ 发向公共服务器 $18.181.0.31:1234$,NAT选择的映射A的公网IP和端口为 $155.99.25.11:62000$后
		 2.且所有来自IP$18.181.0.31(任何端口)$的数据包都能通过$155.99.25.11:62000$ 到达A的内部IP端口$10.0.0.1:4321$
		 3.A再通过私有IP端口 $10.0.0.1:4321$ 发向B, NAT选择的映射A的公网IP和端口依然为 $155.99.25.11:62000$
- 端口受限锥体NAT
	- 一旦内部地址 (iAddr:iPort) 映射到外部地址 (eAddr:ePort)，来自 iAddr:iPort 的任何数据包都将通过 eAddr:ePort 发送。
	- 仅当 iAddr:iPort 先前已将数据包发送到 hAddr:hPort 时，外部主机 ( hAddr:hPort ) 才可以通过将数据包发送到 eAddr:ePort 来将数据包发送到 iAddr:iPort。
	 举例: 
		 1.A先通过私有IP端口$10.0.0.1:4321$ 发向公共服务器 $18.181.0.31:1234$,NAT选择的映射A的公网IP和端口为 $155.99.25.11:62000$后
		 2.只有来自$18.181.0.31:1234$ 的数据包都能通过$155.99.25.11:62000$ 到达A的内部IP端口$10.0.0.1:4321$
		 3.A再通过私有IP端口 $10.0.0.1:4321$ 发向B, NAT选择的映射A的公网IP和端口依然为 $155.99.25.11:62000$
- 对称NAT
	- 一个内部 IP 地址加上目标 IP 地址和端口的组合映射到单个唯一的外部源 IP 地址和端口；如果同一内部主机发送具有相同源地址和端口但目的地不同的数据包，则将使用不同的映射。
	- 只有从内部主机接收到数据包的外部主机才能发回数据包。
	 举例: 
		 1.A先通过私有IP端口$10.0.0.1:4321$ 发向公共服务器 $18.181.0.31:1234$,NAT选择的映射A的公网IP和端口为 $155.99.25.11:62000$后
		 2.只有来自$18.181.0.31:1234$ 的数据包都能通过$155.99.25.11:62000$ 到达A的内部IP端口$10.0.0.1:4321$
		 3.A再通过私有IP端口 $10.0.0.1:4321$ 发向B, NAT选择的映射A的公网IP和端口不再为 $155.99.25.11:62000$，而可能为$155.99.25.11:62001$

实际NAT实现可能参考并结合以上模型，并非完全对应

#  NAT 穿透 (NAT traversal)
NAT 设备没有自动方法来确定来自外部网络的传入数据包的目的地内部主机,使得内部网络不适合托管服务。
比如[点对点](https://en.wikipedia.org/wiki/Peer-to-peer "点对点")文件共享、[VoIP](https://en.wikipedia.org/wiki/VoIP "网络电话")服务和[视频游戏控制台](https://en.wikipedia.org/wiki/Video_game_console "视频游戏机")等应用程序也要求客户端同时是服务器。传入请求无法轻松关联到正确的内部主机。
此外，许多此类服务在应用程序数据中携带 IP 地址和端口号信息，可能需要用[深度数据包检查](https://en.wikipedia.org/wiki/Deep_packet_inspection "深度包检测")进行替换
网络地址转换技术尚未标准化。因此，用于 NAT 穿越的方法通常是专有的并且记录很少。许多遍历技术需要伪装网络之外的[服务器的帮助。](https://en.wikipedia.org/wiki/Server_(computing) "服务器（计算）")
有些方法仅在建立连接时使用服务器，而其他方法则基于通过服务器中继所有数据，这增加了带宽要求和延迟，不利于实时语音和视频通信。
最近[对称 NAT](https://en.wikipedia.org/wiki/Symmetric_NAT "对称NAT")的激增降低了许多实际情况下的 NAT 穿越成功率，例如移动和公共 WiFi 连接。打洞技术（例如 STUN 和 ICE）在没有中继服务器帮助的情况下无法穿越对称 NAT.
TURN使用了尝试预测每个 NAT 设备要打开的下一个端口来遍历对称 NAT，然而端口预测技术仅对使用已知确定性算法进行端口选择的 NAT 设备有效。这种可预测但非静态的端口分配方案在大规模 NAT（例如[4G LTE](https://en.wikipedia.org/wiki/4G_LTE "4G LTE")网络中使用的 NAT）中并不常见，因此端口预测在这些移动宽带网络上基本上无效。

## 实现NAT穿透的方法

当不同 NAT 后面的对等点尝试通信时，就会出现NAT[穿越问题。](https://en.wikipedia.org/wiki/NAT_traversal "NAT穿越")解决这个问题的一种方法是使用[端口转发](https://en.wikipedia.org/wiki/Port_forwarding "转发端口")。另一种方法是使用各种NAT穿越技术。TCP NAT 穿越最流行的技术是[TCP 打洞](https://en.wikipedia.org/wiki/TCP_hole_punching "TCP打孔")技术。
- [NAT 端口映射协议](https://en.wikipedia.org/wiki/NAT_Port_Mapping_Protocol)(NAT-PMP) 是 Apple 推出的一种协议，作为 IGDP 的替代方案。
- [端口控制协议](https://en.wikipedia.org/wiki/Port_Control_Protocol "端口控制协议")(PCP) 是 NAT-PMP 的后继者。
- [](https://en.wikipedia.org/wiki/UPnP "通用即插即用")[家庭或小型办公室](https://en.wikipedia.org/wiki/Small_office/home_office "Small office/home office")设置中的许多小型 NAT 网关都支持[UPnP](https://en.wikipedia.org/wiki/UPnP "UPnP") [Internet 网关设备协议](https://en.wikipedia.org/wiki/Internet_Gateway_Device_Protocol "互联网网关设备协议")(UPnP IGD) 。它允许网络上的设备请求路由器打开端口。[](https://en.wikipedia.org/wiki/Small_office/home_office "小型办公室/家庭办公室")
- [交互式连接建立](https://en.wikipedia.org/wiki/Interactive_Connectivity_Establishment "互动连接建立")(ICE) 是一个完整的协议，用于使用 STUN 和/或 TURN 进行 NAT 遍历，同时选择可用的最佳网络路由。它填补了 STUN 规范中未提及的一些缺失部分和缺陷。
- [NAT 会话遍历实用程序](https://en.wikipedia.org/wiki/STUN "眩晕")(STUN) 是用于 NAT 打洞的一组标准化方法和网络协议。它是为 UDP 设计的，但也扩展到 TCP。
- [Traversal using Relays around NAT](https://en.wikipedia.org/wiki/Traversal_Using_Relays_around_NAT "使用 NAT 周围的中继进行遍历")（TURN）是专门为 NAT 穿越而设计的中继协议。
- [NAT 打洞](https://en.wikipedia.org/wiki/NAT_hole_punching "NAT打洞")是一种通用技术，它利用 NAT 处理某些协议（例如 UDP、TCP 或 ICMP）的方式来允许先前阻止的数据包通过 NAT。
    - [UDP打洞](https://en.wikipedia.org/wiki/UDP_hole_punching "UDP打洞")
    - [TCP打洞](https://en.wikipedia.org/wiki/TCP_hole_punching "TCP打洞")
    - [ICMP 打洞](https://en.wikipedia.org/wiki/ICMP_hole_punching "ICMP 打洞")
- [套接字安全](https://en.wikipedia.org/wiki/SOCKS "袜子")(SOCKS) 是一项创建于 20 世纪 90 年代初的技术，它使用代理服务器在网络或系统之间中继流量。
- [应用程序级网关](https://en.wikipedia.org/wiki/Application-level_gateway "应用级网关")(ALG) 技术是防火墙或 NAT 的一个组件，提供可配置的 NAT 遍历过滤器。[[2]](https://en.wikipedia.org/wiki/NAT_traversal#cite_note-techniquesHu-2) 据称，该技术产生的问题多于它解决的问题。[[3]](https://en.wikipedia.org/wiki/NAT_traversal#cite_note-3)

## NAT 打洞
为了打洞，每个客户端连接到一个不受限制的第三方服务器，该服务器临时存储每个客户端的外部和内部[地址](https://en.wikipedia.org/wiki/Network_address "Network address")以及[端口信息。](https://en.wikipedia.org/wiki/Port_(computer_networking) "Port (computer networking)")然后，服务器将每个客户端的信息转发给另一个客户端，并使用该信息，每个客户端尝试建立直接连接；由于使用有效端口号的连接，限制性防火墙或路由器接受并转发每一侧的传入数据包。

[跨网络地址转换器的对等通信](https://bford.info/pub/net/p2pnat/)

## P2P友好型NAT的要素

#####  一致的端点映射
简单解释: NAT对来源于一个同一个私有IP和端口，发向不同的对端IP和端口时，NAT选择映射为同一个公网IP和端口
如:
A先通过私有IP端口$10.0.0.1:4321$ 发向公共服务器 $18.181.0.31:1234$,NAT选择的映射A的公网IP和端口为 $155.99.25.11:62000$后
A再通过私有IP端口 $10.0.0.1:4321$ 发向B, NAT选择的映射A的公网IP和端口依然为 $155.99.25.11:62000$
即所谓的**锥形NAT**而不是**对称型NAT** （对称型NAT无法穿透）

##### 处理未经请求的 TCP 连接
当NAT接收到一个SYN数据包，发现该数据包是未经请求的传入链接尝试时，默默地丢弃该SYN数据包更友好，否则回传TCP RST 或者ICMP错误报告会干绕打洞流程。

##### 不干扰有效载荷
有些NAT会自作主张地扫描UDP或TCP报文正文，并判断其中的IPV4地址数据，将其转换为映射的公网IP地址。这会影响公共服务器获取到客户端的私有IP和端口
因为穿透报文会对报文中的IP地址做下混淆或反码处理

##### 发夹转发
发夹转发即当NAT设备识别到接受的IP再NAT内部，会向内部转发。
不支持发夹转发，即丢弃这些报文
当涉及多级NAT时，如果NAT设备不支持发夹转发时，则会导致对端永远无法收到这些报文


## NAT穿透的标准协议
#### STUN（Session Traversal Utilities for NAT）（RFC 5389）

需要一个已知的第三方 STUN 服务器支持，该服务器必须架设在公网上

RFC 5389 于 2008 年重新标准化-NAT 的会话遍历实用程序




#### TURN（Traversal  Using Relays around NAT）协议（RFC 5766）

引入一个中继服务器，做转发，此时就不再是端对端了

#### ICE（Interactive Connectivity  Establishment）交互式链接建立协议（RFC 5245）

用于协商，能STUN就STUN，不能STUN，再TURN






