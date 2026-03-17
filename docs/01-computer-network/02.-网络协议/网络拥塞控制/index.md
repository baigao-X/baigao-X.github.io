# 网络拥塞控制<no value>

# 术语


### 网络缓冲区(Network Buffer)

![[Pasted image 20251208214908.png]]

- 当**发送速率**超过带宽，数据会在**网络缓冲区** 排队，数据不会丢包(不考虑物理链路错误导致的丢包)，只会导致**RTT**升高，
- 网络缓冲区会带来**延迟抖动**，和**缓冲区膨胀**

#### Buffer Size
指Network buffer的总内存大小，通常以字节为单位

*Buffer size = Queue Length x Packet Size x 8*
#### Queue Length
指发送队列最多能够排队等待的包数量

#### Max Queuing Time (最大排队时间)/ MaxBuffering Time (最大缓冲时间)：
 指特定瓶颈带宽下和缓冲区大小下，一个数据包能够最多缓冲的时间，它反映缓冲区对RTT的膨胀的影响程度

*Max Queuing Time = Buffer Size /BtlBw = (Queue Legth x Packet Size x 8) / BtlBw*

![[Pasted image 20251208233655.png]]

### 发送速率(sending rate)
发送方以多块的速率发送数据
wireshark 中可以通过**Throughput**表示，表示单位时间内发送方实际发送出去的数据量无论是否被确认

### 交付速率(delivery rate)
网络实际帮你送达的速度
wireshark 中可以通过**Goodput**表示，表示单位时间内被接收端确认的数据量

### 拥塞窗口(cwnd， Congestion   Window)
发送方根据网络状况决定的，一次能发多少
拥塞窗口是由发送端自己动态维护的， 不在TCP协议报文中体现，可以通过**BIF**来间接表示

- 慢启动会让TCP启动过程的cwnd初始值小
- 拥塞避免
- cwnd 变化通常以RTT为周期，cwnd增减不一定意味着发送速率增减
- *发送速率=cwnd /  RTT*, cwnd 表示一个RTT周期能发送的数据量
#### 接收窗口(rwnd，Receive Window)
 - TCP 中 接收方中通告的可接收大小，发送方无论任何时候都不能发送超过接收窗口限制的数据
 - 会在TCP 应答中体现,但是可以通过BIF 间接观察
#### 实际发送窗口(swnd，SendWindow，Effective Window)

理论：**swnd <= min(rwnd,cwnd)**---RFC5681
实际：受**应用层生成数据的速度、发送缓冲区(send buffer size)、接收缓冲区（reveive buffer size）、BDP&network buffer size** 应用

#### BIF (在途数据量，Bytes in filght、bytes out、 outstanding bytes)
已经发出，但是还未收到ACK的数据
BIF 的上限就越是 swnd
![[Pasted image 20251207202228.png]]

- 当**接收窗口构成限制**,BIF 主要受限于 swnd：*BIF最大值=swnd=rwnd*
![[Pasted image 20251207203155.png]]


- 当 **接收窗口足够大**,主要受限于cwnd的动态控制，*BIF最大值 ≈ swnd ≈ rwnd*
   实际BIF最大值与CWND一般存在一个差值，该差值受限于ACK返回频率、RTT是否稳定、发送端是否使用PACING机制
![[Pasted image 20251207203513.png]]


#### 滑动窗口
即表示cwnd 和rwnd 动态变化的机制形象化描述

### 瓶颈带宽(Bottleneck Bandwidth)
代表端到端路径上最窄处的带宽


### Latency (单程延迟)
表示一个数据包从发送端到接收端所经所的总时间，有以下部分组成
- 传播延迟(Propagation Delay): 电信号和光信号在物理介质需要的时间，取决于距离和传输介质特性
- 传输延迟(Transmision Delay)：串行化延迟，表示数据逐比特写入网络链路的时间，受**数据包大小**和**链路带宽**影响
- 处理延迟(Processing Delay)： 网络设备转发数据包所需要的处理时间，如校验、查找路由表、防火墙。 普通交换机可以忽略不计算，NAT设备、防火墙延时在0.1-0.5ms之间
- 排队延迟(Queuing Delay)： 网络缓冲区排队时间，**唯一一个不固定，实时变化的**
但是因为难以测量，一般使用RTT

