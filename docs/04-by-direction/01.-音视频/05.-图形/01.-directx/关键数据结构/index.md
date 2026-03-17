# 关键数据结构<no value>

# 核心概念
资源与视图分离, 纹理数据是资源
DirectX 11 中有多种资源视图，每种都有特定的用途：
1. 着色器资源视图（SRV） ：用于着色器读取资源
2. 渲染目标视图（RTV） ：用于渲染输出
3. 深度模板视图（DSV） ：用于深度和模板测试
4. 无序访问视图（UAV） ：用于计算着色器的读写操作
同一个资源可以有多种不同的视图，这是 DirectX 11 资源绑定模型的核心概念。

# 渲染流程中的对象链
## 设备对象

- **ID3D11Device** : 负责创建所有 DirectX 资源
- **ID3D11DeviceContext** : 负责设置渲染状态和提交渲染命令
- ID3D11DeviceContext.**Draw()** : 执行渲染
- **D3D11CreateDevice()**：//指定API版本，创建 ID3D11Device 和ID3D11DeviceContext
## 纹理对象
- **ID3D11Texture2D**: 纹理对象,**ID2D1RenderTarget**是其类型的抽象。 直接存储像素数据，但无法直接绑定到渲染管线，需通过视图进行访问
- ID3D11Device.**CreateTexture2D**: 创建纹理
- **ID3D11ShaderResourceView** 使着色器能够访问纹理数据
- ID3D11Device.**CreateShaderResourceView**: 创建着色器资源视图
- **ID3D11RenderTargetView** 视图 将纹理作为渲染目标， 继承自 **ID3D11View**。 
- ID3D11Device.**CreateRenderTargetView**: 创建渲染目标试图，创建基于*交换链*后台缓冲区的渲染目标视图
- ID3D11DeviceContext.**ClearRenderTargetView**: 清空渲染
- ID3D11DeviceContext.**OMSetRenderTargets**: 设置渲染目标

## 着色器对象

- **ID3D11VertexShader** : 顶点着色器，处理顶点变换 （负责确认视频帧的显示区域）
- ID3D11Device.**CreateVertexShader**: 创建顶点着色器
- **ID3D11PixelShader** : 像素着色器，处理像素颜色计算（负责处理颜色空间转换和特效）
- ID3D11Device.**CreatePixelShader**: 创建像素着色器
- **ID3D11InputLayout**: 顶点输入布局，用于桥接顶点缓冲区和顶点着色器
- ID3D11Device.**CreateInputLayout**: 创建顶点输入布局对象
- **ID3D11Buffer** : 常量缓冲区，向着色器传递参数
## 采样器对象
  负责**纹理数据读取规则的定义**
- **ID3D11SamplerState** : 控制纹理采样方式
- ID3D11Device.**CreateSamplerState** 创建采样器对象
## 混合状态对象
负责透明度处理，颜色混合效果，多层渲染)
- **ID3D11BlendState** : 控制像素混合方式
- ID3D11Device.**CreateBlendState** 创建混合状态对象


# 渲染流程
## 初始化阶段
1. 创建 D3D11 设备和上下文
2. 创建渲染目标视图
3. 创建着色器和采样器
4. 创建顶点缓冲区和输入布局
## 帧数据处理阶段
1. 将视频数据上传到纹理
2. 根据视频格式选择合适的像素着色器
3. 设置常量缓冲区参数（如色彩空间转换矩阵）
## 渲染阶段
通过 TextureCopyRect 或 TextureResizeShader 方法执行实际渲染：

1. 设置渲染目标和视口
2. 设置顶点和像素着色器
3. 设置着色器资源和采样器
4. 设置混合状态
5. 执行绘制操作