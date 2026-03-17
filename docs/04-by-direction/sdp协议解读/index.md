# Sdp协议解读<no value>


	SDP（Session Description Protocol）全称是会话描述协议。主要用于两个会话实体之间的媒体协商。
-----
## 概述

标准SDP结构由一个**会话级描述**和多个**媒体级描述**组成，通常SDP中都会包含一个$音频媒体$描述和一
$视频媒体$描述。


每条描述信息都是 **\<type\> = \<value\>** 的形式("=" 两侧不允许有空格存在。)。其中，\<type\>是描述的目标，它由单个字符构成；\<value\>是对<\type\>的解释或约束。会话和媒体都有各自的\<type\>，另外还有一些公共的\<type\>。

- **会话级** 字段
``` sdp
v=(protocol version,协议版本)
o=(owener/creator and session identifier,会话的创建者)
s=(session name,会话名称)
t=(time the session is active,会话存活时间)
```

- \<network type\> 
	- IN: 表示互联网 


- **媒体级** 字段
``` sdp
m=(media,媒体)

```

- **公共** 字段
``` sdp
c=(connection information,网络信息)
a=(attribute,属性)
```
#### a=* (zero or more media attribute lines)

支持两种类型： 
- property attributes
``` sdp
a=<flag>

a=recvonl
```
- value attributes
``` sdp
a=<attribute>:<value

a=orient:landscape
```


一个典型的SDP格式为:

``` sdp
v=0 
o=- 5910110687297165449 2 IN IP4 127.0.0.1 
s=- t=0 0 
... 

//第一个媒体流，音频流 
m=audio 54797 UDP/TLS/RTP/SAVPF 8 
... 

//第2个媒体流，视频流 
m=video 9 UDP/TLS/RTP/SAVPF 125 
...
```

WEBRTC中 SDP 字段可以大致分为这几类：

![[Pasted image 20241011085344.png]]
## 会话描述
媒体信息是标准SDP的内容，其格式如下所示：

#### o=(owner/creator and session identifier)
 表示会话的创建者
``` sdp
o=<username> <session id> <version> <network type> <address type>
<address>

o=- 0 0 IN IP6 ::1
```

- \<network type\> 
	- IN: 表示互联网 
#### t=(time the session is active)

``` sdp
t=<start time> <stop time>

t=0 0
```
使用NTP时间表示形式(减去2208988800可转换成UNIX时间)



#### a=msid-semantic

``` sdp
a=msid-semantic:WMS lgsCFqt9kN2fVKw5wg3NKqGdATQoltEwOdMS 
```
WebRTC媒体流（WMS）在PeerConnection的生命周期中提供了一个唯一的标识符。这个标识符将被用于特定媒体流（在我们的例子中为音频和视频媒体流）的每个媒体描述 m= 下的的a=msid属性。这代表RTP媒体流（由每个RTP包中的SSRC字段识别）属于该媒体流，并且是该媒体流的一个轨道。  
它将RTP媒体流与 MediaStream WebRTC 对象进行关联。关于这一点的更多信息请参考草案draft-ietf-mmusic-msid

## 媒体描述
### 媒体信息
媒体信息是标准SDP的内容，属于SDP中的核心内容。其格式如下所示：

#### m=(media name and transport address)

``` sdp
m=<media> <port> <transport> <fmt list>
m=<media> <port>/<number of ports> <transport> <fmt list>>

m=video 0 RTP/AVP 96
m=video 49170/2 RTP/AVP 31
```

- \<media\> : media type
	- audio
		- video
	- application
	- data
	- control
 
- \<port\> : 0 或指定的传输端口。 
	- \<transport\>是 UDP,范围是\[1023,65535\]
	- \<transport\>是 RTP,应该是一个偶数端口，如果存在RTCP，则是RTP端口+1
 
- \<transport\>: 
	- UDP ：  User Datagram Protocol
	- RTP/AVP： the IETF's Realtime Transport Protocol using the
Audio/Video profile carried over UDP

