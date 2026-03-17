# Ps Ts流协议解读<no value>

# 1. MPEG2-PS

	Program Stream(节目流) 
	一种多路复用数字音视频等的封装容器。PS是Program Stream(程序流或节目流)的简称。PS流由PS包组成，而一个PS包又由若干个PES包组成，它是为可靠稳定的储存媒介如光盘而设计的。
	PS包的包头中包含了同步信息与时钟恢复信息。一个PS包最多可包含具有同一时钟基准的16个视频PES包和32个音频PES包。



## 1.1. PS流结构


![[Pasted image 20231213173912.png]]


### 1.1.1. PSH：Program Stream pack Header 
	是PS包的包头
	主要包含时间戳、最大比特率、帧号信息
	若当前码流类型为音频流，则可选择是否包含PSH头部
	若当前码流类型为视频流，且当前帧的第一个NALU则包含PSH头部。
		若为I帧，PSH头部长度为44字节
			**PS system header**：Partial system header，系统头；
		若为P/B帧，PSH头部长度为20字节

| 字节号   | 字段名        | 含义                                   |
| ----- | ---------- | ------------------------------------ |
| 0-3   | sync bytes | 0x000001BA, PSH头部起始码                 |
| 4-8   |            | 含有当前帧时间戳                             |
| 10-12 |            | 当前设置的最大比特率                           |
| 16-19 |            | 当set_frame_end_flg置为1，则16-19中存放当前帧帧号 |
|       |            | I帧附加信息(PS system header)             |
| 20-23 |            | 0x000001BB,I帧附加信息起始码                 |
| 24-25 |            | 为18，等于I帧附件信息长度-2                     |
| 26-28 |            | 表示当前设置的最大比特率                         |

![[Pasted image 20231214094334.png]]
![[Pasted image 20231214094742.png]]

### 1.1.2. PSM：Program Stream Map，节目流映射
提供节目流中基本流的描述及其相互关系。当在传输流中承载时，此结构将不修正。当 stream_id 值为 0xBC 时， PSM 作为 PES 包存在。

