# Iperf, Iperf3<no value>

测速工具

# 1. iperf

server端：

``` shell
iperf -s -p 25001 -B 192.168.33.103 (-u)
- s  指定server端
- p 指定端口（要和客户端一致）
- B 绑定ip地址 
- u  udp协议，，默认是tcp协议
```

client端：

```shell
iperf -c -p 25001 -B 192.168.33.104 -4 -f K -n 10M -b 10M （-u）
- c 指定client端
- p 指定端口（要和服务器端一致）
- B 绑定客户端的ip地址
- 4 指定ipv4
- f 格式化带宽数输出
- n 指定传输的字节数
- b 使用带宽数量 
- u 指定udp协议
```

# 2. iperf3
``` shell
iperf3 -s -p 25001
- s 指定服务器端
- p 指定端口号   

iperf3的server端不支持“-u”参数，，默认可以测试tcp和udp
```

- 客户端
``` shell
iperf3 -c RemoteIP -p 25001 -B 192.168.33.104 -4 -f K -n 10M -b 10M --get-server-output（-u）
- c 指定client端
- p 指定端口（要和服务器端一致）
- B 绑定客户端的ip地址
- 4 指定ipv4
- f 格式化带宽数输出
- n 指定传输的字节数
- b 使用带宽数量 
- u 指定udp协议
--get-server-output 获取来自服务器端的结果

```