- \<fmp list\> :  payload type，可以是静态的也可以是动态的，具体看RTP payload定义（96以上是动态的）
``` sdp
#静态的
m=video 49232 RTP/AVP 0

#动态的，对应描述由a中的rtpmap字段补充
m=audio 49230 RTP/AVP 96 97 98
a=rtpmap:96 L8/8000
a=rtpmap:97 L16/8000
a=rtpmap:98 L16/11025/2
```
这里有几个重要的属性说明一下：
#### a=ssrc

ssrc是媒体源的唯一标识，每一路媒体流都有唯一的ssrc来标识它。另外，在ssrc这一行中有时还能看到cname，这是就是该媒体流的别名。ssrc不同的媒体流可以有相同的cname，表示他们属于同一个媒体流（比如，重传流的cname就和原始流的相同）。
#### a=rtpmap
rtpmap是一张PayloadType与编码器的映射表。其格式如下所示：
``` sdp
a=rtpmap:<payload type> <encoding name>/<clock rate>[/<encodingparameters>]
```
- payload type：负载类型，与m line中的fmt中的对应。
- encoding name：端口，编码器名称，如VP8，VP9，OPUS等。
- clock rate：采样率。
- encodingparameters：编码参数，如音频的声道数信息。

#### a=fmtp

fmtp用于指定媒体数据格式。其格式如下所示：  

``` sdp
a=fmtp:<payload type> <format specific parameters>
```
- payload type：负载类型，与m line中的fmt中的对应。
- format specific parameters：参数信息，其含义需要参考相关草案。

#### - a=sendrecv
这一行表示浏览器愿意在这个会话中同时发送和接收音频。其他值可以是sendonly、recvonly和inactive，这些值用于实现不同的场景，如将电话挂起。
### 网络描述
网络描述也是标准SDP的内容。其中记录了传输媒体数据时使用的网络信息。

#### C=
``` sdp
c=IN IP4 217.130.243.155
```
c 代表了连接信息，它指明了媒体发送和接收的地址。
但是WebRTC中强制使用ICE，所以c行中的IP并不会被使用
#### a=candidate
描述了候选地址信息，是一个已经被废弃的属性，但因一些流媒体服务器上海有使用，所以WebRTC依然做了兼容

``` sdp
a=candidate:<candidate_name> <组件ID> <socket_type> <优先级> <ip> <port> typ <ip_type> [tcptype <tcptype> ] [genration 0] 


a=candidate:1467250027 1 udp 2122260223 192.168.0.196 46243 typ host generation 0
a=candidate:1467250027 2 udp 2122260222 192.168.0.196 56280 typ host generation 0
a=candidate:435653019 1 tcp 1845501695 192.168.0.196 0 typ host tcptype active generation 0
a=candidate:435653019 2 tcp 1845501695 192.168.0.196 0 typ host tcptype active generation 0
a=candidate:1853887674 1 udp 1518280447 47.61.61.61 36768 typ srflx raddr 192.168.0.196 rport 36768 generation 0
a=candidate:1853887674 2 udp 1518280447 47.61.61.61 36768 typ srflx raddr 192.168.0.196 rport 36768 generation 0
a=candidate:750991856 2 udp 25108222 237.30.30.30 51472 typ relay raddr 47.61.61.61 rport 54763 generation 0
a=candidate:750991856 1 udp 25108223 237.30.30.30 58779 typ relay raddr 47.61.61.61 rport 54761 generation 0
```
- candidate_name: 候选名称，自定义，字符串
- 组件ID： 1-rtp; 2-rtcp
- socket_type: 表示socket协议  
	- udp
	- tcp
-  优先级: 越大优先级越高
- ip_type
	- host:  从本机网卡获取到的IP地址类型， 即内部ip地址和端口
	- srvflx: server reflexive, STUN服务器获取到的出口IP地址类型，即出口NAT设备映射的公网IP地址和端口
	- prflx: peer reflexive, STUN服务器从打洞报文中获取到的内部IP地址类型，一般情况下和host类型是一致的
	- relay: TURN服务器提供的中继IP地址类型
		一般优先级 host > srflx > prflx > relay。
