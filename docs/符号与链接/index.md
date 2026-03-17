# 符号与链接<no value>

## 0.1. dumpbin 工具

1、对于静态库  
dumpbin -SYMBOLS 库文件  
在命令行中使用上述指令查看库中的显示函数和数据对象。

2、对于动态库  
dumpbin -[EXPORTS](https://so.csdn.net/so/search?q=EXPORTS&spm=1001.2101.3001.7020) 库文件  
在命令行中使用上述指令查看库中的显示函数和数据对象。

3. dumpbin /dependents +dll/exe文件路径