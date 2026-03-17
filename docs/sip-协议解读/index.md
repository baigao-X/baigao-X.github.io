# Sip 协议解读<no value>


# 关键组件
- 用户代理: User Agent
- 注册服务器
- 代理服务器
- 重定向服务器
- 位置服务器
- 


## 分层
### 编码层
###  传输层
###  事务层
- Transaction ID

	负责SIP消息的发送和接收,确保消息按照正常顺序到达目的地，并处理消息的重传和确认超时机制
- 请求事务
- 相应事务

### 对话层
- Dialog ID
	负责管理会话的生命周期，维护会话上下文


## 信令

### REGISTER
### INVITE
### ACK
### BYE
### CANCEL
### OPTION
### INFO 
### MESSAGE


## 

SIP-URI（比如sip：123456@172.18.24.11）、Tel-URL（比如tel：+1312000）和SIPS-URI（sips：123456@172.18.24.11）