- tcptype:
	- passive 被动链接
	- active 主动链接
	- actpass 主被动都可以
#### b=\*(bandwidth information)

``` sdp
b=<modifier>:<bandwidth-value>

b=AS:200
```
- \<bandwidth-value\> : kbit/s 为单位
- \<modifier\>
	- CT: 表示整个会话，所有介质的总带宽
	- AS: 表示指定application，单个介质的带宽




常见字段
-  a=rtpmap
``` sdp
a=rtpmap:<payload type> <encoding name>/<clock rate>[/<encoding
parameters>]

a=rtpmap:96 MP4V-ES/90000
```

-  a=fmtp
``` sdp
a=fmtp:<format> <format specific parameters>

a=fmtp:96 profile-level-id=1; config=000001B001000001B58913000001000000012000C48D88019D0C04241443000001B24C61766336302E332E313030
```

- a=control 
可用于session或media，用于指定Setup的URL
```
a=control:streamid=0
```

### 安全描述 

这部分内容是WebRTC新增到SDP规范中去的。WebRTC的媒体通信提供了一套很好的安全机制。下面几个是其中比较重要的属性：

#### a=ice-ufrag和a=ice-pwd
``` sdp
a=ice-ufrag:Oyef7uvBlwafI3hT 
a=ice-pwd:T0teqPLNQQOf+5W+ls+P2p16
```

指明ICE的账号和密码，用于验证用户的合法性。

#### a=fingerprint

指明DTLS握手时的证书指纹，用于验证加密证书的有效性。

#### a=setup:(active/passive/actpass)

指明终端的角色，决定DTLS时是作为主动握手方还是被动连接方或是都可以。

### 服务质量

这部分内容也是WebRTC新增到SDP规范中去的。这个内容只控制WebRTC中很小一部分服务质量的开关。WebRTC中还包含有非常多的服务质量内容。只有以下一个属性：

#### a=rtcp-fb

在媒体协商过程中，发起方的SDP中指明要开启哪些RTCP反馈，而应答方则在回复的SDP中确认可以开启哪些RTCP反馈。例如：

- nack：丢包重传
- fir：申请关键帧
- transport-cc、goog-remb：拥塞控制算法。


### Plan B和Unified Plan

目前，WebRTC中的SDP包含两种规格，Plan B和Unified Plan。其中，Plan B是由标准SDP演化而来的，而Unified Plan是用来代替Plan B的新规格。两者会在表达传输多路同类媒体流的时候格式会有区别。

