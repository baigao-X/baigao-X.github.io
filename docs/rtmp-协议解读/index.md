# Rtmp 协议解读<no value>

Real Time Message Protocol(实时信息传输协议)
默认TCP::1935
一个会话通过一条socket通道通信。
RTSP 将数据格式化为RTMP Message，实际传输时会将Message划分为带MessageID的chunk发送。


# 1. 时序图

``` mermaid

sequenceDiagram

Client ->> Server: C0,C1
Server ->> Client: S0,S1,S2
Client ->> Server: C2

Client ->> Server: connect
Server ->> Client: NetConnection.Connect.Success

Client ->> Server: createStream
Server ->> Client: createSream response

Client ->> Server: play
Server ->> Client: SetChunkSize
Server ->> Client: StreamBegin
Server ->> Client: NetStream.Play.Start
Server ->> Client: Audio or Video Data
Server -) Client: NetStream.Play.Stop
```

协议规定播放流媒体的两个前提步骤：
- 第一步： 建立**网络连接(NetConnection)** ,代表服务端与客户端之间的连通状态。服务端与客户端之间只会建立一个网络连接
- 第二步： 建立**网络流(NetStream)**，代表发送多媒体数据的通道，可以基于网络连接创建很多网络流。


# 2. 数据包结构

## 2.1. 握手handshaking

客户端通过发送 C0 和 C1 消息来启动握手过程。
客户端必须接收到 S1 消息，然后发送 C2 消息。
客户端必须接收到S2 消息，然后发送其他数据。
服务端必须接收到 C0 或者 C1 消息，然后发送 S0 和 S1 消息。
服务端必须接收到 C2消息，然后发送其他数据。

在实际工程应用一般是:
	客户端将C0、C1块同时发出。
	服务器在收到C1块之后同时将S0、S1、S2发给客户端。
	客户端收到S1之后，发送C2给服务端，握手完成


### 2.1.1. simple handshake (简单握手)
#### 2.1.1.1. c0,s0 
1bytes
```
  0 1 2 3 4 5 6 7
 +-+-+-+-+-+-+-+-+
 |   version     |
 +-+-+-+-+-+-+-+-+
```

- version（1 byte）：版本。在 C0 包内，这个字段代表客户端请求的 RTMP 版本号。在 S0 包内，这个字段代表服务端选择的 RTMP 版本号。当前使用的版本是 3。版本 0-2 用在早期的产品中，如今已经弃用；版本 4-31 被预留用于后续产品；版本 32-255（为了区分 RTMP 协议和文本协议，文本协议通常是可以打印字符）不允许使用。如果服务器无法识别客户端的版本号，应该回复版本 3,。客户端可以选择降低到版本 3，或者终止握手过程。
#### 2.1.1.2. c1,s1

1536 bytes
``` 
    0                   1                   2                   3  
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
   |                           time (4 bytes)                      |  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
   |                           zero (4 bytes)                      |  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
   |                           random  bytes (1528bytes)           |
   |                             ....                              |  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- time（4 bytes）：本字段包含一个时间戳，客户端应该使用此字段来标识所有流块的时刻。时间戳取值可以为零或其他任意值。为了同步多个块流，客户端可能希望多个块流使用相同的时间戳。
- zero（4 bytes）：本字段必须为零。
- random （1528 bytes）：本字段可以包含任意数据。由于握手的双方需要区分另一端，此字段填充的数据必须足够随机（以防止与其他握手端混淆，用户区分出其响应C2/S2来自此RTMP连接发起的握手还是其他方发起的握手）。不过没有必要为此使用加密数据或动态数据。
#### 2.1.1.3. c2,s2
1536 bytes
做为C1和S1的回应。

``` 
    0                   1                   2                   3  
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
   |                           time (4 bytes)                      |  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
   |                           time2 (4 bytes)                     |  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
   |                           random  bytes (1528bytes)           |
   |                             ....                              |  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- time（4 bytes）： 本字段必须包含对端发送的时间戳。
- time2（4 bytes）：本字段必须包含时间戳，取值为接收对端发送过来的握手包的时刻。
- random（1528 bytes）：本字段必须包含对端发送过来的随机数据。握手的双方可以使用时间 1 和时间 2 字段来估算网络连接的带宽和/或延迟，但是不一定有用。 


