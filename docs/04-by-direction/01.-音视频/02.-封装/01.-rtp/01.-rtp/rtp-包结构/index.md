# Rtp 包结构<no value>

  ```
    0                   1                   2                   3  
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
   |V=2|P|X|  CC   |M|     PT      |       sequence number         |  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
   |                           timestamp                           |  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
   |           synchronization source (SSRC) identifier            |  
   +=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+  
   |            contributing source (CSRC) identifiers             |  
   |                             ....                              |  
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```


- V(version)：2 bits，RTP的版本，这里统一为2
- P(padding)：1 bit，如果置1，在packet的末尾被填充，填充有时是方便一些针对固定长度的[算法](http://lib.csdn.net/base/31 "算法与数据结构知识库")的封装
- X(extension)：1 bit，如果置1，在RTP Header会跟着一个header extension
- CC(CSRC count): 4 bits，表示头部后contributing sources的个数
- M(marker): 1 bit，具体这位的定义会在一个profile里
- PT(playload type): 7 bits，表示所传输的多媒体的类型，对应的编号在另一份文档rfc3551中有列出(http://tools.ietf.org/html/rfc3551)
- sequence number: 16 bits，每个RTP packet的sequence number会自动加一，以便接收端检测丢包情况
- timestamp: 32 bits，时间戳
- SSRC: 32 bits，同步源的id，没两个同步源的id不能相同
- CSRC: 上文说到，个数由CC指定，范围是0-15


## RTP封装H264码流
### 1.1.3. Slide 切片

#### 1.1.3.1. Single NAL unit
对于 NALU 的长度小于 MTU 大小的包, 一般采用单一 NAL 单元模式.

#### 1.1.3.2. 组合封包模式
 当 NALU 的长度特别小时, 可以把几个 NALU 单元封在一个 RTP 包中.
- STAP-A模式
- STAP-B的话会多加入一个DON域
#### 1.1.3.3. Fragmentation Units(FUs)

 当NALU的长度超过MTU时,就必须对NALU单元进行分片封包.也称为Fragmentation Units(FUs).