# Websocekt协议解读<no value>

# 1. 标准
[RFC-6455](https://www.rfc-editor.org/info/rfc6455)

基于TCP
# 2. 报文格式

## 2.1. http升级websockt 的协商报文
### 2.1.1. 示例报文
```
- GET /webfin/websocket/ HTTP/1.1。

- Host: localhost。
- Upgrade: websocket。
- Connection: Upgrade。
- Sec-WebSocket-Key: xqBt3ImNzJbYqRINxEFlkg==。
- Origin: [http://服务器地址](https://link.zhihu.com/?target=http%3A//xn--zfru1gfr6bz63i/)。
- Sec-WebSocket-Version: 13。

服务器回应:

- HTTP/1.1 101 Switching Protocols。
- Upgrade: websocket。
- Connection: Upgrade。
- Sec-WebSocket-Accept: K7DJLdLooIwIG/MOpvWFB3y3FE8=。
- WebSocket借用http请求进行握手，相比正常的http请求，多了一些内容。其中：
- Upgrade: websocket。
- Connection: Upgrade。
- 表示希望将http协议升级到Websocket协议。Sec-WebSocket-Key是浏览器随机生成的base64 encode的值，用来询问服务器是否是支持WebSocket。

服务器返回:
```
#### 2.1.1.1. 字段说明
- Upgrade: websocket。
- Connection: Upgrade:  告诉浏览器即将升级的是Websocket协议
- Sec-WebSocket-Accept: 是将请求包“Sec-WebSocket-Key”的值，与”258EAFA5-E914-47DA-95CA-C5AB0DC85B11″这个字符串进行拼接，然后对拼接后的字符串进行sha-1运算，再进行base64编码得到的。用来说明自己是WebSocket助理服务器。
- Sec-WebSocket-Version: WebSocket协议版本号。RFC6455要求使用的版本是13，之前草案的版本均应当被弃用。

### 2.1.2. Websocket 报文格式

![[Pasted image 20240819085930.png]]

```
  0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-------+-+-------------+-------------------------------+
     |F|R|R|R| opcode|M| Payload len |    Extended payload length    |
     |I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
     |N|V|V|V|       |S|             |   (if payload len==126/127)   |
     | |1|2|3|       |K|             |                               |
     +-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
     |     Extended payload length continued, if payload len == 127  |
     + - - - - - - - - - - - - - - - +-------------------------------+
     |                               |Masking-key, if MASK set to 1  |
     +-------------------------------+-------------------------------+
     | Masking-key (continued)       |          Payload Data         |
     +-------------------------------- - - - - - - - - - - - - - - - +
     :                     Payload Data continued ...                :
     + - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
     |                     Payload Data continued ...                |
     +---------------------------------------------------------------+
	 
```


- FIN: 1bit
	 表示这是消息中的最后一个片段
- RSV1、RSV2、 RSV3: 各占1bit
	 预留位，必须为0
- opcode： 4bit
	操作码
	- 0-表示连续帧
	- 1-表示文本框
	- 2-表示二进制帧b
	- 3-7保留用于进一步的非控制帧
	- 8-表示连接关闭
	- 9-表示 ping
	- A-表示 ping pong
	- B-F 保留用于进一步的控制帧
- MASK: 1bit
	 掩饰码
	 定义Payload data 是否被屏蔽，即通过MaskKing-Key运算处理过l
- PayLoad Len： 7bit/7+16bit/7+64bit
	表示Payload Data的长度，以字节为单位,网络字节序表示：
	前7个bit：
	- 0-125，则共占用7bit，其值表示PayLoad Len
	- 126, 则接下来的16bit表示PayLoadLen
	- 127, 则接下来的64bit表示PayLoadLenL
MaskKing-Key： 0-4字节
	根据MASK 标志位而定
Payload： 载荷，都要使用帧首部中指定的值加掩码