### 2.1.2. complex handshake (复杂握手)
#### 2.1.2.1. c0,s0 
1bytes
```
  0 1 2 3 4 5 6 7
 +-+-+-+-+-+-+-+-+
 |   version     |
 +-+-+-+-+-+-+-+-+
```

- version（1 byte）：版本。在 C0 包内，这个字段代表客户端请求的 RTMP 版本号。在 S0 包内，这个字段代表服务端选择的 RTMP 版本号。当前使用的版本是 3。版本 0-2 用在早期的产品中，如今已经弃用；版本 4-31 被预留用于后续产品；版本 32-255（为了区分 RTMP 协议和文本协议，文本协议通常是可以打印字符）不允许使用。如果服务器无法识别客户端的版本号，应该回复版本 3,。客户端可以选择降低到版本 3，或者终止握手过程。
#### 2.1.2.2. c1,s1

1536 bytes
``` 
    0                   1                   2                   3  
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
   |                           time (4 bytes)                      |  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
   |                           version (4 bytes)                   |  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
   |                           key (764 bytes)                     |  
   |                             ....                              |  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
   |                           digest bytes (764bytes)             |
   |                             ....                              |  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
\* *key 和 digest 的先后顺序是不确定的*

- time（4 bytes）：本字段包含一个时间戳，客户端应该使用此字段来标识所有流块的时刻。时间戳取值可以为零或其他任意值。为了同步多个块流，客户端可能希望多个块流使用相同的时间戳。
- version（4 bytes）：本字段必须为零。
- key (764 bytes）: 
	- random-data: (offset) bytes
	- key-data: 128 bytes
	- random-data: (764 - offset - 128 -4) bytes
	- offset : 4 bytes
	
- digest (764 bytes）: 
	- offset : 4 bytes
	- random-data: (offset) bytes
	- digest-data: 32 bytes
	- random-data: (764 - offset - 32) bytes
#### 2.1.2.3. c2,s2
1536 bytes
#### 2.1.2.4. c2,s2
1536 bytes
做为C1和S1的回应。

``` 
    0                   1                   2                   3  
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
   |                           random-data (1504 bytes)            |  
   |                             ....                              |  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
   |                           digest-data (32 bytes)              |  
   |                             ....                              |  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
```

- random-data（1504 bytes）： 
- digest-data（32bytes）：



## 2.2. Chunk Message

传输层封包，数据流切片的最小单位。

### 2.2.1. 包结构

  ```
   +-------------+----------------+------------------+------------+  
   |Basic Header | Message Header | Extend Timestamp | Chunk Data |  
   +-------------+----------------+------------------+------------+  
   |
   |<--------------Chunk Header--------------------->|
```


#### 2.4.1. Basic Header 块基本头 (1-3 bytes)
可变字节，长度取决于块流ID。

- Chunk Type 块类型(fmt) 2 bit： 
- Chunk Stream ID(CSID, 流通道ID): 用一个唯一标识一个特定的流通道 . ChunkStreamID 用于块流中组包消息。即一个消息拆分chunk后，通过ChunkStreamID组装回来。

- 1字节时
	CSID 范围 3 -63 (0,1,2是协议保留)
 
```
    0               
    0 1 2 3 4 5 6 7 
   +-+-+-+-+-+-+-+-+
   |fmt|   CSID    |
   +-+-+-+-+-+-+-+-+
```

- 2字节时
	CSID 范围 64 -319 ,  CSID = 第2个字节 + 64

```
    0               1               
    0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |fmt|      0    |   CSID - 64   |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- 3字节时
	CSID 范围 64 -65599 ,  CSID = 第3个字节*256 + 第2个字节 + 64

```
    0               1               2                 
    0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7    
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
   |fmt|     1     |       CSID - 64               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```


| Chunk Stream 名称 | CSID |  作用 |
| ---- | ---- | ---- | 
| RTMP_CONTROL_CHANNEL | 0x02 | 控制流通道|
| RTMP_COMMAND_CHANNEL | 0x03 | 命令信道|
| RTMP_STREAM_CHANNEL  | 0x05 | 数据流通道|
| RTMP_VIDEO_CHANNEL   | 0x06 | 视频信道|
| RTMP_AUDIO_CHANNEL   | 0x07 | 音频信道|


