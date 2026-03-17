# 交叉编译器的命名规则<no value>

# 1. 命名规则
**`arch [-vendor] [-os] [-(gnu)[eabi|abi]] [-gcc]`**
- `arch`  - 体系架构，如arm，mips
- `vendor` - 工具链提供商
- `os` - 目标操作系统
- `abi|eabi`
	- `abi` - (Application Binary Interface for the ARM Architecture) 二进制应用程序接口。
	- `eabi` - 嵌入式应用二进制接口(Embedded Application Binary Interface). 指定了文件格式、数据类型和寄存器使用标准约定。

### 1.1.1. NOTICE：
1. 没有vendor时，用nonedaiti
2. 没有os支持是，也用none代替
3. 同时没有vendor和os支持时，只用一个none代替即可


# 2. 实例

## 2.1. arm-none-eabi-gcc
用于编译ARM架构的逻辑操作系统（适用于编译ARM的boot、kernel,不使用编译Linux应用,因为没有操作系统支持）。
一般适用于ARM7、Cortex-M、Cortex-R架构的芯片使用。

## 2.2. arm-none-linux-gnueabi-gcc

用于基于ARM架构的Linux系统，可用于编译ARM架构的boot、kernel以及Linux应用
基于gcc生成，使用glibc库。
一般适用于ARM9、ARM11、Cortex-A架构的芯片

## 2.3. arm-linux-gnueabihf-gcc
与 **arm-linux-gnueabi-gcc**的区别只不过是 对浮点数的默认参数的不一致（-mfloat-abi）

-mfloat-abi: 
- `soft`: 不用fpu进行浮点计算，即使有fpu浮点计算单元也不用，使用软件模式
- `softfp`： armel架构(对应**arm-linux-gnueabi-gcc**)采用的默认值，采用fpu进行浮点计算，但是传参用普通寄存器。对中断负载低。
- `hard`: armel架构(对应**arm-linux-gnueabihf-gcc**)的默认值， 采用fpu进行浮点计算，传参也用fpu中的浮点数寄存器，性能好，但是中断负载高


## 2.4. arm-eabi-gcc
Android ARM编译器

## 2.5. armcc
ARM公司推出的编译工具，功能同arm-none-eabi类似。
可以编译裸机程序（boot、kernel），但是不能编译Linux应用。
Keil MDK、ADS、RVDS、DS-5 中的编译器一般都是armcc

## 2.6. arm-none-uclinuxeabi-gcc 
适用于uClinux 系统，使用Glibc

## 2.7. arm-none-symbianelf-gcc
适用于symbian系统