# Gpuview<no value>

GPUView (_GPUView.exe_) 是一个性能分析工具，可帮助开发人员分析 Windows 系统上的 GPU 和 CPU 活动。 它可用于诊断图形密集型应用程序（如游戏或多媒体软件）中的性能问题。
Windows Performance Toolkit (WPT)：的一部分


#### 使用方法
1. 打开cmd，运行GPUView 一并提供的log.cmd 脚本开启记录
3. 执行需要分析的进程
4. 再执行一遍log.cmd，关闭记录，并将记录合成生成为Merged.etl文件
5. 使用GPUView 打开Merged.etl文件 查看分析
