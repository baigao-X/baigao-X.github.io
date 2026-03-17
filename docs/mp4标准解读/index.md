# Mp4标准解读<no value>

# 1. 标准

ISO-14496 Part 12
ISO-14496 Part 14
MP4文件的格式主要由 MPEG-4 Part 12、MPEG-4 Part 14 两部分进行定义。
其中，MPEG-4 Part 12 定义了ISO基础媒体文件格式，用来存储基于时间的媒体内容。
MPEG-4 Part 14 实际定义了MP4文件格式，在MPEG-4 Part 12的基础上进行扩展。

![[Pasted image 20240326142914.png]]

## 1.1. 参考网站

- [ZigZag Sin](https://www.zzsin.com/catalog/mp4_format.html)
# 2. box 格式


1个box由两部分组成：box header、box body。

1. box header：box的元数据，比如box type、box size。
2. box body：box的数据部分，实际存储的内容跟box类型有关，比如mdat中body部分存储的媒体数据。

![[Pasted image 20240326133941.png]]

## 2.1. Box Header

字段定义如下：

- type：box类型，包括 “预定义类型”、“自定义扩展类型”，占4个字节；
    - 预定义类型：比如ftyp、moov、mdat等预定义好的类型；
    - 自定义扩展类型：如果type\==uuid，则表示是自定义扩展类型。size（或largesize）随后的16字节，为自定义类型的值（extended_type）
- size：**包含box header在内的整个box的大小**，单位是字节。当size为0或1时，需要特殊处理：
    - size等于0：box的大小由后续的largesize确定（一般只有装载媒体数据的mdat box会用到largesize）；
    - size等于1：当前box为文件的最后一个box，通常包含在mdat box中；
- largesize：box的大小，占8个字节；
- extended_type：自定义扩展类型，占16个字节；

## 2.2. Box Body

box数据体，不同box包含的内容不同，需要参考具体box的定义。有的 box body 很简单，比如 ftyp。有的 box 比较复杂，可能嵌套了其他box，比如moov。

## 2.3. Box vs FullBox

在Box的基础上，扩展出了FullBox类型。相比Box，FullBox 多了 version、flags 字段。

- version：当前box的版本，为扩展做准备，占1个字节；
- flags：标志位，占24位，含义由具体的box自己定义；

# 3. 常用box 介绍

## 3.1. ftyp / File Type Box
	顶层box
	描述文件遵从的MP4规范与版本

``` cpp
//body 数据格式
aligned(8) class FileTypeBox extends Box(‘ftyp’) { 
	unsigned int(32) major_brand; 
	unsigned int(32) minor_version; 
	unsigned int(32) compatible_brands[]; // to end of the box 
}
```

isom（ISO Base Media file）是在 MPEG-4 Part 12 中定义的一种基础文件格式，MP4、3gp、QT 等常见的封装格式，都是基于这种基础文件格式衍生的。
MP4 文件可能遵循的规范有mp41、mp42，而mp41、mp42又是基于isom衍生出来的。
更多brand 类型参考[这里](http://fileformats.archiveteam.org/wiki/Boxes/atoms_format#Brands)
## 3.2. moov / Movie Box
	顶层box
	媒体的metadata信息，有且仅有一个。
	一般位于mp4文件的开头。
	moov中，最重要的两个box是 mvhd 和 trak：
	
## 3.3. mvhd / Movie Header Box
	mp4文件的整体信息，比如创建时间、文件时长等

``` cpp

//body 数据格式,注意是full box，存在version和 flag
aligned(8) class MovieHeaderBox extends FullBox(‘mvhd’, version, 0) { 
	if (version==1) {
      unsigned int(64)  creation_time;     
      unsigned int(64)  modification_time; 
      unsigned int(32)  timescale;
      unsigned int(64)  duration;
   } else { // version==0
      unsigned int(32)  creation_time;
      unsigned int(32)  modification_time;
      unsigned int(32)  timescale;
      unsigned int(32)  duration;
	}
	template int(32) rate = 0x00010000; // typically 1.0
	template int(16) volume = 0x0100; // typically, full volume const bit(16) reserved = 0;
	const unsigned int(32)[2] reserved = 0;
	template int(32)[9] matrix = {0x00010000,0,0,0,0x00010000,0,0,0,0x40000000 };
    // Unity matrix
	bit(32)[6]  pre_defined = 0;
	unsigned int(32)  next_track_ID;
}

```

- creation_time：文件创建时间（相对于UTC时间1904-01-01零点的秒数）；
- modification_time：文件修改时间（相对于UTC时间1904-01-01零点的秒数）；
- timescale：一秒包含的时间单位（整数）。举个例子，如果timescale等于1000，那么，一秒包含1000个时间单位（后面track等的时间，都要用这个来换算，比如track的duration为10,000，那么，track的实际时长为10,000/1000=10s）；
- duration：影片时长（整数），根据文件中的track的信息推导出来，等于时间最长的track的duration；
- rate：推荐的播放速率，32位整数，高16位、低16位分别代表整数部分、小数部分（[16.16]），举例 0x0001 0000 代表1.0，正常播放速度；
- volume：播放音量，16位整数，高8位、低8位分别代表整数部分、小数部分（[8.8]），举例 0x01 00 表示 1.0，即最大音量；
- matrix：视频的转换矩阵，一般可以忽略不计；
- next_track_ID：32位整数，非0，一般可以忽略不计。当要添加一个新的track到这个影片时，可以使用的track id，必须比当前已经使用的track id要大。也就是说，添加新的track时，需要遍历所有track，确认可用的track id；
## 3.4. trak / Track Box
	一个mp4可以包含一个或多个轨道（比如视频轨道、音频轨道），轨道相关的信息就在trak里。
	trak是container box，至少包含两个box，tkhd、mdia；


## 3.5. tkhd (单个trak) / Track Header Box

``` cpp
//body 数据格式,注意是full box，存在version和 flag
aligned(8) class TrackHeaderBox 
  extends FullBox(‘tkhd’, version, flags){ 
	if (version==1) {
	      unsigned int(64)  creation_time;
	      unsigned int(64)  modification_time;
	      unsigned int(32)  track_ID;
	      const unsigned int(32)  reserved = 0;
	      unsigned int(64)  duration;
	   } else { // version==0
	      unsigned int(32)  creation_time;
	      unsigned int(32)  modification_time;
	      unsigned int(32)  track_ID;
	      const unsigned int(32)  reserved = 0;
	      unsigned int(32)  duration;
	}
	const unsigned int(32)[2] reserved = 0;
	template int(16) layer = 0;
	template int(16) alternate_group = 0;
	template int(16) volume = {if track_is_audio 0x0100 else 0}; const unsigned int(16) reserved = 0;
	template int(32)[9] matrix= { 0x00010000,0,0,0,0x00010000,0,0,0,0x40000000 }; // unity matrix
	unsigned int(32) width;
	unsigned int(32) height;
}
```

- version：tkhd box的版本；
- flags：按位或操作获得，默认值是7（0x000001 | 0x000002 | 0x000004），表示这个track是启用的、用于播放的 且 用于预览的。
    - Track_enabled：值为0x000001，表示这个track是启用的，当值为0x000000，表示这个track没有启用；
    - Track_in_movie：值为0x000002，表示当前track在播放时会用到；
    - Track_in_preview：值为0x000004，表示当前track用于预览模式；
- creation_time：当前track的创建时间（相对于UTC时间1904-01-01零点的秒数）；
- modification_time：当前track的最近修改时间（相对于UTC时间1904-01-01零点的秒数）；
- track_ID：当前track的唯一标识，不能为0，不能重复；
- duration：当前track的完整时长（需要除以timescale得到具体秒数）；
- layer：视频轨道的叠加顺序，数字越小越靠近观看者，比如1比2靠上，0比1靠上；
- alternate_group：当前track的分组ID，alternate_group值相同的track在同一个分组里面。同个分组里的track，同一时间只能有一个track处于播放状态。当alternate_group为0时，表示当前track没有跟其他track处于同个分组。一个分组里面，也可以只有一个track；
- volume：audio track的音量，介于0.0~1.0之间；
- matrix：视频的变换矩阵；
- width、height：视频的宽高(16.16浮点)；

**PS: mp4info 不会显示width和height字段**，注意识别

## 3.6. mdia
mdia Box 可以看到官方的定义也是一个比较简单的 Box 相当于一个分隔符，重要的信息都在 Data 区，里面包含了一些重要的 Box，所以该 Box 是 Full Box。

## 3.7. mdhd Box / Media Header Box
这个 Box 是 Full Box，意味着Box Header 有 Version 和 Flag 字段，该 Box 里面主要定义了该 Track 的媒体头信息，其中我们最关心的两个字段是 Time scale 和 Duration，分别表示了该 Track 的时间戳和时长信息，这个时间戳信息也是PTS 和 DTS 的单位

``` cpp
//body 数据格式,注意是full box，存在version和 flag
aligned(8) class TrackHeaderBox 
  extends FullBox(‘mdhd’, version, 0){ 
	if (version==1) {
	      unsigned int(64)  creation_time;
	      unsigned int(64)  modification_time;
	      unsigned int(32)  timescale;
	      unsigned int(64)  duration;
	   } else { // version==0
	      unsigned int(32)  creation_time;
	      unsigned int(32)  modification_time;
	      unsigned int(32)  timescale;
	      unsigned int(32)  duration;
	}
	bit(1) pad = 0;
	unsigned int(5)[3] language;  //ISO-639-2/T language code
	unsigned int(16) pre_defined = 0;
}
```

- creation_time：当前track的创建时间（相对于UTC时间1904-01-01零点的秒数）；
- modification_time：当前track的最近修改时间（相对于UTC时间1904-01-01零点的秒数）；
- timescale :Track的时间计算单位
- duration：当前track的完整时长（需要除以timescale得到具体秒数）；
- pad 
- language: 媒体语言码，参考标准ISO639-2/T
- qualiiy(pre_defined) 媒体回放质量，默认为00即可

***PS: mp4info 工具会少显示数据**

## 3.8. hdlr Box / Handler Reference Box
	声明当前track的类型，以及对应的处理器（handler）

``` cpp
//body 数据格式,注意是full box，存在version和 flag
aligned(8) class HandlerBox extends FullBox(‘hdlr’, version = 0, 0) { 
	unsigned int(32) pre_defined = 0;
	unsigned int(32) handler_type;
	const unsigned int(32)[3] reserved = 0;
   	string   name;
}
```

- handler_type的取值包括：
	- vide（0x76 69 64 65），video track；
	- soun（0x73 6f 75 6e），audio track；
	- hint（0x68 69 6e 74），hint track；
- name: handler名称
## 3.9. minf  / Media Information Box
	 一个Container Box，简单理解就是一个分隔符，重点都在子Box里
	不是 Full Box

## 3.10. vmhd /Video Media Header Box
该Box为Full Box意味着 Box Header 有四字节的 version 和 Flag 字段。
这个 Box 的大小是固定的， Vmhd 大小是 20 字节，一般都是用默认值填充。
``` cpp
//body 数据格式,注意是full box，存在version和 flag
aligned(8) class VideoMediaHeaderBox extends FullBox(‘vmhd’, version = 0, 1) { 
	template unsigned int(16) graphicsmode = 0; // copy, see below
	template unsigned int(16)[3] opcolor = {0, 0, 0};
}
```

- graphicsmode: 视频合成模式，为0时拷贝原始图像，否则与opcolor进行合成. 默认为0
- Opcolor:  颜色值，RGB颜色值。默认为0.

## 3.11. smhd /Sound Media Header Box
该Box为Full Box意味着 Box Header 有四字节的 version 和 Flag 字段。
这个 Box 的大小是固定的， Smhd 大小是 16 字节，一般都是用默认值填充。

``` cpp
//body 数据格式,注意是full box，存在version和 flag
aligned(8) class SoundMediaHeaderBox extends FullBox(‘smhd’, version = 0, 0) { 
	template unsigned int(16) balance = 0; // copy, see below
	template unsigned int(16) reserved = 0;
}
```

- Balance: 声音的均衡. 默认为0

## 3.12. dinf
	 一个Container Box，一般用来定位媒体信息。
一般会包含一个 **Dref Box**，Dref 下面会有若干个 **Url Box** 或者也叫 **Urn Box**，这些 Box 组成一个表，用来定位 Track 的数据。
Track 可以被分成若干个段，每一段都可以根据 **Url** 或者 **Urn** 指向的地址来获取数据，sample 描述中会用这些片段的序号将这些片段组成一个完整的 track，一般情况下当数据完全包含在文件中，Url 和 Urn Box 的字符串是空的。

这个 Box 存在的意义就是允许 MP4 文件的媒体数据分开最后还能进行恢复合并操作，但是实际上，Track 的数据都保存在文件中，所以该字段的重要性还体现不出来。


## 3.13. Dref / Data Reference Box 

``` cpp
//body 数据格式,注意是full box，存在version和 flag
//url
aligned(8) class DataEntryUrlBox (bit(24) flags)
	extends FullBox(‘smhd’, version = 0, flags) { 
	string location;
}

//urn
aligned(8) class DataEntryUrlnBox (bit(24) flags)
	extends FullBox(‘smhd’, version = 0, flags) { 
	string name;
	string location;
}

aligned(8) class DataReference 
	extends FullBox(‘smhd’, version = 0, 0) { 
	unsigned int(32) entry_count;
	for (i=1; i * entry_count; i++) {
		DataEntryBox(entry_version, entry_flag) data_entry;	
	}
}

```

- DataReference
	- entry count 下面URL的数目
- DataEntryUrl
	- url 中的flags：  flags 值为1，则表明 “url” 中的字符串为空，表示 track 数据已包含在文件中，所以 Url 的 Url Box Data 部分为空。


## 3.14. stbl / Sample Table Box

最关键的 Stbl Box.
MP4文件的媒体数据部分在mdat box里，而stbl则包含了这些媒体数据的索引以及时间信息，了解stbl对解码、渲染MP4文件很关键。
这个 Box 主要核心作用就是把音视频的媒体数据和时间进行映射，可以根据这 Box 里面的信息，实时定位到视频各个时间段的数据。
这个 Box 通常包含以下子 Box：

- 采样描述容器： Sample Description 即 Stsd Box。给出视频、音频的编码、宽高、音量等信息，以及每个sample中包含多少个frame
- 采样时间容器： Time To Sample 即 Stts Box
- 播放时间容器： (composition)Time To Sample 即 Ctts Box
- Chunk 采样容器: Sample To Chunk 即 Stsc。
- 采样大小容器： Sample Size 即 Stsz
- Chunk 偏移容器：Chunk Offest 即 Stco。。
- 采样同步容器： Sync Sample 即 Stss


### 3.14.1. stsd (Sample Description Box）
该 Box 存储了编码类型和初始化解码器需要的信息，与特定的 Track Type 有关，根于不同的 Track 使用不一样的编码标准。
给出视频、音频的编码、宽高、音量等信息，以及每个sample中包含多少个frame等信息。

``` cpp


// VisualSampleEntry、AudioSampleEntry、HintSampleEntry 见后面定义

aligned(8) class SampleDescriptionBox (unsigned int(32) handler_type) extends FullBox('stsd', 0, 0){
	unsigned int(32) entry_count;
	for (int i = 1 ; i <= entry_count ; i++) {
	      switch (handler_type){
	        case ‘soun’: // for audio tracks
				AudioSampleEntry();
				break;
			case ‘vide’: // for video tracks
			   VisualSampleEntry();
			   break;
			case ‘hint’: // Hint track
			   HintSampleEntry();
			   break;	         
		}
	}
}

```

- SampleDescriptionBox
	- entry_count : 表示ample description 数目

#### 3.14.1.1. avc1
  编码数据是H264则这个Box是Avc1 Box内部含有音视频是新


``` cpp
aligned(8) abstract class SampleEntry (unsigned int(32) format) extends Box(format){
	const unsigned int(8)[6] reserved = 0;
	unsigned int(16) data_reference_index;
}

class VisualSampleEntry(codingname) extends SampleEntry (codingname){ 
	unsigned int(16) pre_defined = 0;
	const unsigned int(16) reserved = 0;
	unsigned int(32)[3] pre_defined = 0;
	unsigned int(16) width;
	unsigned int(16) height;
	template unsigned int(32) horizresolution = 0x00480000; // 72 dpi 
	template unsigned int(32) vertresolution = 0x00480000; // 72 dpi 
	const unsigned int(32) reserved = 0;
	template unsigned int(16) frame_count = 1;
	string[32] compressorname;
	template unsigned int(16) depth = 0x0018;
	int(16) pre_defined = -1;
}
```

 - data_reference_index：当MP4文件的数据部分，可以被分割成多个片段，每一段对应一个索引，并分别通过URL地址来获取，此时，data_reference_index 指向对应的片段（比较少用到）；
- width、height：视频的宽高，单位是像素；
- horizresolution、vertresolution：水平、垂直方向的分辨率（像素/英寸），16.16定点数，默认是0x00480000（72dpi）；
- frame_count：一个sample中包含多少个frame，对video track来说，默认是1；
- compressorname：仅供参考的名字，通常用于展示，占32个字节，比如 AVC Coding。第一个字节，表示这个名字实际要占用N个字节的长度。第2到第N+1个字节，存储这个名字。第N+2到32个字节为填充字节。compressorname 可以设置为0；
- depth：位图的深度信息，比如 0x0018（24），表示不带alpha通道的图片；


##### 3.14.1.1.1. avcC
	
	 这个Box包含真实的H264码流 SPS和PPS信息
``` cpp

aligned(8) class AvcCBox extends Box(‘avcC’) { 
	unsigned int(8) configuratoin_version = 0x01;
	unsigned int(8) avc_profile_indication;
	unsigned int(8) profile_compatibility;
	unsigned int(8) avc_level_indication;
	bit(6) reserved;
	bit(2) length_size_minus_one;
	bit(3) reserved;
	bit(5) number_of_sps;
	unsigned int(16) sps_length;
	unsigned int(8)[sps_length] sps;
	bit(3) reserved;
	bit(5) number_of_pps;
	unsigned int(16) pps_length;
	unsigned int(8)[pps_length] pps;
}

```

- configuration version : 保留位默认0x01  
- avc profile indication : H.264编码配置：0x64表示Baseline，0x4D表示Main
- profile compatibility  : 
- avc level indication   :H.264编码级别  
- reserved  :             （6bit）默认都是1
- length size minus one  : （2 bit）读出的每个packet的前几字节代表数据大小 
- reserved  :（3 bit）
- number of sps  : (5 bit）sps的个数，一般情况是1  
- sps length    
- sps          
- reserved     
- number of pps :（5 bit）pps的个数，一般情况是1             
- pps length   
- pps   


#### 3.14.1.2. mp4a
Audio Track 下的Box

``` cpp
aligned(8) class mp4aBox extends Box(‘avcC’) { 
	unsigned int(8)[6] reserve;
	unsigned int(16) data_reference; 
	unsigned int(8)[8] reserver;
	unsigned int(16) channelcount; 
	unsigned int(16) samplesize; 
	unsigned int(8)[4] reserved;
	unsigned int(32) samplerte; 
} 
```

- reserve: 保留，默认为0
- data_reference_ID : 应用参考 DrefBox
- reserver  : 保留，默认为0
- channelcount  : 声道数，默认值2
- samplesize  :样本数，默认值16
- reserverd  : 保留，默认为0
- samplerate  :采样率（高位）

##### 3.14.1.2.1. esds 

该 Box 则包含了音频的编码信息和音频码率信息，所以解码音频时非常关键。
Esds中可以分为三层，每层为包含关系，分别为 MP4ESDescr(ed)，MP4DecConfigDescr(dcd)，MP4DecSpecificDescr(dsid)

``` cpp
aligned(8) class esdsBox extends FullBox(‘esds’, 0 , 0) { 
	unsigned int(16) es_description_tag = 0x03; //ed_tag
	unsigned int(24) ed_tag_mask = {0x80, 0x80, 0x80};  //作为分隔符号，可能不存在
	unsigned int(8) ed_tag_size;
	unsigned int(16) ed_track_id;
	unsigned int(8) ed_flag;

	unsigned int(8) decoder_config_descriptor_tag = 0x04; //dcd_tag
	unsigned int(24) dcd_tag_mask = {0x80, 0x80, 0x80};  //作为分隔符号，可能不存在
	unsigned int(8) dcd_tag_size;
	unsigned int(8) dcd_mepg_4_audio;
	unsigned int(8) dcd_audio_stream = 0x15;
	unsigned int(24) dcd_buffersize_db;
	unsigned int(32) dcd_max_birrate;
	unsigned int(32) dcd_avg_birrate;

	unsigned int(8) decoder_specific_info = 0x05;          //dsid_tag
	unsigned int(24) dsid_tag_mask = {0x80, 0x80, 0x80};  //作为分隔符号，可能不存在
	unsigned int(8) dsid_tag_size;
	//dsid_audio_specific_config 16bit
	bit(5) asc_object_type;
	bit(4) asc_frequency_index;
	bit(4) asc_channel_configuration;
	bit(1) asc_frame_length_flag;
	bit(1) asc_depends_on_core_coder;
	bit(1) asc_extesion_flag;
}
```
- es_description_tag : 基本流描述标记：默认0x03
	- ed_tag_szie :   esds box在该字段以及该字段之后的字节数(包含当前字节)
	- ed_track_id  : 表示音频的原始es数据的id是0，一般一路音频，这个值就默认是0；
	- ed_flag :
		一般默认00:0x00:00000000
		 其中每个bit还代表是否后面有相应的字段。
		 第一bit为1，则有16bit的dependOn_ES_IS字段；
		 第二bit为1，则有8bit的URL ing字段；
		 第三bit为1，则有16bit的OCR_ES_ID字段；
		 最后5bit，代表streamPriority
		 
- decoder_onfig_descriptor_tag :默认值0x04  
	- dcd_tag_size  长度   (decoder_onfig_descriptor_tag开始到完整的dsid包含的长度?? 不太确定）
	- dcd_mepg_4_audio  :0x40 是 Audio ISO/IEC 14496-3   
	- dcd_audio_stream: 一般默认0x15 
	- dcd_buffersize_db : 建议的解码器缓存大小
	- dcd_max_bitrate : 音频数据最大码率 
	- dcd_avg_bitrate  :音频数据平均码率
	
- decoder_specific_info_description_tag :码规格标记，默认值：0x05 
	- dsid_tag_szie :  解码规格标记及其后面值大小 
	- dsid_audio_specific_config(asc) :    音频规格数据，见下面各个bit位解释
		- asc object type  :     5bit
			 0: Null
			 1: AAC Main
			 2: AAC LC (Low Complexity)
			 3: AAC SSR (Scalable Sample Rate)
		- asc_frequency_index :  4bit
 			0: 96000 Hz
 			1: 88200 Hz
 			2: 64000 Hz
 			3: 48000 Hz
 			4: 44100 Hz
 			5: 32000 Hz
 			6: 24000 Hz
 			7: 22050 Hz
 			8: 16000 Hz
 			9: 12000 Hz
 			10: 11025 Hz
 			11: 8000 Hz
 			12: 7350 Hz
 			13: Reserved
 			14: Reserved
 			15: frequency is written explictly
		- asc channel configuration  ;  4bit，双声道 
			 0: Defined in AOT Specifc Config
			 1:channel: front-center 单声道
			 2: 2channels:front-left, front-right 双声道
			 3: 3channels:front-center,front-left, front-right 3声道
		- asc_frame_length_flag  :  1bit，1024 samples，每个包的大小为 1024字节 也就是一帧音频的大小。
			 0: Each packet contains 1024 samples
			 1: Each packet contains 960 samples
		- asc_depends_on_core_coder :1bit，不太重要 
		- asc_extesion_flag : 1bit，不太重要 
参照: https://wiki.multimedia.cx/index.php?title=MPEG-4_Audio

### 3.14.2. stts (TimeToSample Box)

这个 Box 是记录每一次采样出来的 sample 的时间，也可以理解为解码当前 sample 的解码时间DTS，所以这个 Bos 主要是用来找到任意时间的 sample。
Stts Box 这个表格有两项值，其中一项是连续的样点数目即 sample count 和样点相对时间差值，即 sample delta 即表格中每个条目提供了在同一个时间偏移量里面连续的 sample 序号以及 sample 偏移量。
Sample delta简单可以理解为当前这个 sample 的时长，也就是采样了一次过后多久再采样一次，所以把全部 sample 的 sample delta 加起来就是当前这个 track 的总时长。
大家把这个时间理解为该帧数据当前的解码时间即可，计算公式： DT(n+1) = DT(n) + STTS(n)


``` cpp

aligned(8) class TimeToSampleBox extends FullBox(’stts’, version = 0, 0) {
	unsigned int(32)  entry_count;
	for (int i=1; i <= entry_count; i++) {
		unsigned int(32)  sample_count;
		unsigned int(32)  sample_delta;
	}
}

```

- entry count: 描述了下面采样 sample 的信息个数
- sample_count：单个entry中，具有相同时长（duration 或 sample_delta）的连续sample的个数。
- sample_delta：sample的时长（以timescale为计量）

### 3.14.3. stss（Sync Sample Box）


I 帧是一个视频解码的关键，当视频进行 seek 操作和清晰度切换，首先就要找到当前时间点附近的 I 帧，只有拿到 I 帧才能开始解码。
所以这个 Box 可以说挺简单的，里面存的数据是对应 Sample 的 I 帧。当然音频是不存在这个 Box 的。

``` cpp

aligned(8) class SyncSampleBox (unsigned int(32) handler_type) extends FullBox('stts', 0, 0){
	unsigned int(32) entry_count;
	for (int i = 1 ; i <= entry_count ; i++) {
		unsigned int (32) sample_number; //??? 与实际查出来的不同 
	}
}
```

- entry count: 描述了下面采样 sample 的信息个数
- sample_number：关键帧对应的sample的序号；（从1开始计算）


### 3.14.4. stsz  (SampleSizeBox )

表示每个 Sample 的 Size 的大小。
用于我们在文件读取和解析 Sample 的关键可以找到任意 Sample 对应在文件中的位置。

``` cpp
aligned(8) class SampleSizeBox extends FullBox(‘ctts’, version = 0, 0) { 
	unsigned int(32) sample_size;
	unsigned int(32) sample_count;
	if (sample_size == 0) {
		for (int i=1; i <= sample_count; i++) {
		    unsigned int(32)  entry_size;
		}
	}
}

```

- sample_size: 0表示每一个sample大小不一样，不为0则每一个sample大小为当前值
- sample_count：当前 Track 下有多少个 sample。
- entry_size：对应每一个Sample的大小


### 3.14.5. stso

这个 Box 包含的信息是对应 Chunk 的在文件里的偏移量，然后根据其它表的关联关系就可以读取每个 Sample 的数据。

``` cpp
# Box Type: ‘stco’, ‘co64’
# Container: Sample Table Box (‘stbl’) Mandatory: Yes
# Quantity: Exactly one variant must be present

aligned(8) class ChunkOffsetBox
	extends FullBox(‘stco’, version = 0, 0) { 
	unsigned int(32) entry_count;
	for (i=1; i u entry_count; i++) {
		unsigned int(32)  chunk_offset;
	}
}

aligned(8) class ChunkLargeOffsetBox
	extends FullBox(‘co64’, version = 0, 0) { 
	unsigned int(32) entry_count;
	for (i=1; i u entry_count; i++) {
		unsigned int(64)  chunk_offset;
	}
}

```

- entry_count：chunk 总个数 
- chunk_offset： 每个chunk在文件中的偏移量(这里的偏移量不是相对的，都是与开头的偏移量)


### 3.14.6. stsc (SampleToChunkBox)

上面我们介绍了 Stsz Stco 这2个 Box，分别可以知道 Chunk 的偏移量，又可以知道在 Chunk 里面的各个 Sample 的大小，但是现在还缺一样，就是不知道一个 Chunk 里面包含了多少个 Sample，
这个 Box 提供的就是 Chunk 包含 Sample 的信息。
综上所述媒体数据被分为若干个 Chunk, Chunk 可以有不同的大小，同一个 Chunk 中的样点 Sample 也允许有不同的大小。

``` cpp
aligned(8) class SampleToChunkBox extends FullBox(‘stsc’, version = 0, 0) { 
	unsigned int(32) entry_count;
	for (int i=1; i <= entry_count; i++) {
	    unsigned int(32)  first_chunk;
	    unsigned int(32)  samples_per_chunk;
	    unsigned int(32)  samples_description_index;
   }
}
```

- entry count :Chunk 数量 
- first chunk : Chunk Index 
- sample_per_chunk : 每一个 chunk 里面包含多少的 sample  
- sample_description_index :每一个 sample 的描述。一般可以默认设置为 1

### 3.14.7. ctts（Composition Time to Sample Box）

从解码（dts）到渲染（pts）之间的差值。
对于只有I帧、P帧的视频来说，解码顺序、渲染顺序是一致的，此时，ctts没必要存在。
对于存在B帧的视频来说，ctts就需要存在了。当PTS、DTS不相等时，就需要ctts了，公式为 CT(n) = DT(n) + CTTS(n) 。

``` cpp
aligned(8) class CompositionOffsetBox extends FullBox(‘ctts’, version = 0, 0) { 
	unsigned int(32) entry_count;
	for (int i=1; i <= entry_count; i++) {
      unsigned int(32)  sample_count;
      unsigned int(32)  sample_offset;
   }
}

```

- entry count: 描述了下面采样 sample 的信息个数
- sample_count：连续相同 offset 的sample个数。
- sample_delta：DTS和PTS之间的偏移量



## 3.15. mdat / Media Data Box
	顶层box
	存放实际的媒体数据，一般有多个。 

``` cpp
aligned(8) class MediaDataBox extends Box(‘mdat’) { 
	bit(8) data[];
}

```