#### 2.4.2. Message Header消息头 (0,3,7,11 bytes) 
不定长，字节长度由**块基本头**中的 **块类型 fmt** 决定。
包含了要发送的实际内容的描述信息

##### fmt 为0 时， 11bytes
其他三种能表示的数据它都能表示.
但在**chunk stream 的开始第一个chunk**和**头信息中的时间戳**后退（即值与上一个chunk相比减小，通常在回退播放的时候会出现这种情况）的时候必须采用这种格式。


```
    0               1               2               3  
    0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |timestamp                                      | message length|
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  message length(continue)     |message type id| msg stream id |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |   msg stream id(continue)                     |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- timestamp 时间戳： 3 bytes, 最多表示16777215=0xFFFFFF=2^24-1 ，当超过最大值时，三个字节均置为为1，并将实际时间戳转到 Extend Timestamp

- message length 消息数据长度，占3字节。 表示实际发送的数据如音视频帧数据的长度，单位为字节。

- message type id 消息的类型id, 占1字节。表示实际发送的数据的类型。如

- message stream id (消息的流ID) 4 个字节ID，表示该chunk所在的流ID，小端存储
	表明该次媒体流的唯一标识，用于区分不同流。如ID为0的stream来表示客户端和服务端的连接和控制，ID为1的stream来完成stream的控制和播放
	
- Extend Timestamp (0,4 bytes)
	当chunk中的timestamp 或者 timestamp delta 超过取值范围时，采用该值，。。
	扩展时间戳占 4 个字节，能表示的最大数值就是 0xFFFFFFFF ＝ 4294967295。当扩展时间戳启用时，timestamp字段或者timestamp delta要全置为1，而不是减去时间戳或者时间戳差的值。


PS: obs推流、ffmpeg推流都有自己的定义方式，与这个略有不同

| Message Type | Msg Type id | MessageStreamID | 作用 |
| ---- | ---- | ---- | ---- |
| Set Chunk Size | 1 | 0 | 通知对端，更新最大可接受的Chunk大小，默认为128 (单位: Bytes) ，最小为1 (单位: Bytes) |
| About Message | 2 |  | 通知另一方，终止处理指定cs_ id信道后续的其他消息 |
| Acknowledgement | 3 |  | 由数据接收方(receiver) 发送，当首次收到有效数据大小等于Window Ack Size消息设置的窗口大小时，发送此消息给数据发送方(sender)以表示链接稳定。 |
| User Control Message | 4 | 0 | 用户控制消息，是-系列用于控制消息流的消息的总称。用来作为控制对端用户操作事件的一种手段 |
| Window Ack Size | 5 | 0 | 由数据发送方(sender) 发送，用来设定接收方(receiver)首次有效数据传输到来之后，用于等待确定传输稳定的窗口大小(单位: Bytes) |
| Set Peer Bandwidth | 6 | 0 | 由数据接收方(receiver) 发送，根据已收到但未确认的消息的数据量，来通知约束发送方(sender) 的输出带宽(单位: Bytes)，收到消息需要发送Window AckSize做应答 |
| Audio | 8 | 1 | RTMP音频数据包 |
| Video | 9 | 1 | RTMP视频数据包 |
| Data AMF3 | 15 |  | AMF3编码，音视频MetaData风格\|配置 详情包 |
| Shared Object AMF3 | 16 |  | AMF3编码，共享对象消息(携带用户详情) |
| Command AMF3 | 17 |  | AMF3编码，RTMP 命令消息，可能涉及用户数据 |
| Data AMFO | 18 |  | AMFO编码，音视频MetaData风格\| 配置详情包 |
| Shared Object AMFO | 19 |  | AMFO编码，共享对象消息(携带用户详情) |
| Command AMFO | 20(0x14) | Connect 0，play 1 | AMFO编码，RTMP 命令消息，可能涉及用户数据 |
| Aggregate Message | 22 |  | 整合消息 |
|  |  |  |  |

##### fmt 为1 时， 7bytes
	
省去了表示**message stream id**的4个字节，表示此chunk和上一次发的 chunk 所在的流相同，如果在发送端和对端有一个流链接的时候可以尽量采取这种格式。

```
    0               1               2               3  
    0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |timestamp  delta                               | message length|
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  message length(continue)     |message type id|
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- timestamp delta: 
	- 此时存储的时间戳时和上个chunk的时间差。 3 bytes, 最多表示16777215=0xFFFFFF=2^24-1 ，当超过最大值时，三个字节均置为为1，并将实际时间差转到 Extend Timestamp

