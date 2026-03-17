# H264 Avc<no value>



![[Pasted image 20240806090850.png]]

# 1. H264包结构

## 1.1. NALU

NALU (Network Abstraction Layer Unit) 翻译过来就是网络抽象层单元.
在 H.264/AVC 视频编码标准中，最外面一层叫做 NAL(Network Abstract Layer) 网络抽象层。

### 1.1.1. AnnexB 封装
Annex-B格式 也叫MPEG-2 transport stream format格式（ts格式）, ElementaryStream格式。用于TS流中，以及使用TS作为切片的HLS格式中。


- 起始码startcode:  每个NALU单元以0001 或 001开头

- 防竞争字节（Emulation Prevention Bytes）：
	为了将原始码流可能出现的0001 和001与起始码作区分进行以下变换
```
	0 0 0 => 0 0 3 0
	0 0 1 => 0 0 3 1
	0 0 2 => 0 0 3 2
	0 0 3 => 0 0 3 3
```

 - EBSP 扩展字节序列载荷 (非H264标准定义，JM 项目中的名称)：
	  将带有 startcode 的 NALU 的去掉 0 0 0 1 的 startcode 的后的数据叫做 EBSP。
   
- RBSP 原始字节序列载荷 (H264标准名称):
	EBSP去掉防竞争字节后
 
- SODB 数据比特串:
	 对RBSP补齐部分去除的数据(编码时不足1字节的数据会补0至1字节)

#### 1.1.1.1. NALU header

| nal_unit( NumBytesInNALunit ) | C   | Descriptor |     |
| ----------------------------- | --- | ---------- | --- |
| forbidden_zero_bit            | All | f(1)       |     |
| nal_ref_idc                   | All | u(2)       |     |
| nal_unit_type                 | All | u(5)       |     |

##### 1.1.1.1.1. forbidden_zero_bit

禁止位，正常情况下为 0。在某些情况下，如果 NALU 发生丢失数据的情况，可以将这一位置为 1，以便接收方纠错或丢掉该单元。

##### 1.1.1.1.2. nal_ref_idc

该元素表示这个 NALU 的重要性。可能的值有 4 个，详情如下表。

| nal_ref_idc | 重要性        |
| ----------- | ---------- |
| 3           | HIGHEST    |
| 2           | HIGH       |
| 1           | LOW        |
| 0           | DISPOSABLE |

越重要的 NALU 越不能丢弃。

一般I帧为3，P帧B帧为2

##### 1.1.1.1.3. nal_unit_type

NALU 的类型。详情如下表。

| nal_unit_type | NALU 类型                  |                                          |
| ------------- | ------------------------ | ---------------------------------------- |
| 0             | 未定义                      |                                          |
| 1             | 非 IDR SLICE              | slice_layer_without_partitioning_rbsp( ) |
| 2             | 非 IDR SLICE，采用 A 类数据划分片段 | slice_data_partition_a_layer_rbsp( )     |
| 3             | 非 IDR SLICE，采用 B 类数据划分片段 | slice_data_partition_b_layer_rbsp( )     |
| 4             | 非 IDR SLICE，采用 C 类数据划分片段 | slice_data_partition_c_layer_rbsp( )     |
| 5             | IDR SLICE                | slice_layer_without_partitioning_rbsp( ) |
| 6             | 补充增强信息 SEI               | sei_rbsp( )                              |
| 7             | 序列参数集 SPS                | seq_parameter_set_rbsp( )                |
| 8             | 图像参数集 PPS                | pic_parameter_set_rbsp( )                |
| 9             | 分隔符 AUD                  | access_unit_delimiter_rbsp( )            |
| 10            | 序列结束符                    | end_of_seq_rbsp( )                       |
| 11            | 码流结束符                    | end_of_stream_rbsp( )                    |
| 12            | 填充数据                     | filler_data_rbsp( )                      |
| 13            | 序列参数扩展集                  | seq_parameter_set_extension_rbsp( )      |
| 14~18         | 保留                       |                                          |
| 19            | 未分割的辅助编码图像的编码条带          | slice_layer_without_partitioning_rbsp( ) |
| 20~23         | 保留                       |                                          |
| 24            | STAP-A未指定 单一时间的组合包       | RTP 封装H264                               |
| 25            | STAP-B未指定 单一时间的组合包       | RTP 封装H264                               |
| 26            | MTAP16   多个时间的组合包        | RTP 封装H264                               |
| 27            | MTAP24   多个时间的组合包        | RTP 封装H264                               |
| 28            | FU-A 分片的单元               | RTP 封装H264                               |
| 29            | FU-B 分片的单元               | RTP 封装H264                               |
|               |                          |                                          |

###### 1.1.1.1.3.1. 常见误区:
- IDR Slice 一定是一个 I Slice，而 I Slice 却不一定是一个 IDR Slice，那么即使这个 NALU 的 nalutype 等于 1，他也有可能是 I Slice。那么应该判断这个 Slice 是 I 还是 P 还是 B 呢？


