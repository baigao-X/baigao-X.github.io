# 成果物文件<no value>


# 1. 成果物文件

## 1.1. pdb

exe文件对应的调试信息

## 1.2. exp
dll文件的导出文件
文件是指导出库文件的文件，简称导出库文件，它包含了导出函数和数据项的信息。当LIB创建一个导入库，同时它也创建一个导出库文件。如果你的程序链接到另一个程序，并且你的程序需要同时导出和导入到另一个程序中，这个时候就要使用到exp文件(LINK工具将使用EXP文件来创建动态链接库)。

## 1.3. ilk

## 1.4. 增量链接文件
Microsoft Visual Studio使用的文件 [IDE](https://techterms.com/definition/ide) 用于开发Windows应用程序； 包含已编译的可执行文件的链接器数据（例如， [。EXE](https://m33.wiki/extension/exe.html) or [。DLL](https://m33.wiki/extension/dll.html)）; 支持更快的项目编译和链接，特别是对于大型项目。

# 2. 编译模式差别

不能将debug和release版的DLL混合在一起使用。debug都是debug版，release版都是release版。

## 2.1. Debug 版本：

- /MDd /MLd 或 /MTd 使用 Debug runtime library(调试版本的运行时刻函数库)
- /Od 关闭优化开关
- /D “_DEBUG” 相当于 #define _DEBUG,打开编译调试代码开关(主要针对assert函数)
- /ZI 创建 Edit and continue(编辑继续)数据库，这样在调试过程中如果修改了源代码不需重新编译
- /GZ 可以帮助捕获内存错误
- /Gm 打开最小化重链接开关，减少链接时间

## 2.2. Release 版本：  

- /MD /ML 或 /MT 使用发布版本的运行时刻函数库  
- /O1 或 /O2 优化开关，使程序最小或最快  
- /D “NDEBUG” 关闭条件编译调试代码开关(即不编译assert函数)  
- /GF 合并重复的字符串，并将字符串常量放到只读内存，防止被修改

## 2.3. 查看一个库是Debug还是Release
	dumpbin /headers 库名， 其machine字段会说明 是x86还是x64

	
dll可以用 `dumpbin /dependents 库名` 查看依赖库如果带D则为debug模式，否则为release模式
静态库lib 可以根据`dumpbin /all 库名` 搜索输出种的”Archive member name“，根据输出路径是Release还是Debug区分
导入库lib 只能根据dll信息来判断


# 3. 动态库和静态库区别
- 注意.lib 后缀可能是包含所有符号的静态库，也可能是仅包含DLL链接信息的导入库
	可以使用lib /list 库名来查看，显示出*.obj的是静态库，是*.dll的是动态库导入库，仅用于链接，运行时仍需要dll库 （lib命令在vs安装路径下）

# 4. 编译选项

## 4.1. MD/MDd/MT/MTd

决定运行时库时动态链接还是静态链接。 不同的运行时库（无论动态和静态的差异还是调试和非调试的差异）混用都会有问题。

MT：mutithread，多线程库，定义 _MT。编译器会从运行时库里面选择多线程静态连接库来解释程序中的代码，即连接LIBCMT.lib库

MTd：mutithread+debug，多线程调试版，定义 _MT 和 _DEBUG。连接LIBMITD.lib库

MD：MT+DLL，多线程动态库，定义 _MT 和 _DLL。连接MSVCRT.lib库，这是个导入库，对应动态库为MSVCRT.dll

MDd： MT+DLL+debug，多线程动态调试库，定义_MT 、_DLL和 _DEBUG 。 连接MSVCRTD.lib库，对应动态库为MSVCRTD.dll

- /MD 
	运行库的多线程DLL版本。定义 _MT 和 _DLL，并使编译器将库名 MSVCRT.lib 放入 .obj 文件中。用此选项编译的应用程序静态链接到 MSVCRT.lib。 此库提供允许链接器解析外部引用的代码的层。 实际工作代码包含在 MSVCR100.DLL, 中，该库必须在运行时对于与 MSVCRT.lib 链接的应用程序可用。

- /MDd 
	定义 _DEBUG、_MT 和 _DLL，并使应用程序使用运行库的调试多线程并特定于 DLL 的版本。 它还使编译器将库名 MSVCRTD.lib 放入 .obj 文件中。

- /MT
	使应用程序使用运行库的多线程静态版本。 定义 _MT 并使编译器将LIBCMT.lib 放入 .obj 文件中，以便链接器使用 LIBCMT.lib 解析外部符号。

- /MTd
	定义 _DEBUG 和 _MT。 此选项还使编译器将库名 LIBCMTD.lib 放入 .obj 文件中，以便链接器使用 LIBCMTD.lib 解析外部符号

![[Pasted image 20241017110103.png]]
#### 查看依赖库的运行时库依赖
##### 针对静态库
- dumpbin.exe /directives
	- 若存在 /DEFAULTLIB:MSVCRT， 表示/MD
	- 若存在 /DEFAULTLIB:MSVCRTD， 表示/MDd
	- 若存在 /DEFAULTLIB:LIBCMT， 表示/MT
	- 若存在 /DEFAULTLIB:LIBCMTD， 表示/MTd