| 节号          | 字段名        | 含义                                      |
| ----------- | ---------- | --------------------------------------- |
| 0-3         | sync bytes | 0x000001BC, PSM头部起始码                    |
| 8-9         |            | BASIC信息长度+DEVICE信息长度+加密信息长度             |
|             |            | BASIC信息                                 |
| AA          |            | 0x40, 表示当前BASIC信息                       |
| AA+1        |            | 14,表示BASIC信息长度-2                        |
| AA+2-AA+3   |            | 公司描述符                                   |
| AA+6-AA+11  |            | 当前时间的年月日时分秒以及加密类型                       |
| AA+12       |            | 相机类型                                    |
|             |            | DEVICE信息                                |
| BB          |            | 0x41, 表示当前DEVICE信息                      |
| BB+1        |            | 18,表示DEVICE信息长度-2                       |
| BB+4-BB+19  |            | 设备ID号                                   |
|             |            | 加密信息                                    |
| CC          |            | 0x80, 表示当前加密信息                          |
| CC+1        |            | 6,表示加密信息长度-2                            |
| CC+4        |            | 打包方式，加密算法                               |
| CC+5        |            | 加密轮数，加密长度                               |
| CC+6        |            | 加密类型                                    |
| DD          |            | 视频流信息长度+音频流信息长度+私有数据信息长度                |
|             |            | 视频流信息                                   |
| EE          |            | 视频编码类型，如H264、H265                       |
| EE+1        |            | 码流类型，此处为0xE0，表示视频码流                     |
| EE+2-EE+3   |            | VIDEO信息长度+VIDEO_CLIP信息长度+TIMING_HRD信息长度 |
|             |            | VIDEO信息                                 |
| aa          |            | 0x42,表示当前为VIDEO信息                       |
| aa+1        |            | 14,VIDEO信息长度-2                          |
| aa+2-aa+3   |            | 编码器版本                                   |
| aa+4-aa+5   |            | 编码年月日                                   |
| aa+6-aa+9   |            | 原始图片宽高                                  |
| aa+10       |            | 是否隔行扫描，B帧数目，是否为SVC码流，是否使用e帧，最大参考帧数目     |
| aa+11       |            | 水印类型，显示时是否需要反隔行                         |
| aa+12       |            | JPEG的Q值                                 |
| aa+13-aa+15 |            | 以1/90000s为单位的两帧间时间间隔，是否使用固定频率           |
|             |            | VIDEO_CLIP信息                            |
| bb          |            | 0x44,表示当前为VIDEO_CLIP信息                  |
| bb+1        |            | 10,表示VIDEO_CLIP信息长度-2                   |
| bb+2-bb+3   |            | 裁剪起始x座标                                 |
| bb+4-bb+5   |            | 裁剪起始y座标                                 |
| bb+6-bb+7   |            | 裁剪宽度                                    |
| bb+8-bb+9   |            | 裁剪高度                                    |
|             |            | TIMING_HRD信息                            |
| cc          |            | 0x44,表示当前为TIMING_HRD信息                  |
| cc+1        |            | 10,表示TIMING_HRD信息长度-2                   |
| cc+4-cc+7   |            | 以1/45000s为单位的两帧间时间间隔                    |
| cc+10       |            | 图片宽度                                    |
| cc+11       |            | 图片高度                                    |
|             |            | 音频流信息                                   |
| FF          |            | 音频编码类型，如AAC                             |
| FF+1        |            | 码流类型，此处为0xc，表示音频码流类型                    |
| FF+2-FF+3   |            | AUDIO信息长度                               |
|             |            | AUDIO信息                                 |
| dd          |            | 0x43,表示当前为AUDIO信息                       |
| dd+1        |            | 14,AUDIO信息长度-2                          |
| dd+2-dd+3   |            | 音频帧长度                                   |
| dd+4        |            | 音频声道数                                   |
| dd+5-dd+7   |            | 音频采样率                                   |
| dd+8-dd+10  |            | 音频比特率                                   |
|             |            | 私有数据类型                                  |
| GG          |            | 私有数据类型                                  |
| GG+1        |            | 码流类型，此处为0xbd，表示私有数据类型                   |
| GG+2-GG+3   |            | 0                                       |
| HH          |            | CRC校验                                   |


![[Pasted image 20231214094800.png]]

- packet_start_code_prefix: 24bit, 0x000001 

# 2. MPEG2-TS

**Transport Stream（传输流）**, 由定长的TS包组成（188字节），而TS包是对PES包的一个重新封装

TS流与PS流的区别在于TS流的包结构是固定长度的,而PS流的包结构是可变长度的。PS包由于长度是变化的,一旦丢失某一PS包的同步信息,接收机就会进入失步状态,从而导致严重的信息丢失事件。而TS码流由于采用了固定长度的包结构,当传输误码破坏了某一TS包的同步信息时,接收机可在固定的位置检测它后面包中的同步信息,从而恢复同步,避免了信息丢失。
因此在信道环境较为恶劣、传输误码较高时一般采用TS码流,而在信环境较好、传输误码较低时一般采用PS码流。


## 2.1. TS 流结构
	固定188字节一个包
	
```
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
 | TS Header (4 bytes) |  Payload （188 - 4 bytes， PES）         |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+  
 ```

### 2.1.1. TS Header
	4字节
	