### 1.1.2. 包类型
#### 1.1.2.1. SPS (sequence_parameter_set)序列参数集
保存了一组编码视频序列(Coded  video  sequence)的全局参数
一般情况SPS和PPS的NAL  Unit通常位于整个码流的起始位置。但在某些特殊情况下，在码流中间也可能出现这两种结构，主要原因可能为：
- 解码器需要在码流中间开始解码；
- 编码器在编码的过程中改变了码流的参数（如图像分辨率等）；

![[Pasted image 20231204162121.png]]
##### 1.1.2.1.1. _profile_idc_ 
表示的是当前这路码流的编码档次，我们有时候会说一路码流是 _Baseline_，_Main_ 等等，就是这个属性所决定的。具体的映射表如下：

- Baseline profile(基准档次) 66
- Main profile(主要档次) 77
- Extended profile(档次) 88
- High profile 100
- High 10 profile 110
- High 4:2:2 profile 122
- High 4:4:4 Predictive profile 244
- High 10 Intra profile 100 or 110
- High 4:2:2 Intra profile 100, 110, or 122
- High 4:4:4 Intra profile 44, 100, 110, 122, or 244
- CAVLC 4:4:4 Intra profile 44
##### 1.1.2.1.2. constraint_set0_flag ~ constraint_set5_flag
在编码的档次方面对码流增加的其他一些额外限制性条件。

##### 1.1.2.1.3. level_idc

标识当前码流的Level。编码的Level定义了某种条件下的最大视频分辨率、最大视频帧率等参数，码流所遵从的level由level_idc指定。

##### 1.1.2.1.4. seq_parameter_set_id

表示当前的序列参数集的id。通过该id值，图像参数集pps可以引用其代表的sps中的参数。

##### 1.1.2.1.5. log2_max_frame_num_minus4

用于计算MaxFrameNum的值。计算公式为MaxFrameNum = 2^(log2_max_frame_num_minus4 +  
4)。MaxFrameNum是frame_num的上限值，frame_num是图像序号的一种表示方法，在帧间编码中常用作一种参考帧标记的手段。
##### 1.1.2.1.6. pic_order_cnt_type

表示解码picture order count(POC)的方法。POC是另一种计量图像序号的方式，与frame_num有着不同的计算方法。该语法元素的取值为0、1或2。

##### 1.1.2.1.7. log2_max_pic_order_cnt_lsb_minus4

用于计算MaxPicOrderCntLsb的值，该值表示POC的上限。计算方法为MaxPicOrderCntLsb = 2^(log2_max_pic_order_cnt_lsb_minus4 + 4)。

##### 1.1.2.1.8. max_num_ref_frames

用于表示参考帧的最大数目。
##### 1.1.2.1.9. gaps_in_frame_num_value_allowed_flag

标识位，说明frame_num中是否允许不连续的值。
##### 1.1.2.1.10. pic_width_in_mbs_minus1

用于计算图像的宽度。单位为宏块个数，因此图像的实际宽度为:

frame_width = 16 × (pic\_width\_in\_mbs_minus1 + 1);
##### 1.1.2.1.11. pic_height_in_map_units_minus1

使用PicHeightInMapUnits来度量视频中一帧图像的高度。PicHeightInMapUnits并非图像明确的以像素或宏块为单位的高度，而需要考虑该宏块是帧编码或场编码。PicHeightInMapUnits的计算方式为：

PicHeightInMapUnits = pic\_height\_in\_map\_units\_minus1 + 1;

##### 1.1.2.1.12. frame_mbs_only_flag

标识位，说明宏块的编码方式。当该标识位为0时，宏块可能为帧编码或场编码；该标识位为1时，所有宏块都采用帧编码。根据该标识位取值不同，PicHeightInMapUnits的含义也不同，为0时表示一场数据按宏块计算的高度，为1时表示一帧数据按宏块计算的高度。

按照宏块计算的图像实际高度FrameHeightInMbs的计算方法为：

FrameHeightInMbs = ( 2 − frame_mbs_only_flag ) * PicHeightInMapUnits

##### 1.1.2.1.13. mb_adaptive_frame_field_flag

标识位，说明是否采用了宏块级的帧场自适应编码。当该标识位为0时，不存在帧编码和场编码之间的切换；当标识位为1时，宏块可能在帧编码和场编码模式之间进行选择。

##### 1.1.2.1.14. direct_8x8_inference_flag

标识位，用于B_Skip、B_Direct模式运动矢量的推导计算。

##### 1.1.2.1.15. frame_cropping_flag

标识位，说明是否需要对输出的图像帧进行裁剪。

##### 1.1.2.1.16. vui_parameters_present_flag

标识位，说明SPS中是否存在VUI信息。

##### 1.1.2.1.17. vui_parameter


