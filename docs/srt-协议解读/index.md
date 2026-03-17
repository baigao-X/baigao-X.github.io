# Srt 协议解读<no value>

	Secure Reliable Transport
	
简单理解为 基于UDP的封装协议，针对流媒体传输实现了前向纠错来实现
haivision wowza  两个流媒体服务商一起创造, SRT是基于UDT的协议（UDT协议是基于UDP的传输协议，在IETF已经提交了4个版本）



[协议文档中文版](https://blog.csdn.net/sweibd/article/details/105622218)
	
### 开源实现

#### [Haivision/srt](https://github.com/Haivision/srt)
	https://github.com/Haivision/srt/blob/master/docs/API/API-socket-options.md#list-of-options
#### [srs]( https://ossrs.net/lts/zh-cn/docs/v6/doc/srt)

### 握手模式

#### Caller-Listener
其中一端等待另一端发起连接
Initiator （发起方） 和 Responder （响应方） 角色是根据连接模式分配的。
对于 Caller-Listener 连接：Caller 是 Initiator， 侦听器是响应者。

步骤:
1. 调用方到侦听器：INDUCTION 请求
2. 侦听器到调用方：INDUCTION 响应（报告 Cookie）
3. 调用方到侦听器：CONCLUSION 请求（使用之前返回的 Cookie）
4. 侦听器到调用方：CONCLUSION 响应（确认已建立连接）。
    
#### Rendezvous
双方都尝试发起连接 
对于 Rendezvous 连接：分配 Initiator 和 Responder 角色 基于握手期间的初始数据交换。

步骤:
1. 启动连接后的两个对等节点：带有 cookie 的 WAVEAHAND
2. 收到对端的上述消息后：总结
3. 收到对端发送的上述消息后：AGREEMENT。

### 数据传输模式
#### 消息模式 message

- 每个数据包都有自己的数据包序列号。
- 一个或多个连续的 SRT 数据包可以形成一条消息。
- 属于同一消息的所有数据包都设置了相似的消息编号 在 Message Number 字段中。
在此模式下，单个 发送指令只传递一条具有边界的数据（一条消息）。 此消息可能跨越多个 UDP 数据包和多个 SRT 数据包。这 唯一的大小限制是它应该作为一个整体适合 sender 的缓冲区，并且 接收器。

#### 缓冲区模式 buffer
在此模式下，连续的数据包形成一个连续的流，可以使用 任何大小的部分。

### 流控

#### timestamp-based Packet Delivery(tsbPD)  基于 SRT 时间戳的数据包传送 机制
简单来说就是发送端会保存发送的包队列，以便重新发送接收方应答为NAK的数据包

##### ACK
![[Pasted image 20241209085234.png]]
ACK包具有以下几种类型：
-  **Full ACK**: 每 10 毫秒发送一个完整的 ACK 控制数据包，其中包含上图所有字段
- **Small ACK**:小 ACK 包括 ***Available Buffer Size***（可用缓冲区大小） 字段之前（包括 Available Buffer Size） 字段的字段。Type-specific Information 字段应设置为 0。
- **Light ACK**:仅包含 ***Last Acknowledged Packet Sequence Number*** 字段。Type-specific Information 字段应设置为 0,
	 未达到Full Ack时间阈值(10ms)时，每64个数据包后发送一个
###### Last Acknowledged Packet Sequence Number（最后确认的数据包序列号）
此字段包含最后一个被确认的数据包的序列号加 1。换句话说，如果它是第一个未确认数据包的序列号。
###### RTT
RTT 值（以微秒为单位），由接收方根据之前的 ACK/ACKACK 数据包对交换估计。
###### RTT Variance
RTT 估计值的方差，以微秒为单位。

###### Available Buffer Size
接收方缓冲区的可用大小（以数据包为单位)

###### Packets Receiving Rate
接收数据包的速率，以每秒数据包数为单位

###### Estimated Link Capacity

链路的估计带宽，以每秒数据包数为单位