### 往返时延(RTT，Round-Trip Time)
从sender 发出一个数据包，到receiver 发回的ACK所用的总时间，由以下部分组成
- Sneder->Receiver 的单程延迟
- ACK delay：Receiver 处理数据与生成ACK的延迟
	- 一般可以忽略不计，但是可能因为延迟确认机制、网络卸载功能(LRO**大数据接收卸载**、GRO**通用接收卸载**),负载变高等导致
- ACK从Receiver返回到Sender的单程延迟

#### BaseRTT,最小RTT：
也叫 RTT prop、Min RTT、Initial RTT
无排队情况下的最短往返时延,反应物理路径的最短延迟
*RTT = BaseRTT + Queuing time* （实际RTT = BaseRTT + Buffe 的排队时间）
#### iRTT，chushiRTT
通常表示三次握手中测得的RTT(wireshark 使用该术语)

![[Pasted image 20251207165403.png]]

### 缓冲区大小(Buffer Size)
### 排队延时(Queuing Time)
### 带宽延时积（BDP，bandwidth delay product）
表示在不丢包的情况下,想把整个路径灌满，需要多少数据量
*BDP = BaseRTT  * 瓶颈带宽*

# Buffer /Queuing

Buffer 分为MAC层缓冲区和协议栈缓冲区
### MAC层缓冲区：Tx/Rx Ring (发送/接收环形缓冲区)

- 查看: ethtool -g eth0
- 修改: sudo ethtool -G eth0 rx 4960 tx 4960

#### Tx-Ring
![[Pasted image 20251208224407.png]]

- Tx-Ring: 协调发送速率与物理速率，队列一般为**FIFO**
- Tx-Ring满了，就会在Socket Buffer中排队，会触发Qos机制

#### Rx-RIng

![[Pasted image 20251208224652.png]]
- Rx-ring : 应对突发流量，避免上层处理不过来时丢包，队列时FIFO
- Rx-ring满时直接丢包

### 协议栈缓冲区
skb(socket buffer): 是Linux网络协议栈中贯穿L2-L4处理过程中的数据结构
- 如果是 路由器、交换机、防火墙等中间设备，就对应网络缓冲区(Network Buffer)
- 查看： 
	- sudo sysctl -a | grep net.ipv4.tcp_rmem # 接收缓冲区
	- sudo sysctl -a | grep net.ipv4.tcp_wmem # 发送缓冲区
![[Pasted image 20251208224842.png]]
- 支持Qos队列机制，区分流量优先级与调度策略

**端到端网络模型-App-to-App视角**如下：

![[Pasted image 20251208224117.png]]

FIFO的Tx/Rx Ring 影响相对较小，不考虑后，可以简化为**端到端模型-Tcp视角如下:**
![[Pasted image 20251208230009.png]]

当调大send buffer  和 receive buffer，进一步可以简化成**端到端网络模型-拥塞控制视角**：

![[Pasted image 20251208230235.png]]



# 网络拥塞模型

![[Pasted image 20251209190620.png]]

- **APP limited（应用受限区）**：
	- 发送数据未填满BDP，
	- 可能原因：
		- Sender自身性能
		- Receive Window限制
		- TCP拥塞控制算法如慢启动
		- Sender 主动降速(业务限制传输速率)
- **Bandwidth Limited (带宽受限区)**
	- 发送数据未填满BDP，但是还未填满Network Buffer
- **Buffer limited（缓冲区受限区）**
	-  发送数据溢出了Network Buffer，已经丢包了

