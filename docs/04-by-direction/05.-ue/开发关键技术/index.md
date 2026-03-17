# 开发关键技术<no value>

## key

### UR开发

#### 如何连接串流

#### PlayMode模式进行操纵手柄控制

### 视频流控制
####  InVideo 插件支持拉取一个rtsp流并显示
#### ffmpeg 封装拉流创建纹理
##### ffmpeg 拉流解码
- 创建Windwos共享内存 (*通过`CreateFileMapping`和`MapViewOfFile`实现跨进程数据传输*)
	- DX11/12纹理跨进程共享 提升传输效率
	- **DXGI共享表面**- 结合DXGI接口（如`IDXGISurface`）实现跨进程纹理共享，需与上述DirectX机制配
- 拉流解码H264->YUV或RGB具体格式见UE要求
- 通过sws_scale函数转换为RGB或RGBA以适应UE纹理标准
- 数据放置到共享内存（注意帧同步）
##### UE 纹理显示
###### 纹理创建和内存映射
- 创建**UTexture2D对象**，并分配内存
- 创建Windwos共享内存 (*通过`CreateFileMapping`和`MapViewOfFile`实现跨进程数据传输*)
- UE内存接口锁定纹理内存并写入数据(注意帧同步)
###### 纹理渲染那和显示
- 材质绑定
	- 创建UE材质，将动态纹理作为TextureSample节点输入
	- 在蓝图中通过SetTextureParameter Value 动态更新纹理材质纹理
- 渲染到UI或3D对象
	- UI显示:将材质应用于UImage空间，通过Slate或UMG系统渲染到屏幕
	- 3D对象显示： 将材质赋给StaticMesh或Actor ，实现视频在场景中的动态播放


##### 调试
**调试工具​**​：使用UE的`RenderDoc`插件检查纹理数据是否正确写入。
### 如何导入FAB素材

### 如何创建封装一个插件