``` cpp
transport_packet() {
	//header
	const bit(8) sunc_byte = 0x47;              //同步码，固定为0x47
	bit(1) transport_error_indicator;           //传输错误标志
	bit(1) payload_unit_start_indicator;        //负载起始标志，1表示是一个完整包的开始，0表示不是一个完整的包
	bit(1) transport_priority;                  //传输优先级，1表示优先级更高 
	bit(13) PID;                                //指示存储与分组有效载荷中数据的类型，通过PMT包表明PID对应的音频、 视频、数据
	bit(2) transport_scrambling_control;        //加扰控制标志，表示加密模式。一般卫星付费节目会用这个，互联网场景很少用。
	bit(2) adaption_field_control;              //适配域控制标志，表示包头是否有调整字段或有效负载。
	bit(4) continuity_counter;                  //连续性计数期，表示同个PID的TS流分组计数。0~15循环

	//header end

	if (adaption_field_control == '10' || adaption_field_control == '11') {
		adaptation_field()
	}
	if (adaption_field_control == '01' || adaption_field_control == '11') {
		for (int i=0; i < N; i++) { // N为 188减去其他所有字段后 
			data_byte
		}
	}
}
```

- adaption_field_control:
	- 00: 供未来使用，由 ISO/IEC 所保留
	- 01: 无 adaptation_field，仅有效载荷
	- 10: 仅有 Adaptation_field，无有效载荷。 空包也用10. 
	- 11: Adaptation_field 后随有效载荷。Adaptation_field中的第一个字节表示Adaptation_field的长度
	
-  PID 表

| Value         | 描述                                         |
| ------------- | ------------------------------------------ |
| 0x0000        | PAT(Program Association Table,节目相关表)       |
| 0x0001        | CAT(Conditional Access Table,条件接受表)        |
| 0x0002        | TSDT(Transport Stream Description Table)   |
| 0x0003        | IPMP (Control Information Table)           |
| 0x0004-0x000F | 保留                                         |
| 0x0011        | SDT (Service Description Table, 服务描述表)     |
| 0x0012        | EIT,ST                                     |
| 0x0013        | RST,ST                                     |
| 0x0014        | TDT,TOT,ST                                 |
| 0x0010-0x1FFE | 自定义 PID，可用于 PMT 的 pid、network 的 pid 或者其他目标 |
| 0x1FFF        | 空包                                         |
PCR 的 PID 可以选择 0、1 或者 0x10~0x1FFE 中的任意值

### 2.1.2. adaptation field 调整字段
      
调整字段，或者被翻译为自适应区间  TS header 中 adaptation_filed_control == 1x 时，表示该字段存在。通常以下两种场景时会存在该字段:
1.封装 TS 数据的时候，视频或者音频数据不够 184 个字节的时候，使用该段来指明调整字段 0xFF 的长度，此时的自适应区的 PCR 标志为 0。打包 ts 流时 PAT 和 PMT 表是没有 adaptation field 的，不够的长度直接补 0xff 即可。
2.对于每一帧视频数据进行封装的时候，通常在第一个TS包和最后一个TS包会加上自适应区。用来承载**PCR 相关数据**。此时自适应区的 PCR 标志为 1. 

#### 2.1.2.1. 数据结构

![[Pasted image 20240108135855.png]]

- adaption_field_length（8bits）：指定了其后的 adaptation field 的字节数。

- discontinuity_indicator（1bit）：
    - 为 1 指示当前的 TS Packet 的不连续状态为 true；
    - 为 0，则当前的 TS packet 的不连续状态为 false。
    - 这个 discontinuity_indicator 用于指示两种类型的不连续性：系统时基的不连续性和 continuity_counter（连续计数器）的不连续性。

- random_access_indicator（1bit）：
    - random_access_indicator 是 1bit 的字段，它指示当前的 TS packet 和后续可能具有相同 PID 的 TS packet 包含一些支持此时可以随意访问的信息。
    - 当该字段被设置为 1 时，如果 PES 流类型为 1 或 2，则具有相同 PID 的 TS packet 的下一个 PES packet 的负载开始将会包含 video sequence header 的第一个字节，或者如果 PES 流类型为 3 或 4，则包含 audio frame 的第一个字节。
    - 在视频的情况下， 包含 video sequence header 后的第一个图像的 PES packet 中将会有一个 presentation timestamp。
    - 在音频的情况下，包含音频帧的第一个字节的 PES packet 中将会有一个 presentation timestamp。
    - 在 PCR_PID 中，random_access_indicator 只可以在包含 PCR 字段的 TS packet 中被设置为 1.

