# Rtsp协议解读<no value>


## 0.1. SETUP

``` rtsp
Transport: RTP/AVP/UDP;unicast;client_port=25404-25405;mode=record
Transport: RTP/AVP/UDP;unicast;client_port=25404-25405;mode=record;server_port=30002-30003;ssrc=00000000
```

- transport-spec = transport-protocol/profile/lower-transport
	transport-protocol = "RTP"
	profile = "AVP"
	lower-transport = "TCP" | "UDP"
 
	- RTP/AVP : UDP承载的RTP
	- RTP/AVP/UDP : UDP承载的RTP
	- RTP/AVP/TCP : RTP Over TCP
	- RAW/RAW/UDP: UDP承载的原始数据

- unicast | multicast:  指示是单播还是多播，默认是多播
- mode:
	- PLAY： 播放，默认是播放
	- RECORD：录制
- append： 当mode是RECORD的时候，该参数标志应该追加保存录像而不是覆盖
- interleaved： 表示用什么协议控制流，一般只有RTP Over TCP才会用到。该种模RTP/RTSP/RTCP的数据交织要在一起。RTP/RTCP包前后被两个`$`包裹，后面紧跟1字节的channel标识符，再紧跟2字节的长度。如`interleaved=0-1`则表示channel值为0的为RTP包，channel值为1的为RTCP包
- ttl:   multicast time-to-live
- port: 多播时RTP和RTCP的端口，如`port=3456-3457` ，则RTP端口为3456，RTCP端口为3457
- client_port: 单播时客户端的RTP和RTCP的端口，如`port=3456-3457` ，则RTP端口为3456，RTCP端口为3457
- server_port: 单播时服务端RTP和RTCP的端口，如`port=3456-3457` ，则RTP端口为3456，RTCP端口为3457
- ssrc: 仅单播有用，和RTP中的SSRC比较，表示是否是同步源


## 0.2. RECORD

- Range: 指定时间范围，应该是半开区间，如a-b应当包含a，但不包含b
	- smpte
	- npt
	- clock
	- utc
 ```
Range: clock=19960213T143205Z-;time=19970123T143720Z
Range: npt=0.000-
``` 

- Scale : 倍速

- RTP-INFO:
	- url: 表示RTP对应的流URL
	- seq: 表示第一个数据包的序列号
	- rtptime: 表示事件值对应的RTP时间戳