在Plan B中，只有两个媒体描述（m line），即音频媒体描述（m=audio...）和视频媒体描述（m=video...）。如果有多路同类媒体流则需要通过其媒体描述中的SSRC来区分。 而在Unified Plan中，每个媒体流都有一个媒体描述（m line）。
## WEBRTC SDP 示例
``` sdp
v=0
//sdp版本号，一直为0,rfc4566规定
o=- 7017624586836067756 2 IN IP4 127.0.0.1
//username，没有的话使用-代替，7017624586836067756是整个会话的编号，2表明会话版本
s=-
//会话名，没有的话使用-代替
t=0 0
//两个值分别是会话的起始时间和结束时间，这里都是0表明没有限制
a=group:BUNDLE audio video data
//须要共用一个传输通道传输的媒体，若是没有这一行，音视频，数据就会分别单独用一个udp端口来发送
a=msid-semantic: WMS h1aZ20mbQB0GSsq0YxLfJmiYWE9CBfGch97C
//WMS是WebRTC Media Stream简称，这一行定义了本客户端支持同时传输多个流，一个流能够包括多个
//track,通常定义了这个，后面a=ssrc这一行就会有msid,mslabel等属性

m=audio 9 UDP/TLS/RTP/SAVPF 111 103 104 9 0 8 106 105 13 126
//m=audio说明本会话包含音频，9表明音频使用端口9来传输，在webrtc中通常不使用，
//若是设置为0，表明不传输音频,UDP/TLS/RTP/SAVPF是表示用户来传输音频支持的协议，
//udp、tls、rtp表明使用udp来传输rtp包，并使用tls加密；
//SAVPF表明使用srtcp的反馈机制来控制通讯过程,
//111 103 104 9 0 8 106 105 13 126表示本会话音频支持的编码，后台几行会有详细补充说明

c=IN IP4 0.0.0.0
//这一行表示你要用来接收或者发送音频使用的IP地址，webrtc使用ice传输，不使用这个地址
a=rtcp:9 IN IP4 0.0.0.0
//用来传输rtcp地地址和端口，webrtc中不使用

a=ice-ufrag:khLS
a=ice-pwd:cxLzteJaJBou3DspNaPsJhlQ
//以上两行是ice协商过程当中的安全验证信息
a=fingerprint:sha-256 FA:14:42:3B:C7:97:1B:E8:AE:0C2:71:03:05:05:16:8F:B9:C7:98:E9:60:43:4B:5B:2C:28:EE:5C:8F3:17
//以上这行是dtls协商过程当中需要的证书指纹，用来验证DTLS证书的有效性

a=setup:actpass
//以上这行表明本客户端在dtls协商过程当中，能够作客户端也能够作服务端，参考rfc4145 rfc4572
a=mid:audio
//在前面BUNDLE这一行中用到的媒体标识
a=extmap:1 urn:ietf:params:rtp-hdrext:ssrc-audio-level
//上一行指出我要在rtp头部中加入音量信息，参考 rfc6464
a=sendrecv
//上一行指出我是双向通讯，另外几种类型是recvonly,sendonly,inactive
a=rtcp-mux
//上一行指出rtp,rtcp包使用同一个端口来传输
a=rtpmap:111 opus/48000/2
//a=rtpmap都是对m=audio这一行的媒体编码补充说明，指出了编码采用的编号，采样率，声道等
a=rtcp-fb:111 transport-cc
//以上这行说明opus编码支持使用rtcp来控制拥塞，参考https://tools.ietf.org/html/draft-holmer-rmcat-transport-wide-cc-extensions-01
a=fmtp:111 minptime=10;useinbandfec=1
//对opus编码可选的补充说明,minptime表明最小打包时长是10ms，useinbandfec=1表明使用opus编码内置fec特性
a=rtpmap:103 ISAC/16000
a=rtpmap:104 ISAC/32000
a=rtpmap:9 G722/8000
a=rtpmap:0 PCMU/8000
a=rtpmap:8 PCMA/8000
a=rtpmap:106 CN/32000
a=rtpmap:105 CN/16000
a=rtpmap:13 CN/8000
a=rtpmap:126 telephone-event/8000
a=ssrc:18509423 cname:sTjtznXLCNH7nbRw
//cname用来标识一个数据源，ssrc当发生冲突时可能会发生变化，可是cname不会发生变化，也会出现在rtcp包中SDEC中，
//用于音视频同步
a=ssrc:18509423 msid:h1aZ20mbQB0GSsq0YxLfJmiYWE9CBfGch97C 15598a91-caf9-4fff-a28f-3082310b2b7a
//以上这一行定义了ssrc和WebRTC中的MediaStream,AudioTrack之间的关系，msid后面第一个属性是stream-d,第二个是track-id
a=ssrc:18509423 mslabel:h1aZ20mbQB0GSsq0YxLfJmiYWE9CBfGch97C
a=ssrc:18509423 label:15598a91-caf9-4fff-a28f-3082310b2b7a

m=video 9 UDP/TLS/RTP/SAVPF 100 101 107 116 117 96 97 99 98
//参考上面m=audio,含义相似

c=IN IP4 0.0.0.0
a=rtcp:9 IN IP4 0.0.0.0

a=ice-ufrag:khLS
a=ice-pwd:cxLzteJaJBou3DspNaPsJhlQ
a=fingerprint:sha-256 FA:14:42:3B:C7:97:1B:E8:AE:0C2:71:03:05:05:16:8F:B9:C7:98:E9:60:43:4B:5B:2C:28:EE:5C:8F3:17

a=setup:actpass
a=mid:video
a=extmap:2 urn:ietf:params:rtp-hdrext:toffset
a=extmap:3 http://www.webrtc.org/experiments/rtp-hdrext/abs-send-time
a=extmap:4 urn:3gpp:video-orientation
a=extmap:5 http://www.ietf.org/id/draft-hol ... de-cc-extensions-01
a=extmap:6 http://www.webrtc.org/experiments/rtp-hdrext/playout-delay
a=sendrecv

a=rtcp-mux
a=rtcp-rsize
a=rtpmap:100 VP8/90000
a=rtcp-fb:100 ccm fir
//ccm是codec control using RTCP feedback message简称，意思是支持使用rtcp反馈机制来实现编码控制，
//fir是Full Intra Request简称，意思是接收方通知发送方发送帧过来
a=rtcp-fb:100 nack
//支持丢包重传，参考rfc4585
a=rtcp-fb:100 nack pli
//支持关键帧丢包重传,参考rfc4585
a=rtcp-fb:100 goog-remb
//支持使用rtcp包来控制发送方的码流
a=rtcp-fb:100 transport-cc
//参考上面opus
a=rtpmap:101 VP9/90000
a=rtcp-fb:101 ccm fir
a=rtcp-fb:101 nack
a=rtcp-fb:101 nack pli
a=rtcp-fb:101 goog-remb
a=rtcp-fb:101 transport-cc
a=rtpmap:107 H264/90000
a=rtcp-fb:107 ccm fir
a=rtcp-fb:107 nack
a=rtcp-fb:107 nack pli
a=rtcp-fb:107 goog-remb
a=rtcp-fb:107 transport-cc
a=fmtp:107 level-asymmetry-allowed=1;packetization-mode=1;profile-level-id=42e01f
//h264编码可选的附加说明
a=rtpmap:116 red/90000
//fec冗余编码，通常若是sdp中有这一行的话，rtp头部负载类型就是116，不然就是各编码原生负责类型
a=rtpmap:117 ulpfec/90000
//支持ULP FEC，参考rfc5109
a=rtpmap:96 rtx/90000
a=fmtp:96 apt=100
//以上两行是VP8编码的重传包rtp类型
a=rtpmap:97 rtx/90000
a=fmtp:97 apt=101
a=rtpmap:99 rtx/90000
a=fmtp:99 apt=107
a=rtpmap:98 rtx/90000
a=fmtp:98 apt=116
a=ssrc-group:FID 3463951252 1461041037
//在webrtc中，重传包和正常包ssrc是不一样的，上一行中前一个是正常rtp包的ssrc,后一个是重传包的ssrc
a=ssrc:3463951252 cname:sTjtznXLCNH7nbRw
a=ssrc:3463951252 msid:h1aZ20mbQB0GSsq0YxLfJmiYWE9CBfGch97C ead4b4e9-b650-4ed5-86f8-6f5f5806346d
a=ssrc:3463951252 mslabel:h1aZ20mbQB0GSsq0YxLfJmiYWE9CBfGch97C
a=ssrc:3463951252 label:ead4b4e9-b650-4ed5-86f8-6f5f5806346d
a=ssrc:1461041037 cname:sTjtznXLCNH7nbRw
a=ssrc:1461041037 msid:h1aZ20mbQB0GSsq0YxLfJmiYWE9CBfGch97C ead4b4e9-b650-4ed5-86f8-6f5f5806346d
a=ssrc:1461041037 mslabel:h1aZ20mbQB0GSsq0YxLfJmiYWE9CBfGch97C
a=ssrc:1461041037 label:ead4b4e9-b650-4ed5-86f8-6f5f5806346d

m=application 9 DTLS/SCTP 5000

c=IN IP4 0.0.0.0

a=ice-ufrag:khLS
a=ice-pwd:cxLzteJaJBou3DspNaPsJhlQ
a=fingerprint:sha-256 FA:14:42:3B:C7:97:1B:E8:AE:0C2:71:03:05:05:16:8F:B9:C7:98:E9:60:43:4B:5B:2C:28:EE:5C:8F3:17

a=setup:actpass
a=mid:data
a=sctpmap:5000 webrtc-datachannel 1024
```