# 硬件编解码Api<no value>


API 名称	主要平台	主要推动者/支持者	备注
VAAPI	Linux, Unix-like	Intel (主要), AMD	Linux 平台的事实标准，尤其是对于开源驱动。
VDPAU	Linux, Unix-like	NVIDIA	由 NVIDIA 最早为其 Linux 闭源驱动开发。现在较为老旧，新应用更多转向 VAAPI 或 NVDEC。
NVDEC / NVENC	Linux, Windows	NVIDIA	NVIDIA 现代的、私有的硬件解码/编码 API，性能强大，通过其闭源驱动提供。
DXVA / D3D11 VA	Windows	Microsoft	Windows 平台的标准视频加速接口，集成在 DirectX 中。
VideoToolbox	macOS, iOS	Apple	苹果生态系统的官方硬件加速框架。



### VAAPI

- vainfo

### NVDEC/NVENC

