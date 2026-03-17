# H265 Hevc<no value>

**高效视频编码**（HEVC），也称为H. 265和MPEG-H part 2

(H.264码流分Annex-B和AVCC两种格式。H.265码流是Annex-B和HVCC格式。)


# 1. NALU header

| nal_unit( NumBytesInNALunit ) | Descriptor |
| ----------------------------- | ---------- |
| forbidden_zero_bit            | f(1)       |
| nal_unit_type                 | u(6)       |
| nuh_layer_id                  | u(6)       |
| nuh_temporal_id_plus1         | u(3)       |

# 2. 包类型
相较H274的SPS、PPS， H265还多了VPS
## 2.1. VPS (video_parameter_set)视频参数集
