# Rtp 载荷 Aac<no value>

## 0.1. 协议文档：
[rfc3640](https://tools.ietf.org/html/rfc3640)


## 0.2. 示例
```
m=audio 0 RTP/AVP 96
a=rtpmap:96 mpeg4-generic/22050
a=fmtp:96 streamtype=5;profile-level-id=1;mode=AAC-hbr;sizelength=13;indexlength=3;indexdeltalength=3;config=1210

-- config
HEX 1210 
OBJ 0001 0010 0001 0000

audioObjectType: 00010 -> 2

xxxx x010 0001 0000
samplingFrequencyIndex: 0100 -> 4

xxxx xxxx x001 0000
channelConfiguration: 0010 -> 2
```

- `config` : 共占2字节(16bit) 

|  字段   | 比特数  | 说明 |
|  ----  | ----  | ---- |
| audioObjectType  | 5 | aac的profile, 通常情况是1, 或者2|
| samplingFrequencyIndex  | 4 | aac的采样频率的索引 |
| channelConfiguration  | 4 | aac的通道数, 1~6表示为相应的通道数量, 7表示8通道|
| revsert  | 3 | 保留|

- samplingFrequencyIndex 索引表

| 值 | 表示的采样频率 |
| --- | ---|
| 0 |	96000 |
| 1 |	88200 |
| 2 |	64000 |
| 3 |	48000 |
| 4 |	44100 |
| 5 |	32000 |
| 6 |	24000 |
| 7 |	22050 |
| 8 |	16000 |
| 9 |	12000 |
| 10 |	11025 |
| 11 |	8000 |
| 12 |	7350 |
| 15 |	表示自定义的采样频率, 一般不会出现 |

