# 弱网<no value>

衡量弱网的几个指标

- 带宽（吞吐量）：单位时间内传输的数据量，单位通常是：每秒比特数：bps。 反映网络传输能力。
- 丢包：数据丢包个数=发送的数据包数-接收的数据包数。 反映网络可靠性。
- 时延：数据包从发送开始到接收到该数据，所耗费的时间，单位通常是ms。反映网络速度。 
- 抖动(Jitter)：指时延的变化，即两个数据包时延的差值。 反映网络稳定性。
- 乱序：指接收到的数据包顺序和发送顺序不一致的次数。反映网络稳定性。一般乱序比较严重的时候，丢包也会比较严重，所以一般都以丢包指标为主，忽略乱序指标。



# 1. 如何分析和衡量

# 2. 如何模拟

- [[01. 流量控制 （Traffic Control, tc）| tc]]  : linux 通过流量控制模块设置
``` 
1. tc qdisc del root dev em4
2. tc qdisc add dev em4 root handle 1: htb default 30
3. tc class add dev em4 parent 1: classid 1:20 htb rate 5mbit ceil 5mbit
4. tc qdisc add dev em4 parent 1:20 handle 20: sfq perturb 10
5. tc class add dev em4 parent 1:1 classid 1:30 htb rate 1000mbit
6. tc qdisc add dev em4 parent 1:30 handle 30: sfq perturb 10
7. tc filter add dev em4 protocol ip parent 1: prio 1 u32 match ip dport 10990 0xffff flowid 1:20

1. 删除网卡em4上现有的所有规则
2. 在网卡em4上添加一个名字为1:的root HTB qdisc
3. 在root qdisc下创建一个分支1:20，这个分支的带宽固定为5mbit/s
4. 在分支1:20下创建一个classless 的 sfq qdisc，该队列可以随机的发送其内部存放的packet，perturb 10表示该对队列每间隔10s更新一下自身的hash算法。这个随机算法能较为公平的发送其内部的数据包。
5. 在root qdisc下创建一个分支1:30，这个分支的带宽为1000mbit/s
6. 在分支1:30下创建一个classless 的 sfq qdisc，记为30
7. 在root队列上添加filter，该filter将所有目的端口为10990的包，都存放到了分支1:20下，剩余的默认存放到1:30下。
通过上述代码可以让所有发往端口10990的包的发送速度控制在 tc的速度限定值（5mbits/s）/用户数
```
		
