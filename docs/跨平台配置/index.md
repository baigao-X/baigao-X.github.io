# 跨平台配置<no value>

## 0.1. 主流操作系统(工具链自带的宏)

### 0.1.1. UNIX System V
▪ A/UX ▪ AIX ▪ HP-UX ▪ IRIX ▪ LynxOS ▪ SCO OpenServer ▪ Tru64 ▪ Xenix ▪ Solaris ▪ OS/2

### 0.1.2. BSD UNIX-386BSD
▪ BSD/OS ▪ FreeBSD ▪ NetBSD ▪ NEXTSTEP ▪ Mac OS X ▪ iOS ▪ OpenBSD ▪ SUN OS▪ OpenSolaris

#### 0.1.2.1. APPLE 系列
- `__APPLE__ && __MACH__`  :  

#### 0.1.2.2. FreeBSD 系列
- `__FreeBSD__`  :  
- `__FreeBSD_kernel__`  :   从FreeBSD 8.3,9.1 10.0.1 开始

#### 0.1.2.3. Solaris 系列
- `sun`  :  
- `__sun`  :
-


### 0.1.3. UNIX-Like
▪ GNU ▪ Linux ▪ Android ▪ Debian ▪ Ubuntu ▪ Red Hat ▪ Linux Mint ▪ Minix ▪ QNX ▪ GNU/Linux ▪ GNU/Hurd ▪ Debian GNU/Hurd ▪ GNU/kFreeBSD ▪ StartOS

- `__linux__` :
- `linux` : 已过时(不符合POSIX标准)   
- `__linux` : 已过时(不符合POSIX标准)   

#### 0.1.3.1. GNU/Linux 
- `__gnu_linux__`

#### 0.1.3.2. Android
- `__ANDROID__`


### 0.1.4. Windows

- `_WIN16`: 16bit
- `_WIN32`: 32bit/64bit
- `_WIN64`:  64bit

#### 0.1.4.1. For DOS

▪ Windows 1.0 ( 1985) ▪ Windows 2.0 ( 1987) ▪ Windows 2.1 ( 1988)  
▪ windows 3.0 ( 1990) ▪ windows 3.1 ( 1992) ▪ Windows 3.2 ( 1994)

#### 0.1.4.2. Win 9x

▪ Windows 95 ( 1995) ▪ Windows97 ( 1996) ▪ Windows 98 ( 1998)  
▪ Windows 98 SE ( 1999) ▪ Windows Me ( 2000)

#### 0.1.4.3. NT系列

##### 0.1.4.3.1. 早期版本

▪ Windows NT 3.1 ( 1993) ▪ Windows NT 3.5 ( 1994) ▪ Windows NT 3.51 ( 1995)  
▪ Windows NT 4.0 ( 1996) ▪ Windows 2000 ( 2000)

##### 0.1.4.3.2. 客户端

▪ windows xp ( 2001) ▪ Windows Vista ( 2005) ▪ Windows 7 ( 2009)  
▪ Windows Thin PC ( 2011) ▪ Windows 8 ( 2012) ▪ Windows RT ( 2012)  
▪ Windows 8.1 ( 2013)

##### 0.1.4.3.3. 服务器

▪ Windows Server 2003 ( 2003) ▪ Windows Server 2008 ( 2008)  
▪ Windows Home Server ( 2008) ▪ Windows HPC Server 2008 ( 2010)  
▪ Windows Small Business Server ( 2011) ▪ Windows Essential Business Server  
▪ Windows Server 2012 ( 2012) ▪ Windows Server 2012 R2 ( 2013)

##### 0.1.4.3.4. 特别版本

▪ Windows PE ▪ Windows Azure  
▪ Windows Fundamentals for Legacy PCs

#### 0.1.4.4. 1.4.4嵌入式系统

▪ Windows CE ▪ Windows Mobile ( 2000) ▪ Windows Phone ( 2010)

### 0.1.5. DOS

dos，是磁盘操作系统的缩写，是个人计算机上的一类操作系统。


## 0.2. 主流编译器


### 0.2.1. MSVC(Microsoft Visual C++)
MSVC是微软Windows平台Visual Studio自带的C/C++编译器。
- `_MSC_VER`:  编译器版本号

