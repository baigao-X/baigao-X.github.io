# Hls（M3U8） 协议解读<no value>

**HTTP Live Streaming**
由苹果公司推出的基于HTTP的能自适应的流媒体传输协议， 支持直播和点播，在Android、iOS常用。
本质是服务端将视频流切成一个个小的媒体文件（一般是ts文件或者fmp4文件）。然后将这些媒体文件的播放地址按照播放顺序罗列到一个索引文件中。然后使用http下载的方式下载。
- 标准文档： https://datatracker.ietf.org/doc/html/draft-pantos-http-live-streaming-23
与MPEG-DASH国际标准大致相同，只是细节存在差异 

# 1. m3u8文件

即索引文件。

## 1.1. 格式说明

- 必须以 UTF-8[RFC-3629]编码，不能包含BOM(Byte Order Mark)字节序。不能包含 UTF-8 控制字符（U+0000 到 U+001F 和 U+007F 至 U+009F)，CR (U+000D) 和 LF 除外(U+000A)。
- 每行内容只能是以下三种之一
	- URI
	- 空行。除了明确指定的元素之外不能有空格的存在。
	- 以#开头的字符串，表示注释或者标签。标签以#EXT开头，大小写敏感。

### 1.1.1. 属性列表
某些特定的标签的值是一个属性列表。


## 1.2. media playlist

形如
```
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-PLAYLIST-TYPE:VOD
#EXT-X-TARGETDURATION:11
#EXTINF:10.000,
url_462/193039199_mp4_h264_aac_hd_7.ts
#EXTINF:10.000,
url_463/193039199_mp4_h264_aac_hd_7.ts
#EXTINF:10.000,
url_464/193039199_mp4_h264_aac_hd_7.ts
#EXTINF:10.000,
url_465/193039199_mp4_h264_aac_hd_7.ts
#EXTINF:10.000,
url_466/193039199_mp4_h264_aac_hd_7.ts
#EXTINF:10.000,
url_467/193039199_mp4_h264_aac_hd_7.ts
#EXTINF:10.000,
url_468/193039199_mp4_h264_aac_hd_7.ts
#EXTINF:9.950,
url_469/193039199_mp4_h264_aac_hd_7.ts
#EXTINF:10.050,
url_470/193039199_mp4_h264_aac_hd_7.ts
#EXTINF:10.000,
url_471/193039199_mp4_h264_aac_hd_7.ts
#EXTINF:10.000,
url_472/193039199_mp4_h264_aac_hd_7.ts
#EXT-X-ENDLIST

```


## 1.3. master playlist

形如
```
#EXTM3U
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=2149280,CODECS="mp4a.40.2,avc1.64001f",RESOLUTION=1280x720,NAME="720"
url_0/193039199_mp4_h264_aac_hd_7.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=246440,CODECS="mp4a.40.5,avc1.42000d",RESOLUTION=320x184,NAME="240"
url_2/193039199_mp4_h264_aac_ld_7.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=460560,CODECS="mp4a.40.5,avc1.420016",RESOLUTION=512x288,NAME="380"
url_4/193039199_mp4_h264_aac_7.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=836280,CODECS="mp4a.40.2,avc1.64001f",RESOLUTION=848x480,NAME="480"
url_6/193039199_mp4_h264_aac_hq_7.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=6221600,CODECS="mp4a.40.2,avc1.640028",RESOLUTION=1920x1080,NAME="1080"
url_8/193039199_mp4_h264_aac_fhd_7.m3u8

```
## 1.4. 码率自适应

为了适应观众的不同网络环境，服务端会以不同的比特率和分辨率创建多个流，并把不同流的媒体文件地址放到不同的索引文件(media playlist)中。 并最终把不同流的索引文件的地址放到一个主索引文件（master playlist）中

![[Pasted image 20231219210855.png]]![[Pasted image 20231219211031.png]]