- elementary_stream_priority_indicator（1bit）：
    - 该字段指示在具有相同 PID 的 packet 中，在该 TS packet 的负载内携带的 ES 数据（即音视频数据）的优先级。为 1 指示该负载比其他的 TS packet 的负载具有更高的优先级。
    - 在 video 的情况下，如果 payload 包含一个或多个来自 intra-coded slice 的字节，则该字段可能会被设为 1。
    - 值为 0 表示该 payload 与其他没有设置该字段为 1 的 pakcet 具有相同的优先级。
    
- PCR_flag（1bit）：为 1 指示 adaptation field 包含两部分编码的 PCR 字段。为 0 指示 adaptation field 没有包含任何的 PCR 字段。

- OPCR_flag（1bit）：为 1 指示 adaptation field 包含两部分编码的 OPCR 字段；为 0 指示 adaptation field 没有包含任何 OPCR 字段。

- splicing_point_flag（1bit）：为 1 指示 splice_countdown 将会在相关 adaptation field 中指定的拼接点出现；为 0 指示 splice_countdown 在 adaptation field 中不存在。

- transport_private_data_flag（1bit）：为 1 指示 adaptation field 包含一个或多个 private_data 字节；为 0 指示 adaptation field 中没有包含任何的 private_data 字节。

- adaptation_field_extension_flag（1bit）：为 1 表示有 adaptation field 扩展出现；为 0 表示 adaptation field 中不存在 adaptation field 扩展。

- 当 PCR_Flag == 1 时，有如下字段：
	- PCR 主要用来实现解码端的时钟同步。
    - program_clock_reference_base（33bits）：
    - const1_value0（6bits）：保留字段，固定为 1.
    - program_clock_reference_extension（9bits）：
    - 节目时钟参考（PCR）是一个在两部分编码的 42bit 字段（program_clock_reference_base 和 program_clock_reference_extension）。
        - 第一部分 program_clock_reference_base 是一个 33bit 字段，其值由 PCR_base 指定。
        - 第二部分 program_clock_reference_extension 是一个 9bit 字段，其值由 PCR_ext 指定。
        - PCR 指示包含 program_clock_reference_base 最后一个 bit 的字节在系统目标解码器的输入端的预期到达时间。
    
- 当 OPCR_flag == 1 时，有如下字段：
    - original_program_clock_reference_base（33bits）：
    - const1_value2（6bits）：保留字段，固定为 1.
    - original_program_clock_reference_extension（9bits）：
- 当 splicing_point_flag == 1 时，有如下字段：
    - splice_countdown（8bits）：
- 当 transport_private_data_flag == 1 时，有如下字段：
    - transport_private_data_length（8bits）：
    - transport_private_data：[transport_private_data_length]bytes
- 当 adaptation_field_extension_flag == 1 时，有如下字段：
    - adaptation_field_extension_length（8bits）：
    - ltw_flag（1bit）：为 1 表示有 ltw_offset 字段存在。
    - piecewise_rate_flag（1bit）：为 1 表示有 piecewise_rate 字段存在
    - seamless_splice_flag（1bit）：
    - const1_value1（5bits）：保留字段，固定为 1.
    - 当 ltw_flag == 1 时，有如下字段：
        - ltw_valid_flag（1bit）：
        - ltw_offset（15bits）：
    - 当 piecewise_rate_flag == 1 时，有如下字段：
        - reserved（2bits）：保留字段
        - piecewise_rate（22bits）：
    - 当 seamless_splice_flag == 1 时，有如下字段：
        - splice_type（4bits）：
        - DTS_next_AU0（3bits）：
        - marker_bit0（1bit）：
        - DTS_next_AU1（15bits）：
        - marker_bit1（1bit）：
        - DTS_next_AU2（15bits）：
        - marker_bit2（1bit）：