|  IDE版本/发布时间  |  \_MSC_VER\_  | 工具级版本|
|  ----  | ----  | ---- |
| 1.0  | 800 ||
| 3.0  | 900 ||
| 4.0  | 1000 ||
| 4.2  | 1020 ||
| 5.0  | 1100 ||
| 6.0 (Visual C++ 6.0)  | 1200 |V60|
| 6.0 SP6  | 1200 ||
| 7.0  | 1300 |V70|
| 7.1(2003)  | 1310 |V71|
| 8.0(2005)  | 1400 |V80|
| 9.0(2008)  | 1500 |V90|
| 9.0 SP1  | 1500 ||
| 10.0(2010)  | 1600 |V100|
| 10.0(2010) SP1  | 1600 ||
| 11.0(2012)  | 1700 |V110|
| 12.0(2013)  | 1800 |V120|
| 14.0(2015)  | 1900 |V140|
| 14.0(2015 Update 1)  | 1900 |
| 14.0(2015 Update 2)  | 1900 |
| 14.0(2015 Update 3)  | 1900 |
| 15.0(2017)  | 1910 |V141|
|16.0(2019) | 1920|V142|
| 17.0(2022)| 1930|V143|

### 0.2.2. GCC 
GCC原名GNU C Compiler，后来逐渐支持更多的语言编译（C++、Fortran、Pascal、Objective-C、Java、Ada、Go等），所以变成了GNU Compiler Collection（GNU编译器套装），是一套由GNU工程开发的支持多种编程语言的编译器。GCC是自由软件发展过程中的著名例子，由自由软件基金会以GPL协议发布，是大多数类Unix（如Linux、BSD、Mac OS X等）的标准编译器，而且适用于Windows（借助其他移植项目实现的，比如MingW、Cygwin等）。GCC支持多种计算机体系芯片，如x86、ARM，并已移植到其他多种硬件平台。

- `__GNU__`

### 0.2.3. Cygwin
Cygwin是一个Windows下Unix-like模拟环境，具体说就是Unix-like接口（OS API，命令行）重定向层，其目的是不修改软件源码仅重新编译就可以将Unix-like系统上的软件移植到Windows上（这个移植也许还算不上严格意义上的无缝移植）。始于1995年，最初作为Cygnus软件公司工程师Steve Chamberlain的一个项目。

- `__CYGWIN__`


### 0.2.4. MinGW / MinGW-w64
MingW（Minimalist GNU on Windows）是一个Linux/Windows下的可以把软件源码中Unix-like OS API调用通过头文件翻译替换成相应的Windows API调用的编译环境，其目的和Cygwin相同。从而把Linux上的软件在不修改源码的情况下编译为可直接在Win下执行的exe。

- `__MINGW32__` : MinGW32、MinGW-w64 32bit、MinGW-w64 64bit
- `__MINGW64__` : MinGW-w64 64bit

### 0.2.5. Oracle Solaris Studio

- `__SUNPRO_C`: C编译器
- `__SUNPRO_CC` : C++编译器


## 0.3. 主流架构

### 0.3.1. Inter x86

- `i386`、`__i386`、`__i386__`:  GNU C 定义
- `__i386__`:  Sun Studio 定义
- `__i386__` 、`__IA32__`:  Stratus VOS C 定义
- `_M_I86` : 仅针对16位体系结构定义，由Visul Studio，Digital Mars 和watcom C/C++ 定义
- `_M_IX86` : 仅针对32位体系结构定义，由Visul Studio，Inter C/C++, Digital Mars 和watcom C/C++ 定义
- `__X86__`:  Watcom C/C++ 定义
- `_X86_`:  MinGW32 定义
- `__THW_INTER__`:  XL C/C++ 定义
- `__I86__`:  Digital Mars 定义
- `__INTER__`:  CodeWarrior 定义
- `__`386:  Diab 定义

### 0.3.2. Inter Itanium(IA-64)

- `__ia64__`、`__IA64`、`__IA64__`:  GNU C 定义
- `__ia64`:  HP aCC 定义
- `_M_IA64`:  Visual Studio 定义,  Inter C/C++ 定义
- `__itanium__`:  Inter C/C++ 定义

### 0.3.3. ARM

- `__arm__` : GNU C 和 RealView定义
- `__thumb__` : GNU C 和 RealView定义 ，在Thumb模式下定义
- `__TARGET_ARCH_ARM` : RealView定义
- `__TARGET_ARCH_THUMB`: RealView定义 在Thumb模式下定义
- `__ARM`: ImageCraft C定义 
- `_M_ARM`:  Visual Studio定义 
- `_M_ARMT`:  Visual Studio 在Thumb 模式定义 

### 0.3.4. ARM64
- `__aarch64__` 由GNU C定义

## 0.4. QT 跨平台定义
在 qsystemdetection.h 头文件









