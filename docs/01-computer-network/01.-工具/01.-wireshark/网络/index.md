# 网络<no value>

# 1. 常用过滤语法

 - eth[0] & 1 过滤出组播包和广播包



# 2. 包类型：
## 2.1. 以太网FCS错误（ETHERNET FRAME CHECK SEQUENCE INCORRECT） ：

    以太网计算出的校验码与接收到的不一致
	  说明出现错包，硬件故障？
    过滤条件： eth.fcs_bad (wireshark 默认不显示FCS错包，需要配置)


## 2.2. Fragmented 网络层分片包
 同个包的分片 IP.Identification 字段具有相同ID


## 2.3. Dup ACK 重复确认包 
  Seq 重复(代表没有发送新的数据)，Ack重复，会携带一个SACK
  一般代表客户端和服务端之间存在丢包，TCP正在尝试恢复
   典型场景：
	   客户端给服务端发1，2，3，4，5个包
	   服务端接收到1包，回复ack 接收到了1包
	   中途丢失了2包，服务端会在后续接收到的3，4，5都重复ack 接收到了1包

## 2.4. TCP Previous segment not captured
	在这个包之前有些包没有被抓包捕获到
	通过Seq序号不连续得知

## 2.5. TCP Retransmission