###### Receiving Rate

估计接收速率，以每秒字节数为单位

##### NAK 
![[Pasted image 20241209132753.png]]
检测到数据包序号有差距后，可以**立即触发** NAK 数据包的发送。
也可以使用**定期** NAK 报告机制定期发送 NAK 报告。
###### Loss List
由1-N个Nak list组成，未收到的连续的多个seq，合并成一个list
- type A:  List 仅 一个包未收到
	- **Lost packet sequence number** :  最高位为0，其余的值表示未收到的包序号
- tyep B: List 超过1个连续的seq号组成
	- **Range of lost packets from sequence number**: 最高位为1，表示Nak list的 开始seq
	- **Up to sequence number**: 最高位为0， 表示Nak List 的结尾seq (该seq也是未收到的seq)

##### ACKACK
![[Pasted image 20241209141246.png]]

收到ACK之后立即应答,否则对端会持续发送ACK
基于发送ACK和接收ACKACK的时间差，就是**RTT**

###### Acknowledgement Number

##### Message Drop Request
![[Pasted image 20241209143149.png]]

收到 **Nak**包之后，但是发送队列中已经没了，发送Drop Request包。 表示对端不用再等了
- First Packet Sequence Number: 队列的第一个seq
- Last Packet Sequence Number: 队列的最后一个seq


#### Periodic NAK Reports: 机制
- 常规 NAK :
	检测到数据包序号有差距后，可以**立即触发** NAK 数据包的发送。
 - 开启定期 NAK 报告机制:  
	 NAK 数据包将包含接收方在发送定期 NAK 报告时认为已丢失的所有数据包。
	**NAKInterval** =  (RTT + 4 * RTTVar) / 2, 下限为 20 毫秒

#### Too-Late Packet Drop 机制
 SRT 1.0.5 中引入
发送包将超过设定的播放时间之外的缓存队列丢弃，这些包无法再进行重传
建议的阈值是 SRT 延迟值的 1.25 倍。
请注意，SRT 发送方会将数据包保留至少 1 秒，以防延迟不足以应对大型 RTT（即，如果 TLPKTDROP_THRESHOLD小于 1 秒）。

### 加密

#### Key Material 密钥材料

##### 握手阶段 Key Material Extension Message

![[Pasted image 20241211103424.png]]

4bytes: 表示KM State
56bytes: 表示 KMX message,格式等同User-Defined control packet 包中的KMX mesage

#####  用户定义的控制数据包 User-Defined control packet
###### KMX message
![[Pasted image 20241211132807.png]]
- KK :
	Key-based Encryption 表示提供的是奇数键还是偶数键还是都提供
- KEKI: 
	Key Encryption Key Index (KEKI):密钥加密密钥索引
- SLen /4 : Salt 长度(以字节为单位)/4
- KLen /4: SEK 长度(为单位)/4
- Salt
	一个可变宽度字段，长度为SLen
- Wrapped Key :
	 8bytes + n * KLen bytes,  n由KK字段决定是1还是2
![[Pasted image 20241211151608.png]]
	- ICV Integrity Check Vector: 
		64 位完整性检查向量（AES 密钥包装完整性）此字段用于检测密钥是否已正确解包。
		如果手头的 KEK 无效，则验证将失败，并且将丢弃未包装的密钥。
		 应为wrap 使用常量IV A6A6A6A6A6A6A6A6，所以可以用于完整性键检查
	- xSEK: 
		可变宽度
		表示奇数/偶数SEK。如果只有一个密钥，KK字段会表示提供了那个SEK，如果都存在，该字段为偶数键
	- oSEK
		可变宽度，KK字段表示两个字段都存在时，该字段才存在，表示奇数键

##### KM刷新
建议的 KM 刷新周期是在发送 2^25 个使用相同 SEK 加密的数据包之后。
建议的 KM 预通告期为 4000 个数据包（即，在 2^25 减去 4000 个数据包时生成、包装和发送新密钥;旧密钥在 2^25 加 4000 个数据包时停用）。
#### 加密方法

