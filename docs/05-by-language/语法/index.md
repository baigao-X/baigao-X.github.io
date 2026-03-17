# 语法<no value>

	GN 是一个为 [Ninja](https://ninja-build.org/) 生成构建文件的元构建系统。

[官方教程](https://gn.googlesource.com/gn/+/main/docs/quick_start.md)

## 关键字

GN 构建系统有很多关键字，主要分为以下几个类别：

### 目标类型 (Target Types)
#### 基本目标类型
- target 下面关键字都可以作为target_type,等价于 target(target_type, target_name) 
- executable - 生成可执行文件
- static_library - 生成静态库
- shared_library - 生成动态库/共享库
- source_set - 源文件集合，会生成一堆.o
- test 生成一个测试目标(本质也是一个可执行程序)
- loadable_module - 可加载模块 
- group - 元目标，用于组织依赖图 
- action - 自定义构建动作 
- copy - 文件复制操作 
### 目标属性 (Target Properties)
#### 源文件和依赖
- sources - 源文件列表
- deps - 依赖的目标列表 
- data_deps - 运行时数据依赖 
- public_deps - 公共依赖
- data - 数据文件
#### 配置相关
- configs - 配置列表 : define、cflags、ldflags用这个配置
- public_configs - 公共配置
- all_dependent_configs - 所有依赖目标的配置
- declare_args - 声明的变量有全局作用域，可以被构建过程引用，可以被其他.gni文件引用 //理解为cmake 中的option
#### 测试和可见性
- testonly - 标记仅用于测试 
- visibility - 可见性控制
#### 输出控制
- output_name - 输出文件名
- output_dir - 输出目录
- outputs - 输出文件列表
### 配置类型 (Config Types)
- config - 编译配置定义
### 工具链相关 (Toolchain)
- toolchain - 工具链定义
- tool - 工具定义
### 模板相关 (Templates)
- template - 自定义模板定义
- forward_variables_from - 变量转发
### 控制流和函数
- if / else - 条件语句 
- foreach - 循环语句
- import - 导入 .gni 文件
- assert - 断言 
- print - 调试输出 
### 内置变量和函数
- target_name - 当前目标名称
- current_toolchain - 当前工具链
- default_toolchain - 默认工具链
- host_os
- targert_os
- target_os - 目标操作系统 "mac|android|ios|fuchsia|chromeos|watchos"
- target_cpu - 目标 CPU 架构 "x86|x64|arm|arm64"
- is_debug - 是否为调试构建
- set_defaults - 设置目标参数
- set_default_toolchain - 设置了默认工具链，在定义大多数目标时就不再需要显式声明 toolchain 属性，GN 会自动使用这个默认值
### 路径和标签
- // - 源码根目录路径前缀
- : - 同一 BUILD.gn 文件内的目标引用
- (toolchain) - 指定工具链的目标引用
GN 的语法类似 Python，支持条件语句、循环和表达式 1 。你可以使用 gn help 命令获取完整的参考文档