### 2.1.3. 有效载荷数据



#### 2.1.3.1. PSI（节目特定信息）

在 MPEG-2 中定义了节目特定信息（PSI），PSI 用来描述传送流的组成结构
在多路复用中尤为重要的是 **PAT** 和 **PMT** 

##### 2.1.3.1.1. pointer_filed

``` cpp

psi_packet() 
	extend transport_packet(PID=0x00/*PAT*/,0x11/*SDT*/,PMT) {
	if (PID /*in transport_packet*/ == PSI // PAT PMT
		 && payload_unit_start_indicator /*in transport_packet*/ == 1) { 

		unsigned int(8) pointer_filed;
		bit(8)[pointer_filed] other;
		data_byte
	}
}

```

当负载是PSI时,
	当 TS 头部中的 payload_unit_start_indicator 为 1 则存在该字段
		表示该字段之后到有效负载第一个字节（不含）之间的字节数。

##### 2.1.3.1.2. SDT (Service Description Table 服务描述表)
用于描述系统提供的MPEG 传输流中包含的电视、广播或其他服务。
由数字视频广播 (DVB)提出，见《数字视频广播 (DVB)； DVB 系统中的服务信息 (SI) 规范》

``` cpp

program_association_section() { 
	bit(8) table_id;         //可以是0x42(66),表示描述的是当前流的信息,也可以是0x46(70),表示是其他流的信息(EPG使用此参数)
	const bit(1) section_syntax_indicator = 1;
	const bit(1) reserved_future_use = 0;
	bit(2) reserved;
	bit(12) sestion_length;    //头两bit必定为'0',后10比特指定改分段的字节数(包括到CRC)
	bit(16) transport_stream_id; //流id， 用于在一个网络中从其他的多路复用中识别此传送流，其值用户自定义
	bit(2) reserved;
	bit(5) version_number;             //整个节目相关表的版本号。当current_next_indicator 为1时，表示当前有效的节目相关表的版本号；当为0时，表示下一个有效的节目相关表的版本号。
	bit(1) current_next_indicator;     //1时表示当前发送的节目相关表有效。0时表示当前表无效，下一个表有效
	bit(8) section_number;             //此分段的编号。从0x00开始累计
	bit(8) last_section_number;        //表示完整节目相关表的最后分段编号。
	bit(8) reserved_future_use;

	//节目循环
	for (int i=0; i< N;i++) {
		unsigned int(32) service_id    //服务器ID,实际上就是PMT段中的program_number.
		const bit(1) reserved_future_use; 
		bit(1) EIT_schedule_flag;      //EIT信息,1表示当前流实现了该节目的EIT传送
		bit(1) EIT_present_following_flag; //EIT信息,1表示当前流实现了该节目的EIT传送
		bit(3) running_status; //运行状态信息:1-还未播放 2-几分钟后马上开始,3-被暂停播出,4-正在播放,其他---保留
		bit(1) free_CA_mode;    //加密信息,''1''表示该节目被加密. 紧接着的是描述符,一般是Service descriptor,分析此描述符可以获取servive_id指定的节目的节目名称.具体格式请参考 EN300468中的Service descriptor部分.
		bit(12) descriptors_loop_length;  //表示一下描述符的总长度
		for (int j=0; j<N;j++) {
			descriptor()   //分析这个，节目名称和节目号码已经联系起来了.机顶盒程序就可以用这些节目名称代替 PID让用户选择,从而实现比较友好的用户界面!
		}
	}

	unsigned int(32) CRC_32;
}

```

##### 2.1.3.1.3. PAT (Program Association Table 节目相关表/节目关联表)

	Program Association Table 节目关联表.
	PID值为0
	PAT 表,每个 TS 流对应一张,给出了一路 MPEG-2 码流中有多少套节目，以及它与 PMT 表 PID 之间的对应关系；
	同时还提供网络信息表（NIT）的位置，即 NIT 的 TS 包的包标识符（PID）的值。
	TS 流中中，PAT 包重复实现，大约 0.5 秒出现一个，保证实时解码性
	
