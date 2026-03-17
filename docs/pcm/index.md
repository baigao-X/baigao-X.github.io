# Pcm<no value>

PAM：**P**ulse **A**mplitude **M**odulation 脉冲幅度调制(数字信号过程采样)
PCM：**P**ulse **C**ode **M**odulation 脉冲编码调制

裸流

----- 


## 1. 参数：
### 1.1 Sample Rate: 采样频率：
- 8kHz(电话)
- 44.1kHz(CD)
- 48kHz(DVD)

### 1.2 Sample Size 量化位数
通常为16-bit

### 1.3 Number of Channels 通道个数
- `立体声(steroe)： 既包含左声道和右声道
- `单声道(mono)
### 1.4 Sign  是否有符号
- `有符号`:  -128~127
- `无符号`:  0~255
### 1.5 Byte Ordering 字节序
- `little-endian`
- `big-endian`
### 1.6 Inter or Floating Point 整形还是浮点数


## 2. 文件格式：
![[Pasted image 20230713155450.png]]