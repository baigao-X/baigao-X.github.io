# Perf<no value>


# 1. 常用命令

## 1.1. 采样

- 对一个正在运行的程序进行采样：
``` shell
perf record -p PID -g -- sleep 60
```

- 全新执行一个进程，并进行采样
``` shell
perf record -F 99 -g ./test -- sleep 60
```

## 1.2. 转换火焰图


```
//将采样的perf.data 文件
perf script -i perf.data &> perf.unfold

//将采样的perf.data 文件转换成火焰图
// 依赖：https://github.com/brendangregg/FlameGraph.git

./FlameGraph/stackcollapse-perf.pl perf.unfold &> perf.folded

./FlameGraph/flamegraph.pl perf.folded > perf.svg
```
# 2. 