##### 计算
    - 通过字段`pic_width_in_mbs_minus1`和`pic_height_in_map_units_minus1`计算视频宽高：  
        width=(pic_width_in_mbs_minus1+1)×16  
        height=(pic_height_in_map_units_minus1+1)×16×(2−frame_mbs_only_flag)  
        若无需裁剪（`frame_crop_*_offset=0`），公式可简化
    - 计算帧率：  
        fps=num_units_in_ticktime_scale​


#### 1.1.2.2. PPS (pics_parameter_set)图像参数集
![[Pasted image 20231204162140.png]]

##### 1.1.2.2.1. pic_parameter_set_id

表示当前PPS的id。某个PPS在码流中会被相应的slice引用，slice引用PPS的方式就是在Slice header中保存PPS的id值。该值的取值范围为[0,255]。

##### 1.1.2.2.2. seq_parameter_set_id

表示当前PPS所引用的激活的SPS的id。通过这种方式，PPS中也可以取到对应SPS中的参数。该值的取值范围为[0,31]。

##### 1.1.2.2.3. entropy_coding_mode_flag

熵编码模式标识，该标识位表示码流中熵编码/解码选择的算法。对于部分语法元素，在不同的编码配置下，选择的熵编码方式不同。例如在一个宏块语法元素中，宏块类型mb_type的语法元素描述符为“ue(v)  
| ae(v)”，在baseline profile等设置下采用指数哥伦布编码，在main profile等设置下采用CABAC编码。

标识位entropy_coding_mode_flag的作用就是控制这种算法选择。当该值为0时，选择左边的算法，通常为指数哥伦布编码或者CAVLC；当该值为1时，选择右边的算法，通常为CABAC。

##### 1.1.2.2.4. bottom_field_pic_order_in_frame_present_flag

标识位，用于表示另外条带头中的两个语法元素delta_pic_order_cnt_bottom和delta_pic_order_cn是否存在的标识。这两个语法元素表示了某一帧的底场的POC的计算方法。

##### 1.1.2.2.5. num_slice_groups_minus1

表示某一帧中slice group的个数。当该值为0时，一帧中所有的slice都属于一个slice group。slice group是一帧中宏块的组合方式，定义在协议文档的3.141部分。

##### 1.1.2.2.6. num_ref_idx_l0_default_active_minus1、num_ref_idx_l0_default_active_minus1

表示当Slice Header中的num_ref_idx_active_override_flag标识位为0时，P/SP/B  
slice的语法元素num_ref_idx_l0_active_minus1和num_ref_idx_l1_active_minus1的默认值。

##### 1.1.2.2.7. weighted_pred_flag

标识位，表示在P/SP slice中是否开启加权预测。

##### 1.1.2.2.8. weighted_bipred_idc

表示在B Slice中加权预测的方法，取值范围为[0,2]。0表示默认加权预测，1表示显式加权预测，2表示隐式加权预测。

##### 1.1.2.2.9. pic_init_qp_minus26和pic_init_qs_minus26

表示初始的量化参数。实际的量化参数由该参数、slice header中的slice_qp_delta/slice_qs_delta计算得到。

##### 1.1.2.2.10. chroma_qp_index_offset

用于计算色度分量的量化参数，取值范围为[-12,12]。

##### 1.1.2.2.11. deblocking_filter_control_present_flag

标识位，用于表示Slice header中是否存在用于去块滤波器控制的信息。当该标志位为1时，slice header中包含去块滤波相应的信息；当该标识位为0时，slice header中没有相应的信息。

##### 1.1.2.2.12. constrained_intra_pred_flag

若该标识为1，表示I宏块在进行帧内预测时只能使用来自I和SI类型宏块的信息；若该标识位0，表示I宏块可以使用来自Inter类型宏块的信息。

##### 1.1.2.2.13. redundant_pic_cnt_present_flag

标识位，用于表示Slice header中是否存在redundant_pic_cnt语法元素。当该标志位为1时，slice header中包含redundant_pic_cnt；当该标识位为0时，slice header中没有相应的信息。
#### 1.1.2.3. SEI
自定义信息，如果你想在码流中放入一些别的额外信息，就可以写到 SEI 信息里面。不过要注意的是，SEI 信息的重要性级别并不高，有些时候会在转码，转封装的过程中被忽略掉.

### 1.1.3. avcC 封装

AVCC格式也叫AVC1格式，MPEG-4格式，常用于mp4/flv等封装中。

这种格式通常被用于可以被随机访问的多媒体数据，如存储在硬盘的文件。MP4、MKV通常用AVCC格式来存储。

它的原理是在NALU 前面添加固定字节（可能是1字节、2字节或4字节，其中4字节较常见），这几个字节组成一个整数（大端字节序）表示整个 NALU 的长度，在读取的时候，先把这个整数读出来（例如ffmpeg从extradata获取），拿到这个 NALU 的长度，再按照长度读取整个 NALU。