# 工具<no value>

音视频分析工具：


# 1. 未分类 
## 1.1. Elecard StreamEye

视频查看工具
查看帧的排列信息

## 1.2. mp4box

## 1.3. yamdi

## 1.4. VideoEye
![[Pasted image 20240327142115.png]]
# 2. MP4/fMP4 文件解析

# 3. Mp4 Explorer
	适用于解析Mp4文件
	 会展现box层级 
	 会显示解析出的字段含义
	 但是不会显示对应的二进制值 

![[Pasted image 20240327142242.png]]

## 3.1. mp4info

	适用于解析Mp4文件
	会展现box层级，并过滤出对应的二进制值(对于一些box尾部会少显示一部分值)
	只会展现文件的基本信息，不会按照box显示出对应的字段含义 

![[Pasted image 20240327142602.png]]


# 4. FLV 文件解析

## 4.1. FlvAnlyzer
	适用于解析FLV文件
	会展现完整的数据结构，并按照选中的内容过滤对应的二进制内容
	 会显示Header、Tag内部的字段含义

![[Pasted image 20240327163452.png]]

## 4.2. FlvParse
	适用于解析FLV文件
	会展现完整的数据结构，并按照选中的内容过滤对应的二进制内容
	 可以过滤选择是否显示Header、Tag内部的字段细节含义
	 但是老是容易崩溃

![[Pasted image 20240327163904.png]]


# 5. Http-FLV 播放器
https://github.com/bilibili/flv.js

# 6. Websocket-fmp4 播放器

https://github.com/v354412101/wsPlayer


7.![[Snipaste_2026-03-14_23-07-55.jpg]]