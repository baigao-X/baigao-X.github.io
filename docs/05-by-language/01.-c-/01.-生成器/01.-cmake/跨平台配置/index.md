# 跨平台配置<no value>

# 1. CMAKE_SYSTEM_NAME 变量
 系统变量
```cmake
if (CMAKE_SYSTEM_NAME MATCHES "Windows")
elseif (CMAKE_SYSTEM_NAME MATCHES "Linux")
elseif (CMAKE_SYSTEM_NAME MATCHES "Darwin")
endif()
```


# 2. 关于系统的一些简写变量
-  `WIN32` : 32 位 Windows 和 64 位 Windows 都为真
- `APPLE` : 对于所有苹果产品（MacOS 或 iOS）都为真
- `UNIX`  : 对于所有 Unix 类系统（`FreeBSD, Linux, Android, MacOS, iOS`）都为真
- `ANDROID` : 
- `IOS`  : 
- `MacOs`  :

```cmake
if (WIN32)
elseif (UNIX AND NOT APPLE)
elseif (APPLE)
endif()
```


# 3. CMAKE_CXX_COMPILER_ID 变量
编译器
```cmake
if (CMAKE_CXX_COMPILER_ID MATCHES "GNU")
elseif (CMAKE_CXX_COMPILER_ID MATCHES "NVIDIA")
	#NVIDIA 的CUDA编译器
elseif (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
	#Mac使用
elseif (CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
	#Visul Studio 使用的编译器 
endif()
```

# 4. 关于编译器的一些简写变量

```cmake
if (MSVC)
elseif (CMAKE_COMPILER_IS_GNUCC)
else()
endif()
```

# 5. 关于系统位数
``` cmake
IF(CMAKE_CL_64)
ELSE(CMAKE_CL_64)
ENDIF(CMAKE_CL_64)
```

# 6. CMAKE_GENERATOR
生成器
- `Ninja` :