# Fmp4标准解读<no value>

fMP4 跟普通 mp4 基本文件结构是一样的。普通mp4用于点播场景，fmp4通常用于直播场景。
它们有以下差别：
- 普通mp4的时长、内容通常是固定的。fMP4 时长、内容通常不固定，可以边生成边播放；
- 普通mp4完整的metadata都在moov里，需要加载完moov box后，才能对mdat中的媒体数据进行解码渲染；
- fMP4中，媒体数据的metadata在moof box中，moof 跟 mdat （通常）结对出现。moof 中包含了sample duration、sample size等信息，因此，fMP4可以边生成边播放；
# 1. 标准

![[Pasted image 20240327113233.png]]
![[Pasted image 20240327140903.png]]
# 2. 常用box 介绍

## 2.1. mvex
**当存在mvex时，表示当前文件是fmp4（非严谨）**。
此时，sample相关的metadata不在moov里，需要通过解析moof box来获得。

### 2.1.1. mehd (Movie Extends Header Box)

mehd是可选的，用来声明影片的完整时长（fragment_duration）。
如果不存在，则需要遍历所有的fragment，来获得完整的时长。对于fmp4的场景，fragment_duration一般没办法提前预知。

```cpp
aligned(8) class MovieExtendsHeaderBox extends FullBox(‘mehd’, version, 0) {
	if (version==1) {
		unsigned int(64)  fragment_duration;
	} else { // version==0
		unsigned int(32)  fragment_duration;
	}
}

```


### 2.1.2. trex（Track Extends Box）

用来给 fMP4 的 sample 设置各种默认值，比如时长、大小等。

``` cpp
aligned(8) class TrackExtendsBox extends FullBox(‘trex’, 0, 0){ 
	unsigned int(32) track_ID;
	unsigned int(32) default_sample_description_index; 
	unsigned int(32) default_sample_duration;
	unsigned int(32) default_sample_size;
	unsigned int(32) default_sample_flags
}
```


- track_id：对应的 track 的 ID，比如video track、audio track 的ID；
- default_sample_description_index：sample description 的默认 index（指向stsd）；
- default_sample_duration：sample 默认时长，一般为0；
- default_sample_size：sample 默认大小，一般为0；
- default_sample_flags：sample 的默认flag，一般为0；
	- reserved：4 bits，保留位；
	- is_leading：2 bits，是否 leading sample，可能的取值包括：
	    - 0：当前 sample 不确定是否 leading sample；（一般设为这个值）
	    - 1：当前 sample 是 leading sample，并依赖于 referenced I frame 前面的 sample，因此无法被解码；
	    - 2：当前 sample 不是 leading sample；
	    - 3：当前 sample 是 leading sample，不依赖于 referenced I frame 前面的 sample，因此可以被解码；
	- sample_depends_on：2 bits，是否依赖其他sample，可能的取值包括：
	    - 0：不清楚是否依赖其他sample；
	    - 1：依赖其他sample（不是I帧）；
	    - 2：不依赖其他sample（I帧）；
	    - 3：保留值；
	- sample_is_depended_on：2 bits，是否被其他sample依赖，可能的取值包括：
	    - 0：不清楚是否有其他sample依赖当前sample；
	    - 1：其他sample可能依赖当前sample；
	    - 2：其他sample不依赖当前sample；
	    - 3：保留值；
	- sample_has_redundancy：2 bits，是否有冗余编码，可能的取值包括：
	    - 0：不清楚是否存在冗余编码；
	    - 1：存在冗余编码；
	    - 2：不存在冗余编码；
	    - 3：保留值；
	- sample_padding_value：3 bits，填充值；
	- sample_is_non_sync_sample：1 bits，不是关键帧；
	- sample_degradation_priority：16 bits，降级处理的优先级（一般针对如流传过程中出现的问题）；
## 2.2. mvhd 

- duration：影片时长不再有效，一般是0


## 2.3. moof (Movie Fragment Box)

moof是个container box，相关 metadata 在内嵌box里，比如 mfhd、 tfhd、trun 等。
fmp4文件分为若干对**moof**和**mdat** 

### 2.3.1. mfhd (Movie Fragment Header Box)

``` cpp 
aligned(8) class MovieFragmentHeaderBox extends FullBox(‘mfhd’, 0, 0){
	unsigned int(32)  sequence_number;
}
``` 

- sequence_number : movie fragment 的序列号。根据 movie fragment 产生的顺序，从1开始递增。

### 2.3.2. traf (Track Fragment Box)

对 fmp4 来说，数据被分为多个 movie fragment。
一个 movie fragment 可包含多个track fragment（每个 track 包含0或多个 track fragment）。
每个 track fragment 中，可以包含多个该 track 的 sample。
每个traf 由 tfhd、tfdt、trun组成
#### 2.3.2.1. tfhd (Track Fragment Header Box）
	tfhd 用来设置 track fragment 中 的 sample 的 metadata 的默认值。