``` cpp

program_association_section() { 
	bit(8) table_id;         //根据值，确认是不同的节目表
	const bit(1) section_syntax_indicator = 1;
	const bit(1) zero = 0;
	bit(2) reserved;
	bit(12) sestion_length;    //头两bit必定为'0',后10比特指定改分段的字节数(包括到CRC)
	bit(16) transport_stream_id; //流id， 用于在一个网络中从其他的多路复用中识别此传送流，其值用户自定义
	bit(2) reserved;
	bit(5) version_number;             //整个节目相关表的版本号。当current_next_indicator 为1时，表示当前有效的节目相关表的版本号；当为0时，表示下一个有效的节目相关表的版本号。
	bit(1) current_next_indicator;     //1时表示当前发送的节目相关表有效。0时表示当前表无效，下一个表有效
	bit(8) section_number;             //此分段的编号。从0x00开始累计
	bit(8) last_section_number;        //表示完整节目相关表的最后分段编号。

	//节目循环
	for (int i=0; i< N;i++) {
		unsigned int(32) program_number    //当其为 0 时，接下来的 PID 为网络 PID。当其为 1 时，表示这是 PMT，因此接下来的 PID 为 PMT
		const bit(3) reserved = 0b111;
		if (program_number == '0') {
			bit(13) network_PID
		} else {
			bit(13) program_map_PID       //节目号 PMT 对应内容的 PID 值
		}
	}

	unsigned int(32) CRC_32;

}

```

- table_id

|           |                                                                |
| --------- | -------------------------------------------------------------- |
| 值         | 描 述                                                            |
| 0x00      | program_association_section                                    |
| 0x01      | conditional_access_section (CA_section)                        |
| 0x02      | TS_program_map_section                                         |
| 0x03      | TS_description_section                                         |
| 0x04      | ISO_IEC_14496_scene_description_section                        |
| 0x05      | ISO_IEC_14496_object_descriptor_section                        |
| 0x06      | Metadata_section                                               |
| 0x07      | IPMP_Control_Information_section (defined in ISO/IEC 13818-11) |
| 0x08-0x3F | ITU-T H.222.0 建议书 \| ISO/IEC 13818-1 保留                        |
| 0x40-0xFE | 用户专用                                                           |
| 0xFF      | 禁用                                                             |
|           |                                                                |
##### 2.1.3.1.4. PMT (Program Association Table 节目映射表）

	Program Map Table，节目映射表，该表的 PID 是由 PAT 表 提供给出的。表征一路节目所有流信息。
	要的作用就是指明了音视频流的 PID 值。
	包含：
	当前节目中包含的所有 Video 数据的 PID
	当前节目中包含的所有 Audio 数据的 PID
	与当前节目关联在一起的其他数据的 PID（如数字广播，数据通讯等使用的 PID）

