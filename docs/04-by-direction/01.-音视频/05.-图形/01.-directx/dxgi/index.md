# Dxgi<no value>

(DirectX Graphics Infrastructure) 是微软DirectX中的底层图形接口
负责管理显示适配器、显示模式、资源分配等硬件相关操作。
它作为**Direct2D、Direct3D与其他图形API的公共基础层**，提供跨API的通用功能

---------- 



### **1. DXGI的核心功能**

1. **设备与适配器管理**
    
    - 枚举物理GPU（`IDXGIAdapter`）、检测显示器信息（`IDXGIOutput`）。
    - 支持多GPU系统（如NVIDIA SLI/AMD CrossFire）的配置。
2. **交换链（SwapChain）控制**
    
    - 创建和管理前后缓冲区（`IDXGISwapChain`），实现双缓冲/三缓冲渲染，避免画面撕裂。
    - 支持全屏模式切换、垂直同步（VSync）等显示设置。
3. **资源共享与表面（Surface）操作**
    
    - 通过`IDXGIResource`接口在不同API（如Direct3D与Direct2D）间共享纹理、缓冲区等资源。
    - 提供`IDXGISurface`用于跨进程或跨API的显存数据访问。
4. **高动态范围（HDR）与显示色彩管理**
    
    - 在DXGI 1.5+中支持HDR元数据传递（如`SetHDRMetaData`）。
    - 管理显示器的色域、亮度范围等参数。

### 2. DXGI关键接口

| 接口               | 功能描述                                 |
| ---------------- | ------------------------------------ |
| `IDXGIFactory`   | 创建适配器、交换链，管理全屏窗口关联性。                 |
| `IDXGIAdapter`   | 表示物理GPU，获取显存、驱动版本等硬件信息。              |
| `IDXGIOutput`    | 关联显示器，获取分辨率、刷新率等显示模式。                |
| `IDXGISwapChain` | 控制前后缓冲区交换，支持Present操作和全屏切换。          |
| `IDXGIResource`  | 跨API共享资源（如纹理），支持查询资源属性。              |
| `IDXGIDevice`    | 连接DXGI与图形设备（如Direct3D设备），用于资源创建和互操作。 |

### IDXGIFactory
- **CreateDXGIFactory1()** : 创建

### IDXGISwapChain
它管理着用于显示到屏幕的一组缓冲区。交换链通常包含两个或多个缓冲区：
- 前台缓冲区（Front Buffer）：当前显示在屏幕上的内容
- 后台缓冲区（Back Buffer）：用于渲染下一帧的内容

- IDXGIFactory.**CreateSwapChain** 创建交换链
- **GetBuffer**(): 获取后台缓冲区，缓冲区内部数据存放的就是纹理数据
- **Present()** : 将后台缓冲区呈现到屏幕
- 