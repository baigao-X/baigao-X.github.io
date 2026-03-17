# 概述<no value>


	 以下栈回溯基于C进程而言


# 1. 跟踪(Tracing) :
	Tracing功能的基础是符号(symbols), 即目标文件中的函数信息。
	但是显示的时符号，非直接函数名
	 若无符号会直接提示没法tracing
## 1.1. 符号（symbols）
	符号可以在目标文件中，也可以存放在单独的文件中
	symbols 的存在不需要 -g参数，默认编译不strip就会有
	 一般发行进程会单独提供一个符号文件搭配运行进程用于调试。
- .dynsym 动态符号表
- .symtab 局部(local)符号表 (本地符号常常会被剥离以减少大小)

# 2. 栈展开(stack unwinding)
有两种方式：

## 2.1. Debug symbol: DWARF 格式

- 依赖DWARF格式的 debug symbols， bpf工具对着这种方式兼容性不好，无法展开DWARF类型的调用栈 即`.debug_info`
- 编译时需要使用 -g 参数生活的调试信息。会提供符号名和其对应的文件名以及行号
-  提供调用栈展开(stack unwinding)的支持
 - 一般占用空间很大

## 2.2. frame pointer (栈帧)
- 一种时依赖frame pointer(栈帧)，bpf只支持这种。
- 依赖程序编译时不能开启`-fomit-frame-pointer`，开启该选项会影响性能。



