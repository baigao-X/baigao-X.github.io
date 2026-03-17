# Webrtc 协议解读<no value>


1.先基于HTTP 协商端口和协议

再基于协商的协议(TCP和UDP均可)和端口基于ICE协议打洞，再发送RTP/SRTP包。 


![[Pasted image 20240819092039.png]]

## 获取本地音视频.
### getUserMedia.

获取本地音视频流

## 服务模块：

### Signaling Server: 信令服务 
两个浏览器之间交换SDP的服务
	- HTTP
	- Websocket
	- SIP
	- WHIP/WHEP协议： 还在草案阶段，未形成RFC文档
		- [whip](https://datatracker.ietf.org/doc/draft-ietf-wish-whip/)
		- [whep](https://datatracker.ietf.org/doc/draft-ietf-wish-whep/)

### TURN Server: 转发服务
	转发服务，帮助两个浏览器之间转发媒体数据的服务。这是一种透明转发服务，并不会实现数据缓存，因此当多人会议时，浏览器之间需要传输`N*N + N*(N-2)`份数据。一般只应用在非常少的通信场景中，比如一对一
### SFU Server 选择性转发服务
	服务器上有缓存数据，因此浏览器只需要上传一份数据，服务器会复制给其他参会者。
- Forward: 转发
- Simulcast: 按照带宽来实现不同端的传输码率不同
	- 不需要转码，降码率的实际操作是让推流端提供多个不同比特率的媒体源，并支持切换

### MCU Server 多点控制服务
	多点控制服务，服务器将会议中的流合并成一路，这样浏览器只需要传输`N*2`份数据，上传一路下载一路数据。

## 实时网络传输.

### 涉及协议.

- ICE，即 Interactive Connectivity Establishment（ RFC8845 + RFC 8863）
	- SDP 部分描述在(RFC 8839）
	- STUN，即 Session Traversal Utilities for NAT（ RFC 8489）  
	- TURN，即 Traversal Using Relays around NAT（ RFC 8656）  
- SDP，即 Session Description Protocol（ RFC 4566）  
	- 关于ICE部分在 (RFC-8845),(RFC 8839)
- DTLS，即 Datagram Transport Layer Security（ RFC 9147）  
- SCTP，即 Stream Control Transport Protocol（ RFC 9260）  
- SRTP，即 Secure Real-Time Transport Protocol（ RFC 3711）
	- SRTP**加密所有 RTP 数据包的媒体负载**。请注意，**只有有效负载受到保护，RTP 标头不受保护**。

### RTCPeerConnection.
	负责维护每一个端到端连接的完整生命周期.
	 管理穿越NAT的完整ICE工作流

![[Pasted image 20240819092249.png]]

#### 发信号和协商会话.
	 该动作的目的是为了避免对端没有监听信道，无法响应。因此需要先通过上层应用协议，交互，建立协商会话。
	可以选择使用以下几种协议获取他自定义网关，webrtc并未规定具体实现

• SIP（Session Initiation Protocol，会话初始协议）  
应用级发信协议，广泛用于通过 IP 实现的语音通话（VoIP）和视频会议。  
• Jingle  
XMPP 协议的发信扩展，用于通过 IP 实现的语音通话（VoIP）和视频会议的会  
话控制。  
• ISUP（ISDN User Part， ISDN用户部分）  
全球各大公共电话交换网中用于启动电话呼叫的发信协议

- Asterisk 就是这么一个 流行、免费、开源的框架，被全球很多私人公司和大型运营商采用。Asterisk 有一个 WebSocket 模块，该模块支持将 SIP 作为发信协议：浏览器建立到 Asterisk 网关的 WebSocket 连接，然后两者通过交换 SIP 消息来协商会话！

建立发信通道后，就可以使用SDP协议进行交互码流信息
SDP 不包含媒体本身的任何信息，仅用于描述“会话状况”，表现为一系 列的连接属性：要交换的媒体类型（音频、视频及应用数据）、网络传输协议、使用的编解码器及其设置、带宽及其他元数据。
关于SDP字段的说明见[[00. SDP协议解读]]


#### 交互连接建立(ICE).
每个 RTCPeerConnection 连接对象都包含一个“ICE 代理”
该步骤是为了建立端到端的连接，让流数据减少延时。

(1) ICE 代理向操作系统查询本地 IP 地址；  
(2) 如果有配置， ICE 代理会查询外部 STUN 服务器，以取得本地端的公共 IP 和端  
口号；  
(3) 如果有配置， ICE 代理会将 TURN 服务器追加为最后一个候选项；假如端到端的  
连接失败，数据将通过指定的中间设备转发。


chrome://webrtc-internals: chrome 浏览器对webrtc会话的状态监测数据

#### 交付媒体和应用数据.

- DTLS: 协商加密密钥，后续的SRTP和SCTP因为不具备协商密钥对的能力，所以直接复用DTLS协商的密钥。
- SRTP: 用于传输音频和视频流
- SCTP: 用于传输应用数据： 实际上裸SCTP就能够完全支持特性（也就是说UDP这层完全是多余的），只不过受限于现有路由设备和NAT设备不能正确处理SCTP，所以必须承载与UDP进行传输
为解决SRTP、SCTP需要占用不同端口的问题，WebRTC使用多路复用扩展，使所有流能通过一个端口通信

### DataChannel API.
用于端到端传输任意应用数据（经过SCTP）

## 拥塞算法

#### TWCC
TWCC全称是Transport wide Congestion Control
原理是在**接收端**保存数据包状态，然后构造RTCP包反馈给发送端，反馈信息包括包到达时间、丢包状态等；在**发送端进行带宽估计**，进行拥塞控制。

#### REMB
REMB（Receiver Estimated Maximum Bitrate）是一种带宽估计算法
具体来说，REMB算法的基本原理如下：
接收端监测缓存：接收端会定期监测自己的视频缓存情况，包括缓存的大小、缓存时间等指标。
发送端发送带宽估计值：当缓存情况较好时，接收端会向发送端发送一个带宽估计值，告诉发送端当前的可用带宽。
发送端根据估计值调整码率和分辨率：发送端会根据接收端发来的带宽估计值，适当地调整视频的码率和分辨率，以适应当前的网络带宽。