``` cpp
TS_program_map_section() {
	//8bytes 
	const bit(8) table_id = 0x02;         //固定位0x02,表示为PMT 
	const bit(1) section_syntax_indicator = 1;
	const bit(1) zero = 0;
	const bit(2) reserved = 0b11;
	bit(12) sestion_length;    //头两bit必定为'0',后10比特指定改分段的字节数(包括到CRC)
	bit(16) program_number; //流id， 用于在一个网络中从其他的多路复用中识别此传送流，其值用户自定义
	const bit(2) reserved = 0b11;
	bit(5) version_number;             //整个节目相关表的版本号。当current_next_indicator 为1时，表示当前有效的节目相关表的版本号；当为0时，表示下一个有效的节目相关表的版本号。
	bit(1) current_next_indicator;     //1时表示当前发送的节目相关表有效。0时表示当前表无效，下一个表有效
	bit(8) section_number;             //此分段的编号。从0x00开始累计。 PMT中固定为0 
	bit(8) last_section_number;        //表示完整节目相关表的最后分段编号。PMT中固定为0 
	const bit(3) reserved = 0b111;
	bit(13) PCR_PID;                   //表示TS包的PID值，该TS包含PCR域
	const bit(3) reserved = 0b1111;
	bit(12) program_info_length;      //前两位bit为-01，该域指出跟随其后对节目信息描述的byte数
	//8bytes end


	for (int i=0; i< N;i++) {
		descriptor()
	}
	//流循环 
	for (int i=0; i< N1; i++) {
		unsigned int(8) stream_type;  //指定特定PID节目元素包类型.  指定编码类型,同时可区分音视频  
		const bit(3) reserved = 0x111;
		bit(13) elementary_PID; //该节目TS包的PID值。
		bit(4) reserved;
		bit(12) ES_info_length; //前两位bit为00，指示跟随其后的描述相关节目元素byte数
		for(int j=0; j<N2; j++) {
			descriptior();
		}
	}

	unsigned int(32) CRC_32;  // 整个PAT的校验码
}
```

- stream_type 枚举表
	- 0x1b : H264 
	- 0x24 : H265 
	- 0x0F: AAC

##### 2.1.3.1.5. CAT (条件接受表)
	将一个或多个专用EMM流分别与唯一的PID相关联，描述TS流的加密方式
##### 2.1.3.1.6. NIT (网络信息表）
	描述整个网络，如多少个TS流，频点和调试方式。

#### 2.1.3.2. PES
	即音视频有效数据的载体


# 3. PES 层（Packet Elemental Stream）

    
Packetized Elementary Streams (分组的ES)，ES形成的分组称为PES分组，是用来传递ES的一种数据结构。PES流是ES流经过PES打包器处理后形成的数据流，在这个过程中完成了将ES流分组、打包、加入包头信息等操作（对ES流的第一次打包）。PES流的基本单位是PES包。PES包由包头和payload组成。

PTS/DTS是打在PES包的包头里面的

## 3.1. 包结构
### 3.1.1. PES Herader
	视频流/音频流/私有数据流都包含若干PES包。每个PES包由PES头部和码流数据两部分组成
	 对于视频流，每帧视频流分为若干NALU，每个NALU分为若干个段，每个段需加一个PES头部。第一个NALU的第一段的PES头部中可包含pts信息和user_data信息。

- 固定头 （6 bytes）
	- Sync Bytes 3bytes:  起始码，固定为0x000001
	- stream_id (8bits), PES包负载的流类型。 0xE0: 视频流；0xC0：音频流。 0xBD: 私有数据
	- PES_packet_length(16bits): PES包长度。包括此字段之后的可选包头和有效负载长度。