##### KEK :the Key Encrypting Key 密钥加密密钥 
长期密钥，基于密码的密钥生成函数 （PBKDF2）生成
用于生成一个包装[RFC3394](https://www.rfc-editor.org/rfc/rfc3394)
KEK 必须至少与 SEK 一样长

###### 生成方式
``` 
SEK = PRNG(KLen) 
Salt = PRNG(128) 
KEK = PBKDF2(passphrase, LSB(64,Salt), Iter, KLen)

Wrap = AESkw(KEK, SEK) 
```

- **passphrase**: 指代用户密码，基于该密码迭代生成KEK
- **PRNG** : 伪随机数生成器
- **LSB(n, v)**: 指采用v的n个最低有效位函数 
- **PBKDF2**: 标准密钥迭代算法
- **Iter** 迭代次数，使用2048
- **KLen**: KM 消息的一个字段
- **Wrap/AESkw** 指代密钥包装函数

##### SEK, Stream Encrypting Key 流加密密钥,
短期密钥。最多 2^25 个数据包，并进一步重新生成密钥。

##### 加密Payload
```
EncryptedPayload = AES_CTR_Encrypt(SEK, IV, UnencryptedPayload)
IV = (MSB(112, Salt) << 2) XOR (PktSeqNo)
```

- - **LSB(n, v)**: 指采用v的n个最高有效位函数 
- PktSeqNo: SRT 数据包的 Packet Sequence Number 字段的值

### 协议报文
#### Data Packet 数据包
![[Pasted image 20241206102032.png]]


#### Control Packets 控制数据包
![[Pasted image 20241206101919.png]]

- Timestamp: 以微秒为单位。该值相对于建立 SRT 连接的时间。
- Destination SRT Socket ID: 一个固定宽度的字段，提供应将数据包调度到的 SRT 套接字 ID。当数据包是连接请求时，该字段可能具有特殊值 “0”

##### Caller-Listener Handshake
 控制数据包的一种：	
 - Control Type: 0x0000 表示握手类型
 - SubType: 不适用全0
- Type-Specific Information: 握手包不使用该字段，全0
- CIF: 见下表具体内容
![[Pasted image 20241206101831.png]]
###### Indcution Request ： caller->listener

| 字段名称                           | 值生成类型  | 字段值    | 字段含义                                          | 备注  |
| ------------------------------ | ------ | ------ | --------------------------------------------- | --- |
| Destination SRT Socket ID      | **固定** | 0      | 目标Socket ID                                   |     |
| HS Version                     | **固定** | 4      | 握手版本号                                         |     |
| Encryption Field               | **固定** | 0      | 加密字段                                          |     |
| Extension Field                | **固定** | 2      | 扩展字段                                          |     |
| MTU                            | **配置** | <=1500 |                                               | MSS |
| Max FlowWindow Size            | **配置** | 8192   | 此字段的值是允许“正在传输”的最大数据包数（即尚未收到 ACK 控制数据包的已发送数据包数 |     |
| initial packet sequence Number | **随机** |        |                                               |     |
| SRT Socket ID                  | **随机** |        |                                               |     |
| SYN Cookie                     | **固定** | 0      | Caller 的 Socket ID                            |     |
| Peer Ip Address                | **规律** |        | 表示数据发送方的IP                                    |     |

###### Indcution Response ： listener --> caller
| 字段名称                           | 值生成类型  | 字段值                                                     | 字段含义                                          | 备注  |
| ------------------------------ | ------ | ------------------------------------------------------- | --------------------------------------------- | --- |
| Destination SRT Socket ID      | **规律** | Indcution Request  ***SRt Socket ID***                  | caller  Socket ID                             |     |
| HS Version                     | **固定** | 5                                                       | 握手版本号                                         |     |
| Encryption Field               | **配置** |                                                         | 加密字段                                          |     |
| Extension Field                | **固定** | 0x4A17                                                  | 扩展字段                                          |     |
| MTU                            | **配置** | <=1500                                                  |                                               | MSS |
| Max FlowWindow Size            | **配置** | 8192                                                    | 此字段的值是允许“正在传输”的最大数据包数（即尚未收到 ACK 控制数据包的已发送数据包数 |     |
| initial packet sequence Number | **规律** | Indcution Request  ***initial packet sequence Number*** |                                               |     |
| SRT Socket ID                  | **随机** |                                                         | 表示listenr的Socket ID                           |     |
| SYN Cookie                     | **随机** |                                                         | 根据主机、端口和当前时间制作的 Cookie，精度为 1 分钟，以避免 SYN 泛洪攻击  |     |
| Peer Ip Address                | **规律** |                                                         | 表示数据发送方的IP                                    |     |

###### Conclusion Response ： caller->listener

| 字段名称                           | 值生成类型  | 字段值                                                                                            | 字段含义                                          | 备注  |
| ------------------------------ | ------ | ---------------------------------------------------------------------------------------------- | --------------------------------------------- | --- |
| Destination SRT Socket ID      | **规律** | Indcution Response  ***SRt Socket ID***                                                        | listener  Socket ID                           |     |
| HS Version                     | **固定** | 5                                                                                              | 握手版本号                                         |     |
| Encryption Field               | **配置** |                                                                                                | 加密字段                                          |     |
| Extension Field                | **固定** | 0x4A17                                                                                         | 扩展字段                                          |     |
| MTU                            | **配置** | 1500                                                                                           |                                               | MSS |
| Max FlowWindow Size            | **配置** | 8192                                                                                           | 此字段的值是允许“正在传输”的最大数据包数（即尚未收到 ACK 控制数据包的已发送数据包数 |     |
| initial packet sequence Number | **规律** | Indcution Request  ***initial packet sequence Number***                                        |                                               |     |
| SRT Socket ID                  | **规律** | Indcution Request  ***SRt Socket ID***                                                         | caller的Socket ID                              |     |
| SYN Cookie                     | **规律** | **Caller-Listener模式**为***Indcution Response  ***SYN Cookie*** <br>**Rendezvous模式**存在 Cookie 竞赛 | 表示归纳的响应方Cookie                                |     |
| Peer Ip Address                | **规律** |                                                                                                | 表示数据发送方的IP                                    |     |

**Extension-SRT_CMD_HSREQ**

| 字段名称                 | 值生成类型  | 字段值 | 字段含义        | 备注          |
| -------------------- | ------ | --- | ----------- | ----------- |
| SRT Version          | **固定** |     | 表示SRT库的版本号  |             |
| SRT Flags            | **配置** |     |             | 见SRT Flags表 |
| Receiver TSBPD Delay | **配置** |     | 接收的TSBPD 延迟 | 毫秒单位        |
| Sender TSBPD Delay   | **配置** |     | 发送的TSBPD 延迟 | 毫秒单位        |

**SRT Flags**

| 位掩码字段         | 值生成类型 | 字段值 | 字段含义                                               |
| ------------- | ----- | --- | -------------------------------------------------- |
| TSBPDSND      | 配置    | 1   | 是否将使用 **TSBPD机制**进行发送                              |
| TSBPDRCV      | 配置    | 1   | 是否将使用 **TSBPD机制**进行接收                              |
| CRYPT         | 配置    | 1   | 是否了解SRT数据包中**KK**字段                                |
| TLPKTDROP     | 配置    | 1   | 是否开启Too-Late Packet Drop 机制                        |
| PERIODICNAK   | 配置    |     | 是否将定期发送 NAK 数据包                                    |
| REXMITFLG     | 配置    |     | 是否了解SRT数据包中的**R**字段                                |
| STREAM        | 配置    |     | 表示连接中要使用的**传输模式**<br>1-表示**缓冲区模式**<br>0-表示**消息模式** |
| PACKET_FILTER | 配置    |     | 表示是否支持数据包过滤器                                       |