##### fmt 为2 时， 3bytes

省去了表示**message stream id**的4个字节，表示此chunk和上一次发的 chunk 所在的流相同，如果在发送端和对端有一个流链接的时候可以尽量采取这种格式。
省去了表示**message type id**的1个字节，表示此chunk和上一次发的 chunk 所在的流相同，如果在发送端和对端有一个流链接的时候可以尽量采取这种格式。

```
    0               1               2               
    0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |timestamp   delta                              | 
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

##### fmt 为3 时， 0bytes
 表示该chunk message header 与上一个完全相同。
 时间戳也相同，表示是同一个message 拆分成多个chunk



### 控制流消息 (CSID = 0x02)
当**基本头**中的 **CSID** = 2,表示控制流信道。
	
#### Set Chunk Size (Msg Type id=0x01)
设置块大小
默认最大的块大小为 128 字节，客户端和服务器可以使用此消息来修改默认的块大小。
例如，假设客户端想要发送的音频数据大小为131 字节，而块大小为 128 字节。
在这种情况下，客户端可以通知服务器新的块大小为 131 字节，然后就可以使用一个块来发送完整的音频数据了。

```
 0               1               2               3   
 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|0|                       chunk size (4 bytes)                  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
 - 0：当前比特位必须为零。
- chunk size（31比特）：本字段标识了新的最大块大小，以字节为单位，发送端之后将使用此值作为最大的块大小。本字段的有效值为 1 - 2147483647(0x7FFFFFFF)，由于消息的最大长度为 16777215(0xFFFFFF)

####  Window Acknowledgement Size (Msg Type id=0x05)
应答窗口大小，客户端和服务器发送这个消息来通知对方应答窗口的大小。
发送方在发送了等于窗口大小的数据之后，等待接收对方的应答消息（在接收到应答之前停止发送数据）。
接收方必须发送应答消息，在会话开始时，或从上一次发送应答之后接收到了等于窗口大小的数据。
```
 0               1               2               3   
 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|        Window acknowledgement size (4 bytes)                  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```


#### Set PeerBandwidth (Msg Type id=0x06)

当**消息头**中的 **message type id** = 0x6
设置流带宽，客户端和服务器发送此消息来说明对方的出口带宽限制。
接收方以此来限制自己的出口带宽，即限制未被应答的消息数据大小。
接收到此消息的一方，如果窗口大小与上次发送的不一致，应该回复应答窗口大小的消息。
```
 0               1               2               3   
 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|        Window acknowledgement size (4 bytes)                  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   Limit Type  |
+-+-+-+-+-+-+-+-+
```

- Window acknowledgement size: 
- Limit Type :
	- 硬限制（0）：应该限制出口带宽为指明的窗口大小。
	- 软限制（1）：应该限制出口带宽为指明的窗口大小，或已经生效的小一点的窗口大小。
	- 动态限制（2）：如果上一次为硬限制，此消息被视为硬限制，否则忽略此消息 