- 可选包头
	- const2bits（2bits）：保留字段，固定为 '10'。
	- PES_scrambling_control（2bits）: 加密模式，00 未加密；01 或 10 或 11 有用户定义.
	- PES_priority（1bit）：有效负载的优先级，该字段为 1 的 PES packet 的优先级比为 0 的 PES packet 高.
	- data_alignment_indicator（1bit）：数据定位指示器.
	- copyright（1bit）：版本信息，1 为有版权，0 无版权.
	- original_or_copy（1bit）：原始或备份，1 为 原始，0 为备份.
	- PTS_DTS_flags（2bits）：10 表示 PES 头部有 PTS 字段，11 表示有 PTS 和 DTS 字段，00 表示都没有，01 被禁止.
	- ESCR_flag（1bit）：1 表示 PES 头部有 ESCR 字段，0 则无该 ESCR 字段.
	- ES_rate_flag（1bit）：1 表示 PES 头部有 ES_rate 字段，0 表示没有.
	- DSM_trick_mode_flag（1bit）：1 表示有一个 8bit 的 track mode 字段，0 表示没有.
	- additional_copy_info_flag（1bit）：1 表示有 additional_copy_info 字段，0 表示没有.
	- PES_CRC_flag（1bit）：1 表示 PES 包中有 CRC 字段，0 表示没有.
	- PES_extension_flag（1bit）：1 表示 PES 头部中有 extension 字段存在，0 表示没有.
	- PES_header_data_length（1bit）：指定在 PES 包头部中可选头部字段和任意的填充字节所占的总字节数。可选字段的内容由上面的 7 个 flag 来控制的.
	- - 当 PTS_DTS_flags == '10' 时，有 PTS
	- 当 PTS_DTS_flags == '11' 时，有 PTS 和 DTS
	    - PTS(presentation time)显示时间戳和DTS(decoding time stamp)解码时间戳，是用来音视频同步的，DTS/PTS是相对SCR（系统参考）的时间戳，是以 90000 为单位，PTS/DTS 到 ms 的转换公式是 PTS/90，系统时钟频率为 90Khz，所以转换到秒为 PTS/90000。 ![[Pasted image 20240108225148.png]]
	- - 当 ESCR_flag == '1' 时，有 ESCR
	    - ESCR 字段占位 48bit，由 33bit 的 ESCR_base 字段和 9bit 的 ESCR_extension 字段组成.
	    - ESCR_extension（9bits）：周期数，取值范围 0~299，循环一次，base + 1
	- 当 ES_rate_flag == '1' 时，有 ES_rate
	    - 目标解码器接收 PES 分组字节速率，禁止为 0，占 24bits.
	- 当 DSM_trick_mode_flag == '1' 时，有 track mode control
	    - trick_mode_control：表示哪种 trick mode 被应用于相应的视频流，占 8bits；其中 trick_mode_control 占前 3 个 bit，根据其值后面有 5 个bit的不同内容，具体可看上图。
	- 当 additional_copy_info_flag == '1' 时，有 additional_copy_info
	    - additional_copy_info（7bits）：表示和版权相关的私有数据.
	- 当 PES_CRC_flag == '1' 时，有 previous_PES_packet_CRC
	    - previous_PES_packet_CRC（16bits）：CRC 校验值
	- 当 PES_extension_flag == '1' 时，有 PES 扩展字段
	    - PES_private_data_flag（1bit）：1 表示有 PES_private_daa 存在，0 则无
	        - PES_private_data（128bits）：私有数据
	    - pack_header_field_flag（1bit）：1 表示有 pack_header_field
	        - pack_field_length（8bits）：指示 pack_header() 的大小
	        - pack_header()
	    - program_packet_sequence_counter_flag（1bit）：1 表示有 program_packet_sequence_counter
	        - program_packet_sequence_counter（7bits）
	        - MPEG1_MPEG2_identifier（1bit）：1 指示这个 PES packet 携带的信息来自 ISO/IEC 11172-1 流；0 指示这个 PES packet 携带的信息来自节目流.
	        - original_stuff_length（6bits）: 指定在原始的`ITU-T Rec. H.222.0 | ISO/IEC 13818-1`PES packet 头或者原始的`ISO/IEC 11172-1`packet 头中使用的填充字节大小.
	    - P-STD_buffer_flag（1bit）：1 表示有 P-STD_buffer
	        - PES_extension_field_length（7bits）
	        - stream_id_extension_flag（1bit）
	    - Reserved（3bits）：保留字段
	    - PES_extension_flag_2（1bit）: 1 表示有 PES_extension_field_length 和 associated 字段


# 4. ES 层：Elementary Stream
    
基本码流，是由编码器输出的原始基础码流，它只含有解码器所必需的、并与原始图象或原始音频相接近的信息。由由压缩器输出的用于传送 单路视音频信号的原始码流。ES只包含一种内容的数据流，如只含视频或只含音频等。

打包 es 层数据时 pes 头和 es 数据之间要加入一个 AUD(type=9) 的 nalu
关键帧 slice 前必须要加入 SPS(type=7) 和 PPS(type=8) 的 nalu，而且是紧邻的。如下图所示：
![[Pasted image 20240108230324.png]]