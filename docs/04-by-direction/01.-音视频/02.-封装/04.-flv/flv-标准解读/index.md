# Flv  标准解读<no value>


FLV（Flash Video）是一种流媒体格式，因其体积小、协议相对简单，很快便流行开来，并得到广泛的支持。
常见的HTTP-FLV直播协议，就是使用HTTP流式传输通过FLV封装的音视频数据。
而Websocket-FLV就是通过Websocket传输FLV封装的音视频数据。

- 兼容性：
	HTTP-FLV的兼容性很好，除了iOS原生浏览器不支持，其他平台和浏览器都支持了，参考[MSE](https://caniuse.com/?search=mse)。
	i若需要支持iOS浏览器，你可以考虑使用HLS或者使用WASM；注意一般iOS的Native应用，可以选择使用ijkplayer播放器。
- 与HLS区别	
	HLS本质上就是**HTTP文件下载**，而HTTP-FLV本质上是**流传输**。
	CDN对于HTTP文件下载的支持很完善，因此HLS的兼容性比HTTP-FLV 要好很多；
	同样HTTP-FLV的延迟比HLS要低很多，基本上可以做到3的5秒左右延迟，而HLS的延迟一般是8到10秒以上。

# 1. 格式说明

![[Pasted image 20240327150217.png]]


## 1.1. FLV Header
	9bytes

- Signature: 3 bytes, 'FLV'(0x 46 0x4c 0x56) 。
- version: 1bytes, 版本号，固定为1
- TypeFlags 1bytes: 
	- TypeFlagsReserved: 5bit, 保留，固定为0
	- TypeFlagsAudio: 1bit,表示这文件是否有音频
	- TypeFlagsReserved: 1bit, 保留，固定为0
	- TypeFlagsVideo: 1bit,表示这个文件是否有视频
- DataOffset : 4byte,  表示从文件头开始便宜多少是 FLV Body。因为 `FLV Header` 之后就是 `FLV Body` ，因此可以说 `DataOffset` 字段是头部的大小

## 1.2. PreivousTagSize
	4bytes
	表示上一个Tag的大小。第一个的值永远为0.

## 1.3. Tag

### 1.3.1. Tag Header

- TagType: 1bytes, 表示Tag类型
	-  Script Tag， `TagType` 等于 18，这种 Tag 只有一个，而且在开头的位置，主要是存储文件的基本信息，帧率，采样率，持续时间之类的。
	- Video Tag，`TagType` 等于 9，存放 一帧视频的数据，通常是 一帧。
	- Audio Tag，`TagType` 等于 8，存放一帧音频的数据，通常是一帧。
- DataSize: 3bytes，表示后面的 `Tag Data` 部分的大小。
- TimeStamp: 3 字节，当前这一帧视频 或者 音频的 解码时间。
- TimeStampExtended:占1 字节，解码时间扩展，因为 `TimeStamp` 字段只有 3 字节，如果存不下 `dts`（解码时间），就需要用 `TimeStampExtended` 来扩展，`TimeStampExtended` 会作为最高位的字节。
 - StreamID: 这个字段总是0，但是没有说明是干嘛的。
### 1.3.2. Tag Data

不同类型的Tag 有不同的格式
#### 1.3.2.1. Script Tag Data
`Tag Data` 里面全部都是 `AMF` 包，`AMF` 全称是 Action Message Format（信息表）。

- AMF 格式
``` cpp
aligned(8) class AMF { 
	unsigned int(8)  AMFType;     
}

typedef enum {
    AMF_TYPE_STRING   = 0x02,  
    AMF_TYPE_METADATA = 0x08, //数组包,key-value
} AMFType;


// `AMF1` 的 `type` 等于 2，代表这是一个 **`String`** 包
// String size 等于 10，代表这个字符串是 10 个字节，而 `onMetaData` 刚好就是 10 字节
aligned(8) class AMF_STRING 
	extends AMF (AMF_TYPE= 0x02){
	unsigned int(16)  string_size;     
	unsigned int(8)[string_size]  string;     
}

//`AMF2` 的 `type` 等于 8，代表这是一个 **数组** 包，
//metadata count 等于 16，代表这个数组的长度是 16，可以看到 `MetaData` 里面刚好有 16 个 Key-value 键值对。
	
aligned(8) class AMF_METADATA 
	extends AMF (AMF_TYPE= 0x08){
	unsigned int(32) metadata_count;     
	METADATA [metadata_count] metadata;
}

aligned(8) class METADATA {
	unsigned int(16) key_size;     //键名字符串长度
	unsigned int(8)[key_size]  key_name;     
	unsigned int(8) AMFDataType;   // 值类型
	unsigned int[] value;   //每个vlue占用的长度根据值类型而定
}

	
typedef enum {
    AMF_DATA_TYPE_NUMBER      = 0x00,  // value 占用8bytes，存储的是浮点数
    AMF_DATA_TYPE_BOOL        = 0x01,  // value 占用1bytes
    AMF_DATA_TYPE_STRING      = 0x02,  // 后紧跟2bytes，表示string长度
    AMF_DATA_TYPE_OBJECT      = 0x03,
    AMF_DATA_TYPE_NULL        = 0x05,
    AMF_DATA_TYPE_UNDEFINED   = 0x06,
    AMF_DATA_TYPE_REFERENCE   = 0x07,
    AMF_DATA_TYPE_MIXEDARRAY  = 0x08,
    AMF_DATA_TYPE_OBJECT_END  = 0x09,
    AMF_DATA_TYPE_ARRAY       = 0x0a,
    AMF_DATA_TYPE_DATE        = 0x0b,
    AMF_DATA_TYPE_LONG_STRING = 0x0c,
    AMF_DATA_TYPE_UNSUPPORTED = 0x0d,
} AMFDataType;
```
##### 1.3.2.1.1. MetaData
-  duration:  文件的时长，一个浮点数，单位s
- width： 视频宽度，一个浮点数
- height:  视频高度，一个浮点数
- videodatarate: 视频比特率，单位千比特/s ， 浮点数
- framerate: 帧率，浮点数
- videocodecid： 视频编码ID，浮点数
- audiodatarate： 音频比特率，浮点数，
- audiosamplerate： 音频采样率：浮点数，48000
- audiosamplesize： 音频采样位数, 浮点数，16
- stereo： 表示是否立体声， BOOL值
- audiocodecid： 音频编码ID，浮点数
- major_brand:  最推荐按照这种格式来解析， isom
- minor_version: 版本号, 1, string 类型
- compatible_brands: 兼容的格式,存在多个，中间没有符号分隔
- filesize: 表示整个flv文件的大小

### 1.3.3. Video Tag Data


``` cpp
aligned(8) class VideoTagData { 
	bit(4)  FrameType;     
	bit(4)  CodecID;     
}

aligned(8) class AVC_VideoTagData 
	extends VideoTagData (FrameType, CodecId = 7){

	//AVCVIDEOPACKET
	unsigned int(8) AVCPacketType;
	unsigned int(24) CompositionTimeOffset;
}

//配置信息
aligned(8) class ConfigAVC_VideoTagData 
	extends AVC_VideoTagData (FrameType = 1, AVCPacketType = 0){

	//AVCDecoderConfigurationRecord
	unsigned int(8) configurationVersion;
	unsigned int(8) AVCProfileIndication;
	unsigned int(8) AVCLevelIndication;
	bit(6) reserved;
	bit(2) lengthSizeMinusOne;
	bit(3) reserved;
	bit(5) numOfSequenceParameterSets; //SPS的个数
	for (int i=0; i<=numOfSequenceParameterSets; i++) {
		unsigned int(16) sequenceParameterSetLength;
		unsigned int(8)[sequenceParameterSetLength] sequenceParameterSet; //SPS
	}

	unsigned int(16) numOfPictureParameterSets; //PPS的个数
	for (int i=0; i<=numOfPictureParameterSets; i++) {
		unsigned int(16) pictureParameterSetLength;
		unsigned int(8)[pictureParameterSetLength] pictureParameterSet; //PPS
	}
}

aligned(8) class DataAVC_VideoTagData 
	extends AVC_VideoTagData (FrameType = 1,2, AVCPacketType = 1){
	bit(8)[] Data;
}

```

- CodecID: 编码器ID
    1: JPEG (currently unused)
    2: Sorenson H.263
    3 : Screen video
    4 : On2 VP6
    5 : On2 VP6 with alpha channel
    6 : Screen video version 2
    7 : AVC
	
对于 AVC(H264)

- FrameType: 帧类型
    1: keyframe (for AVC, a seekableframe), 关键帧, 实际是否是I帧，还需要根据`AVCPacketType`来判断
    2: inter frame(for AVC, a non -seekable frame) ,非关键帧
    3 : disposable inter frame(H.263only)
    4 : generated keyframe(reserved forserver use only)
    5 : video info / command frame

- AVCPacketType
	1. `AVCPacketType` 等于 0，代表后面的数据是 AVC 序列头
	2. `AVCPacketType` 等于 1，代表后面的数据是 AVC NALU 单元
	3. `AVCPacketType` 等于 2，代表 AVC 序列结束。
-  CompositionTime Offset : 时间补偿, 用来计算有B帧场景的PTS。PTS=TimeStamp(in TagHeader) + CompositionTImeOffset


### 1.3.4. Audio Tag Data

``` cpp
aligned(8) class AudioTagData { 
	bit(4)  SoundFormat;     
	bit(2)  SoundRate;       //
	bit(1)  SoundSize;       //1-16bit；0-8bit
	bit(1)  SoundType;       //1-双声道(立体声)，0-单声道
}

aligned(8) class ACC_AudioTagData 
	extends AudioTagData (SoundFormat = 10){

	//ACCAUDIODATA
	unsigned int(8) AACPacketType;
}

//配置信息
aligned(8) class ConfigAAC_AudioTagData 
	extends ACC_AudioTagData (AACPacketType = 0) {

	//Data(AudioSpecifiConfig)
	bit(5)  audioObjectType;     
	bit(4)  samplingFrequencyIndex;     
	bit(4)  channelConfiguration;     
	bit(3)  reserved;
	unsigned int(24) AOT_SpecificCOnfig
}

aligned(8) class DataAAC_AudioTagData 
	extends ACC_AudioTagData (AACPacketType = 1) {

	//Data (Raw AAC frame data)
	bit(8)[] Data; 
  }
```

- SoundFormat:  (4 bits)
    Start Offset: 469 (0x1d5)
    SoundFormat:
    1 = ADPCM
    2 = MP3
    3 = Linear PCM, little endian
    4 = Nellymoser 16 - kHz mono
    5 = Nellymoser 8 - kHz mono
    6 = Nellymoser
    7 = G.711 A - law logarithmic PCM
    8 = G.711 mu - law logarithmic PCM
    9 = reserved
    10 = AAC
    11 = Speex
    14 = MP3 8 - Khz
    15 = Device - specific sound

- SoundRate: (2 bits)
    0 = 5.5-kHz
    1 = 11 - kHz
    2 = 22 - kHz
    3 = 44 - kHz

对于AAC：
	AACPacketType:
		1. 当 `AACPacketType` 等于 0，代表这是 `AudioSpecificConfig`（序列头），`AudioSpecificConfig` 只出现在第一个 `Audio Tag` 中
		2. 当 `AACPacketType` 等于 1，代表这是 AAC Raw frame data，也就是AAC 的裸流