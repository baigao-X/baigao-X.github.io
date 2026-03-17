# Aac<no value>

## 0.1. 概述
AAC（Advanced Audio Coding），被认为是MP3的继任者，相对MP3有更高的压缩效率。
由Fraunhofer IIS、杜比实验室、AT&T、Sony（索尼）等公司共同开发。
1997年由MPEG正式宣布为国际标准，
MPEG-2标准的第7部分-[ISO/IEC 13818-7:1997](https://www.iso.org/standard/25040.html)。
MPEG-4标准中，AAC音频流部分在[ISO/IEC 14496-3 (subpart 4)](https://www.iso.org/standard/76383.html)中规定。



## 0.2. 帧格式
AAC音频格式有：

- `ADIF(Audio Data Interchage Format)`，音频数据交换格式：只有一个统一的头，必须得到所有数据后解码，适用于本地文件。
- `ADTS(Audio Data Transport Stream)`，音视数据传输流：每一帧都有头信息，任意帧解码，适用于传输流。

## 0.3. ADTS 格式


![[Pasted image 20230712142111.png]]

1. 一个ADTS 文件有若干个ADTS Frame组成。
2. ADTS Frame 由ADTS_Header   和AAC ES 组成

### 0.3.1. ADTS_Header 

#2.1.1.0 示例 
```
HEX: FF F1 50 80 2C 7F FC
OBJ: 1111 1111 1111 0001
     0101 0000 1000 0000
     0010 1100 0111 1111
     1111 1100
    
syncword: 0xFFF
ID: 0 -> 0
layer: 00 -> 0
protection_absent: 1 -> 1

0101 0000 1000 0000
profile: 01 -> 1

xx01 0000 1000 0000
sampling_frequency_index: 01 00 -> 4

xxxx xx00 1000 0000
private_bit: -> 0

xxxx xxx0 1000 0000
channel_configuration: 010 -> 2

xxxx xxxx xx00 0000
original/cooy:  0 -> 0

xxxx xxxx xxx0 0000
home: 0 -> 0

xxxx xxxx xxxx 0000
copyright_identification_bit: 0 -> 0
xxxx xxxx xxxx x000
copyright_identification_start: -> 0

xxxx xxxx xxxx xx00
0010 1100 0111 1111

frame_length:  0 0001 0110 0011 -> 163(Hex) -> 355(Dec)

xxxx xxxx xxxx xxxx
xxxx xxxx xxx1 1111
1111 1100
adts_buffer_fullness: 111 1111 1111 -> 7FF

xxxx xxxx xxxx xxxx
xxxx xx00
number_of_raw_data_blocks_in_frame: 0

```

- `len` ：7或9字节(取决于存不存在CRC)
- `组成`：
	- `adts_fixed_header` ： 28bit, 每一帧的内容固定不变 
	- `adts_variable_header` ： 28bit, 每一帧的内容存在不变 
	- `crc` ： 16bit


#### 0.3.1.1. adts_fixed_header 

| 字段                           | 比特数 | 说明                                                |
| ---------------------------- | --- | ------------------------------------------------- |
| syncword                     | 12  | 所有位必须为1，即0xFFF。                                   |
| ID                           | 1   | 0代表MPEG-4, 1代表MPEG-2。                             |
| layer                        | 2   | 所有位必须为0。                                          |
| protection_absent            | 1   | 1代表没有CRC，0代表有。                                    |
| **profile**                  | 2   | 配置级别, 等于 Adui Object Type -1                      |
| **sampling_frequency_index** | 4   | 标识使用的采样频率，具体见下表Table35。                           |
| **private_bit**              | 1   | see ISO/IEC 11172-3, subclause 2.4.2.3 (Table 8). |
| channel_configuration        | 3   | 通道数 取值为0时，通过inband 的PCE设置channel configuration。   |
| original/copy                | 1   | 编码时设置为0，解码时忽略。                                    |
| home                         | 1   | 编码时设置为0，解码时忽略。                                    |

- `prifile / Audio Object Type` : 

|object type ID|Audio Object Type|z|description|
|---|---|---|---|
|0|Null|||
|1|AAC Main|1999|contains AACLC|
|2|AAC LC (Low Complexity)|1999|used in the “AAC Profile”.MPEG-4 AAC LC Audio Object Type is based on the MPEG-2 Part7 Low Complexity profile(LC)combined with Perceptual Noise Subsitution(PNS)(define in MPEG-4 part 3 Subpart 4)|
|3|AAC SSR (Scalable Sample Rate)|1999|Mpeg4 AAC SSR Audio Object Type is based on the MPEG-2 Parg 7 Scalable Sampling Rate profile(SSR)combined with Perceptual Noise Subsitution(PNS)(defined in MPEG-4 Parg 3 Subpart 4)|
|4|AAC LTP (Long Term Prediction)|1999|contains AAC LC|
|5|SBR (Spectral Band Replication)|2003|used with AAC LC in the “High Efficiency AAC Profile”(HE-AAC v1)|
|...|||
- `sampling_frequency_index` :

| index | 采样频率|
|--- | ---|
| 0x0 | 96000|
| 0x1 | 88200|
| 0x2 | 64000|
| 0x3 | 48000|
| 0x4 | 44100|
| 0x5 | 32000|
| 0x6 | 24000|
| 0x7 | 22050|
| 0x8 | 16000|
| 0x9 | 12000|
| 0xa | 11025|
| 0xb | 8000|
| 0xc | reserved|
| 0xd | reserved|
| 0xe | reserved|

#### 0.3.1.2. adts_variable_header

|字段|比特数|说明|
|---|---|---|
|copyright_identification_bit|1|编码时设置为0，解码时忽略。|
|copyright_identification_start|1|编码时设置为0，解码时忽略|
|**frame_length**|13|帧长度，包括header和crc的长度，单位byte|
|adts_buffer_fullness|11|0x7FF说明是`码率可变`的码流
|number_of_raw_data_blocks_in_frame|2|number of AAC Frames(RDBs) in ADTS frame minus 1, 为了最大的兼容性通常每个ADTS frame 包含一个AAC frame。|



