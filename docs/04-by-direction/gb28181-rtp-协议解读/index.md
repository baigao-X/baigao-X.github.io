# Gb28181 Rtp 协议解读<no value>

1)、视频关键帧的封装 `RTP + PS header + PS system header + PS system Map + PES header +h264 data`

2)、视频非关键帧的封装 `RTP +PS header + PES header + h264 data`

3)、音频帧的封装: `RTP + PES header + G711`


# 1. 结构

