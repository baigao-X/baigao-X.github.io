# 流量控制 （Traffic Control, Tc）<no value>

流量控制（`Traffic Control`， `tc`）是`Linux`内核提供的流量限速、整形和策略控制机制。它以`qdisc-class-filter`的树形结构来实现对流量的分层控制 ：

`tc`最佳的参考就是[Linux Traffic Control HOWTO](http://www.tldp.org/HOWTO/Traffic-Control-HOWTO/)，详细介绍了tc的原理和使用方法。
https://wiki.archlinux.org/title/advanced_traffic_control#Stochastic_Fairness_Queueing_(28SFQ)


从上图中可以看到，tc由`qdisc`、`fitler`和`class`三部分组成：

- `qdisc`通过队列将数据包缓存起来，用来控制网络收发的速度
- `class`用来表示控制策略
- `filter`用来将数据包划分到具体的控制策略中

# 1. qdisc

- [无分类qdisc（只能应用于root队列）](http://tldp.org/HOWTO/Traffic-Control-HOWTO/classless-qdiscs.html)
	- `[p|b]fifo`：简单先进先出
	- `pfifo_fast`：根据数据包的`tos`将队列划分到3个`band`，每个`band`内部先进先出
	- `red`：`Random Early Detection`，带带宽接近限制时随机丢包，适合高带宽应用
	- `sfq`：`Stochastic Fairness Queueing`，按照会话对流量排序并循环发送每个会话的数据包
	- `tbf`：`Token Bucket Filter`，只允许以不超过事先设定的速率到来的数据包通过 , 但可能允许短暂突发流量朝过设定值
- [有分类qdisc（可以包括多个队列）](http://tldp.org/HOWTO/Traffic-Control-HOWTO/classful-qdiscs.html)
    - `cbq`：`Class Based Queueing`，借助`EWMA`(`exponential weighted moving average`, 指数加权移动均值 ) 算法确认链路的闲置时间足够长 , 以达到降低链路实际带宽的目的。如果发生越限 ,`CBQ` 就会禁止发包一段时间。
    - `htb`：`Hierarchy Token Bucket`，在`tbf`的基础上增加了分层
    - `prio`：分类优先算法并不进行整形 , 它仅仅根据你配置的过滤器把流量进一步细分。缺省会自动创建三个`FIFO`类。