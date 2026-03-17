# Ice<no value>

#### ip_type
- host:  从本机网卡获取到的IP地址类型， 即内部ip地址和端口
- srvflx: server reflexive, STUN服务器获取到的出口IP地址类型，即出口NAT设备映射的公网IP地址和端口
- prflx: peer reflexive, STUN服务器从打洞报文中获取到的内部IP地址类型，一般情况下和host类型是一致的
- relay: TURN服务器提供的中继IP地址类型
		一般优先级 host > srflx > prflx > relay。

##### base address
- host candidate，base 是其自身
- srvflx address，base 是发送 stun binding request 给 stun 服务器的 host address
- relayed candidate，base 是其自身
- prflx address，base 是发送 stun binding request 给对端的 host address

host、srvflx、prflx 地址的 base address 很好理解，就是 ice agent 发送数据的内网源地址。  
relayed address 的 base address 并不是 ice agent 发送数据的内网源地址，而是 turn 服务器分配的 ip:port 地址对。


1. ​**进取型提名（Aggressive Nomination）已启用**：该模式下，每次检查均携带 `USE-CANDIDATE` 属性，直接确定有效路径，无需二次触发验证


当使用 ​**普通提名（Regular Nomination）​** 时：

1. 首次检查不携带 `USE-CANDIDATE` 属性，仅验证连通性。
2. 若该候选对已成功加入 Valid List，后续触发式检查会被跳过，直接进行二次提名检查