**1979年 Leonard Kleinrock提出的最佳操作点**，即发送速率刚好利用满带宽，而缓冲区还没开始排队，此时RTT对端，且带宽利用率百分之百
拥塞控制点一般有三种切入点：以下三个点都可以从不同意义上认为是发送**拥塞点**
1. 当cwnd 刚超过BDP时，(即发送速率刚超过瓶颈带宽时): 
2. 当缓冲区溢出发生丢包时（基于丢包的）
3. 当RTT超过BaseRTT时(基于延时的拥塞控制算法)
	
![[Pasted image 20251209162419.png]]
### 缓冲区膨胀（bufferbloat）

**大缓冲带来的高延迟**

- 产生的原因：
	- 网络缓冲区为了减少丢包，而将缓冲区设计的过大
	- 基于丢包的拥塞控制算法,如Reno和Cubic，他们依赖丢包作为拥塞信号，不关注RTT增加
	- 不合理的排队策略
- 避免的方法：
	- **主动队列管理(APM)** 主动丢包策略，在网络缓冲区未满时，主动丢弃一些包，来让发送端尽早感知网络阻塞
	- 拥塞算法改进，如BBR
	- 终端主机侧优化技术》：**Pcaing(平滑发送)**，避免一瞬间把网络冲爆
	- 网络设备设计改进

  ![[Pasted image 20251208231108.png]]

- **因此缓冲区膨胀，cwnd也可能膨胀，但是因为随之而来的RTT也变高了，发送速率并没有变高**

### Buffer-BDP 比值
指缓冲区大小与BDP的比值，

- Buffer 过大：
	- 高延迟：因为数据包在队列中堆积
	- 低吞吐量
	- 不公平性： 一些激进的流比如大文件下载会把缓冲区撑满，影响其他数据流
- Buffer 过小：
	- 更频繁的丢包
	- 链路利用率变低，因为丢包会降速，无法充满带宽


![[Pasted image 20251209160350.png]]

- 对于 Reno / NewReno 当buffer < BDP, 当丢包后，cwnd就会降到BDP之下，带宽就用不满
- 对于 Cubic 当buffer < 0.43 x BDP, 当丢包后，cwnd就会降到BDP之下，带宽就用不满

![[Pasted image 20251209171113.png]]

#### Buffer-BDP 比值 <1 Reno/NewReno 拥塞避免阶段
##### RTT和cwnd(BIF)
![[Pasted image 20251209172415.png]]

**拥塞避免时回落到BaseRTT的时间对应的BIF值就是BDP的值。**

##### Send Rate 、Delivery Rate与cwnd

![[Pasted image 20251209173017.png]]

##### RTT、BIF和send rate

![[Pasted image 20251209173916.png]]
#### Buffer-BDP 比值 >1 Reno/NewReno 拥塞避免阶段

![[Pasted image 20251209173558.png]]

**拥塞避免阶段，始终占满带宽**

##### Send Rate 、Delivery Rate与cwnd
![[Pasted image 20251209173744.png]]

##### RTT、BIF和send rate

![[Pasted image 20251209173823.png]]


#### Slow Start 阶段

![[Pasted image 20251209174020.png]]


#### Qos Policing 场景
![[Pasted image 20251209191600.png]]


# 拥塞算法

## TCP 拥塞算法
### 丢包型拥塞控制算法

1. ​**TCP Reno**
    
    - 引入快重传与快恢复，避免因超时导致的性能骤降。
    - ​**示例**：若ssthresh=16，cwnd在超时后重置为1（慢启动），但快恢复阶段cwnd直接设为8

2. ​**TCP CUBIC**
    - 基于立方函数动态调整cwnd，适应高带宽延迟积（BDP）网络。
    - ​**公式**：`cwnd = C*(t-K)^3 + W_max`，其中t为时间，K为上次拥塞事件的时间差
	
### 延迟型拥塞控制算法
1. ​**BBR（Bottleneck Bandwidth and RTT）​**
	-  通过建立流量模型
    - 通过主动测量带宽和最小RTT构建网络模型，动态调整发送速率。
    - ​**优势**：在长肥管道（如卫星链路）中显著降低延迟和丢包率 
![[Pasted image 20251209190313.png]]