####  User Control Message (Msg Type id=0x04 用户控制消息)
告知对方执行该信息中包含的用户控制事件
用户控制消息是在 RTMP 协议层的，而不是在 RTMP chunk 流协议层，这个很容易弄混。该信息在 chunk 流中发送时
-  包结构
```
 0               1               2               3   
 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|   Event Type (2 bytes)         |  Event Data ...
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- 事件类型：

| Event|  Description |
| ---- |  ---- |
| Stream Begin  = 0|  |
| Stream EOF  = 1|  |
| Stream Dry  = 2|  |
| Stream Length  = 3|  |
| StreamIs Recorded  = 4|  |
| PingRequest  = 6|  |
| PingResponse = 7|  |

### Command Message  (CSID = 0x03 命令消息)

当**基本头**中的 **CSID** = 0x03,表示命令信道。
当**消息头**中的 **message type id** = 17(0x11) 或20(0x14)时，表示客户端和服务端之间传递命令消息。

#### 命令编码
客户端和服务器通过 **AMF 编码**的数据交换命令。 发送者发送包含**命令名称**，**事务ID**，包含相关参数的命令对象的消息。
例如，通过连接命令中包含的 APP 参数来告诉服务器连接的对方是哪个客户端。接收方处理命令消息，并使用相同的事务ID应答。应答字符串为 _result 或 _error 或方法名

| Field Name | Type | Description |
| ---- | ---- | ---- |
| Command Name | String | 表示命令名称 |
| Transaction ID | Number | 应答的ID和请求的ID一致。 |
| Properties | Object | 后可以跟多个 properties objecet作为命令的参数 |
| Info | Object |  |

#### 命令类型
下列类对象被用来发送各种命令，分为以下两层：
- NetConnection： 服务端和客户端之间网络连接的一种高级表示方式。 网络连接允许使用以下命令： connect，call，close，createStream
- NetStream：代表了发送音频流，视频流，控制流（播放、暂停等命令）或其他相关数据的频道，

##### NetConnection(网路连接命令)
###### connect 

命令执行过程中的消息流如下:
- 客户端发送**连接命令**给服务器，获得与服务器连接的实例。
- 服务器在接收到连接命令后，发送应答窗口大小的消息给客户端。同时与连接命令中接到的应用建立连接。
- 服务器发送**设置流带宽消息**给客户端。
- 客户端在接收并处理了设置流带宽的消息后，发送**应答窗口大小**的消息给服务器。
- 服务器接着发送**开始流**的用户控制消息给客户端。
- 服务器发送 **result 命令消息**给客户端，通知连接状态是成功或失败。命令消息中包含了事务ID。消息中还包含了像 FMS版本之类的属性，以及级别，编码，描述，对象编码等信息。

- Transaction ID 永远是1,因为这肯定是第一个事务。

| Properties Name | Type | Description | Example Value |
| ---- | ---- | ---- | ---- |
| app | String | 表示流的名称 | testapp |
| type | String |  |  |
| flashVer | String | flash Player 的版本号 | FMLE/3.0 (compatible; Lavf60.3.100) |
| tcUrl | String | 服务URL rtmp://127.0.0.1:1935/testapp |如ffmpeg推流url为/11/22/33 该字段只会保留为/11/22|
| videocodecs | Numbers |  |  |
| audiocodecs | Numbers |  |  |
###### releaseStream

###### createStream 
创建一条流通道

###### call
###### close
###### FCPublish

##### onBWDone
 Set PeerBandwidth 的应答???

###### _checkbw

##### NetStream(网路流命令)

###### play
该命令用来通知服务器开始播放流,多次使用该命令可以创建一个播放列表。
如果想要创建一个动态播放列表来在不同的直播或点播流之间切换，可以通过多次调用播放命令，同时将 Reset 字段设置为 false。
相反，如果想要立即播放指定的流，先清理掉之前的播放队列，再调用播放命令，同时将 Reset 字段设置为 true。

| Field Name | Type | Description | Example Value |
| ---- | ---- | ---- | ---- |
| Command Name | String | 命令名称，固定为 play |  |
| Transaction ID | Number | 事务ID | |
| Commands |Null |0x05|| 
| Stream Name |String | stream 名称 |如ffmpeg拉流url为/11/22/33 该字段只会保留33 |
| Start |Number | 开始 |-2000|
| Duration |Number |||
| Reset |Boolean | ||

###### play2

###### publish
发布，客户端发送此消息，用来发布一个有名字的流到服务器。其他客户端可以使用此流名来播放流，接收发布的音频，视频，以及其他数据消息

| Field Name | Type | Description | Example Value |
| ---- | ---- | ---- | ---- |
| Command Name | String | 命令名称，固定为 publish |  |
| Transaction ID | Number | 事务ID | |
| Commands |Null |0x05|| 
| Publishing Name |String | stream 名称 |如ffmpeg推流url为/11/22/33 该字段只会保留33 |
| Publishing Type |String |  |live / record, append|

服务器接收到此消息后，回复 **onStatus** 命令来标记发布的开始。

###### seek
定位，客户端发送此消息来定位多媒体文件或播放列表的偏移（以毫秒为单位）。

| Field Name | Type | Description | Example Value |
| ---- | ---- | ---- | ---- |
| Command Name | String | 命令名称，固定为 seek |  |
| Transaction ID | Number | 事务ID | |
| Publishing Name |String | stream 名称 |如ffmpeg推流url为/11/22/33 该字段只会保留33 |
| milliseconds |Numbers |  |ms|

服务端回复
	- 成功：NetStream.Seek.Notify
	- 失败：\_error


###### pause
暂停或开始播放

| Field Name | Type | Description | Example Value |
| ---- | ---- | ---- | ---- |
| Command Name | String | 命令名称，固定为 pause |  |
| Transaction ID | Number | 事务ID | |
| Commands |Null |0x05|| 
| Pause/Unpause Flag|Boolean |true:暂停；false：恢复播放  ||
| milliseconds |Numbers | 表示操作该流的客户端时间（用来识别最后一次操作） |ms|

服务端回复
	- 成功：NetStream.Seek.Notify
	- 失败：\_error


###### receiveAudio
###### receiveVideo

######  getStreamLength

###### deleteStream
删除流
| Field Name | Type | Description | Example Value |
| ---- | ---- | ---- | ---- |
| Command Name | String | 命令名称，固定为 deleteStream |  |
| Transaction ID | Number | 事务ID | |
| Commands |Null |0x05|| 
| ??? |Number |  |live / record, append|

###### closeStream
关闭流

### Audio Data  (CSID = 0x06 音频数据, Msg Type id=0x08)
所有的音频包是通过AUDIODATA结构封装
在向RTMP服务器推送音频流或者视频流时，首先要推送一个音频tag（AAC sequence header）和视频tag（AVC sequence header），没有这些信息播放端是无法解码音视频流的

####  AUDIODATA
- SoundFormat 音频格式  (4bit)
	- 0 Linear PCM , platform endian
	- 1 ADPCM
	- 2 MP3 
	- 1 Linear PCM, little endian
	- 7 G.711 A
	- 8 G.711 M
	- 10 AAC
- SoundRate 采样率(2bit)
	- 0=5.5-kHz
	- 1 11-kHz
	- 2 22-kHz
	- 3 44-kHz
- SoundSize 精度(1bit)
	- 0=8bit
	- 1=16bit
- SoundType 类型(1bit)
	- 0=sndMono 单声道
	- 0=sndStereo 双声道
- SoundData : 音频数据
	- 当SoundFormat=AAC时，表示AACAUDIODATA结构.
	
#### AACAUDIODATA
- AACPacketType(8bit)： 
	- 0: AAC 序列头
	- 1: AAC raw 数据
- DATA：AACPacketType == 0 时表示 AuudioSpecificCOnfig; 1时表示AAC Frame Data (AAC裸数据，如ADTS格式的数据需要去掉ADTS头信息)

##### AudioSpecificConfiguration (AAC sequence header)

- audioObjectType(5bit)： 编码结构。 2=AAC-LC
- samplingFrequencyIndex(4bit)： 音频采样率索引值。
- channelConfiguration(4bit)：音频输出声道
- frameLengthFlag(3bit)： 
	- frameLengthFlag(1bit)：  表明IMDCT窗口长度
	- dependsOnCoreCoder(1bit)：  表明是否依赖于corecoder
	- extensionFlag(1bit)： 


	
### Video Data (CSID = 0x07 视频数据, Msg Type id =0x09)

在向RTMP服务器推送音频流或者视频流时，首先要推送一个音频tag（AAC sequence header）和视频tag（AVC sequence header），没有这些信息播放端是无法解码音视频流的

- 帧类型keyframe（4bit）： 是否是关键帧 
- 编码类型format(4bit)： H264-7
- 视频数据：如H264采用AVCVIDEOPACKET格式封装



#### AVCVIDEOPACKET
- AVCPacketType  AVC包类型(8bit)：
	- 0 表示AVC的序列头
	- 1 表示 H264的nalu,表示为数据包
	- 2 表示AVC的序列头end
- Comosition time offset:  组合帧时间，一般没啥用，直接赋值为0 
- Data:   根据AVC包类型，封装相应的数据，
	 RTMP发送视频数据时， 客户端接收到AVCC 序列头之后，才能初始化解码器，进行后续nalu帧的解码操作，这里为了防止第一包数据丢失，一般服务器会在每一个IDR帧之前，都发送一包AVC sequence header，保证客户端能够在极端情况下，具备持续解码播放的能力。
	- AVCPacketType=0时按照AVCC序列头格式封装
	-  AVCPacketType=0为1时按照nalu封装。  

### 块数据(可变大小)
