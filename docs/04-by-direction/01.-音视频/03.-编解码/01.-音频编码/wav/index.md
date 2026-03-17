# Wav<no value>

	**Waveform Audio File Format**（**WAVE**，又或者是因为文件后缀名而受大众所知的**WAV**） 
	一种典型的未压缩音频格式
	它采用`RIFF`（Resource Interchange File Format）文件格式结构。通常用来保存`PCM`格式的原始音频数据，所以通常被称为无损音频。但是严格意义上来讲，`WAV`也可以存储其它压缩格式的音频数据。

---

## 0.1. 格式
`WAV`文件遵循RIFF规则，其内容以区块（`chunk`）为最小单位进行存储。
由以下chunk组成

以下为一般都会有的
- `RIFF chunk`
- `Format chunk`
- `Data chunk`
以下为可选的
- `Fact chunk` 
- `Cue points chunk`
- `Playlist chunk`
- `Associated data list chunk`

### 0.1.1. RIFF chunk 
	12 bytes

| 名称 | 偏移地址 | 长度(bytes) | 字节序 | 内容 |
|--- |--- |--- |--- |---|
|ID |0x00 | 4| big endian| 'RIFF' (0x52494646)|
|Size |0x04 | 4| little endian| fileSize -8 (减去ID和Size的长度)| 
|Type |0x08 | 4| big endian| 'WAVE'(0x57415645)| 


### 0.1.2. FORMAT chunk
	24bytes	

| 名称 | 偏移地址 | 长度(bytes) | 字节序 | 内容 |
|--- |--- |--- |--- |---|
|ID |0x00 | 4| big endian| 'fmt ' (0x666D7420)|
|Size |0x04 | 4| little endian| 16| 
|AudioFormat |0x08 | 2| little endian| 音频格式| 
|NumChannels |0x0A | 2| little endian| 声道格式(单声道=1;立体声=2)| 
|SampleRate |0x0C | 4| little endian| 采样率(每秒块)| 
|ByteRate |0x10 | 4| little endian| 每秒数据字节数 = ByteAlign x SampleRate| 
|ByteAlign |0x14 | 2| little endian| 数据块大小(字节)| 
|BitsPerSample |0x16 | 2| little endian| 采样位数| 

#### 0.1.2.1. Audio Format 

|Code | Symbol  |Data|
|---|---|---|
|`0x0001`|`WAVE_FORMAT_PCM`|PCM|
|`0x0003`|`WAVE_FORMAT_IEEE_FLOAT`|IEEE float|
|`0x0006`|`WAVE_FORMAT_ALAW`|8-bit ITU-T G.711 A-law|
|`0x0007`|`WAVE_FORMAT_MULAW`|8-bit ITU-T G.711 µ-law|
|`0xFFFE`|`WAVE_FORMAT_EXTENSIBLE`|Determined by `SubFormat`|



### 0.1.3. DATA chunk


| 名称 | 偏移地址 | 长度(bytes) | 字节序 | 内容 |
|--- |--- |--- |--- |---|
|ID |0x00 | 4| big endian| 'data' (0x64617461)|
|Size |0x04 | 4| little endian| Data的长度 N = ByteRate * seconds| 
|Data |0x08 | N Byte| N/A| 音频数据| 
|Pad byte|0xxx | 0或1| N/A| 填充字数，若数据长度N则奇数则填充| 

## 0.2. LIST chunk 
8 byte (不包含实际Data)

	使用ffmpeg 转码生成的wav文件一般会存在一个LIST chunk

| 名称 | 偏移地址 | 长度(bytes) | 字节序 | 内容 |
|--- |--- |--- |--- |---|
|ID |0x00 | 4| big endian| 'LIST' (0x4c495354)|
|Size |0x04 | 4| little endian| Data的长度 | 
|Data |0x08 | N Byte| little endian| 比如: ` (INFOISFT...Lavf58.29.100... 26 bytes in total)` |


## 0.3. 2.示例

```

524946462468220057415645666D7420100000000100020044AC0000
10B1020004001000646174610068220000000000

RIFF chunk
52 49 46 46  //'RIFF'
24 68 22 00  // size
57 41 56 45  // 'WAVE'

FORMAT chunk
66 6D 74 20  //'fmt '
10 00 00 00  // 16
01 00        // 1-PCM 
02 00        // channle - 2
44 AC 00 00  // 44100
10 B1 02 00  // 176400
04 00        // 4 
10 00        // 2 - 16bit

data chunk
64 61 74 61   //'data'
00 68 22 00   // size
.....         // data
```


## 0.4. 文献

- [wiki](https://en.wikipedia.org/wiki/WAV)
- [麦吉尔大学# 音频文件格式规范](https://www.mmsp.ece.mcgill.ca/Documents/AudioFormats/WAVE/WAVE.html)
- IBM；微软（1991 年 8 月）[“多媒体编程接口和数据规范 1.0”](https://www.aelius.com/njh/wavemetatools/doc/riffmci.pdf)