``` cpp
//以下均为可选字段，根据tf_flags 决定是否存在对应字段 
aligned(8) class TrackFragmentHeaderBox extends FullBox(‘tfhd’, 0, tf_flags){
	unsigned int(32) track_ID;
	// all the following are optional fields 
	unsigned int(64) base_data_offset; 
	unsigned int(32) sample_description_index; 
	unsigned int(32) default_sample_duration; 
	unsigned int(32) default_sample_size; 
	unsigned int(32) default_sample_flags
}
```
-  tf_flags:
	- 0x000001 base‐data‐offset‐present：存在 base_data_offset 字段，表示 数据位置 相对于整个文件的 基础偏移量。
	- 0x000002 sample‐description‐index‐present：存在 sample_description_index 字段；
	- 0x000008 default‐sample‐duration‐present：存在 default_sample_duration 字段；
	- 0x000010 default‐sample‐size‐present：存在 default_sample_size 字段；
	- 0x000020 default‐sample‐flags‐present：存在 default_sample_flags 字段；
	- 0x010000 duration‐is‐empty：表示当前时间段不存在sample，default_sample_duration 如果存在则为0 ，；
	- 0x020000 default‐base‐is‐moof：如果 base‐data‐offset‐present 为1，则忽略这个flag。如果 base‐data‐offset‐present 为0，则当前 track fragment 的 base_data_offset 是从 moof 的第一个字节开始计算；
sample 位置计算公式为 base_data_offset + data_offset，其中，data_offset 每个 sample 单独定义。如果未显式提供 base_data_offset，则 sample 的位置的通常是基于 moof 的相对位置。


#### 2.3.2.2. trun（Track Fragment Run Box）

track run 表示一组连续的 sample

``` cpp
aligned(8) class TrackRunBox extends FullBox(‘trun’, version, tr_flags) {
   unsigned int(32)  sample_count;
   // the following are optional fields
   signed int(32) data_offset;
   unsigned int(32)  first_sample_flags;

   // all fields in the following array are optional
   {
      unsigned int(32)  sample_duration;
      unsigned int(32)  sample_size;
      unsigned int(32)  sample_flags
      if (version == 0) { 
	      unsigned int(32) sample_composition_time_offset; 
	  } else { 
		  signed int(32) sample_composition_time_offset; 
	  }
   }[ sample_count ]
}

```

- sample_count：sample 的数目；
- data_offset：数据部分的偏移量；
- first_sample_flags：可选，针对当前 track run中 第一个 sample 的设置；
- tr_flags 如下，大同小异：
	- 0x000001 data‐offset‐present：存在 data_offset 字段；
	- 0x000004 first‐sample‐flags‐present：存在 first_sample_flags 字段，这个字段的值，只会覆盖第一个 sample 的flag设置；当 first_sample_flags 存在时，sample_flags 则不存在；
	- 0x000100 sample‐duration‐present：每个 sample 都有自己的 sample_duration，否则使用默认值；
	- 0x000200 sample‐size‐present：每个 sample 都有自己的 sample_size，否则使用默认值；
	- 0x000400 sample‐flags‐present：每个 sample 都有自己的 sample_flags，否则使用默认值；
	- 0x000800 sample‐composition‐time‐offsets‐present：每个 sample 都有自己的 sample_composition_time_offset；
- 0x000004 first‐sample‐flags‐present，覆盖第一个sample的设置，这样就可以把一组sample中的第一个帧设置为关键帧，其他的设置为非关键帧；


## 2.4. mfra (Movie Fragment Random Access Box)
电影片段随机存取
内部由**mfro**和若干个**tfra**组成


### 2.4.1. tfra

``` cpp
aligned(8) class TrackFragmentRandomAccessBox  
extends FullBox(‘tfra’, version, 0) {  
	unsigned int(32) track_ID; 
	const unsigned int(26) reserved = 0; 
	unsigned int(2) length_size_of_traf_num;
	unsigned int(2) length_size_of_trun_num;
	unsigned int(2) length_size_of_sample_num;
	unsigned int(32) number_of_entry;  

	for (inti=1; i <= number_of_entry; i++) {
		if (version==1) {
			unsigned int(64) time;
			unsigned int(64) moof_offset; 
		} else {
			unsigned int(32) time;
			unsigned int(32) moof_offset;
		}
		unsigned int((length_size_of_traf_num+1) * 8) traf_number;
		unsigned int((length_size_of_trun_num+1) * 8) trun_number;
		unsigned int((length_size_of_sample_num+1) * 8) sample_number;
	}
}
``` 

- track_ID :  标识 track_ID 的整数,从1开始 
- length_size_of_traf_num: 表示 traf_number 字段的字节长度减一。	
- length_size_of_trun_num: 表示 trun_number 字段的字节长度减一。	
- length_size_of_sample_num: 表示 sample_number 字段的字节长度减一。	
- number_of_entry:表示该轨道的条目数。	如果该值为0 则表示每个采样都是同步采样，后面没有表格条目。
- time（时间）:表示同步采样的呈现时间（单位：秒）。	
- moof_offset :表示此条目中使用的 "moof "的偏移量。偏移量是文件开头与 "moof "开头之间的字节偏移。	
- traf_number:  表示包含同步采样的 "traf "编号。从1开始 
- trun_number 表示包含同步采样的 "trun "编号。从1开始	
- sample_number 表示同步采样的“trun”编号。从1开始。

### 2.4.2. mfro (MovieFragmentRandomAccessOffsetBox )

表示外接 **mfra** 的字节数，该字段放在mfra字段的字段的最后，用于帮助从文件末尾扫描到**mfra**
``` cpp
aligned(8) class MovieFragmentRandomAccessOffsetBox  
	extends FullBox(‘mfro’, version, 0) {  
	unsigned int(32) size;  
}
```
# 3. 命令 

- 将mp4转换为fmp4 

``` shell
ffmpeg -re -i input.mp4 -c copy -f mp4 -movflags frag_keyframe+empty_moov output.mp4
```