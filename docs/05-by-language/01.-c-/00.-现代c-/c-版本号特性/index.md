# C++ 版本号特性<no value>

## C++ 11
	gcc 4.8.1开始支持
	
- lambda
- 右值引用与移动语义
- 智能指针
- auto
- constexpr 常量表达式
- decltype (类型推导)： `deltype(n) a = n;`
- 委托构造函数: （通过初始化类表调用同一个类的其他构造函数）

## C++ 14
	gcc 6.1开始支持
	
- lambda 支持auto
- 变量模板
- 函数返回值推测 auto

## C++ 17
	gcc 7完全支持,gcc5 实验性支持
	
- std::any
- std::variant 类型安全完全体
 - std::filesystem 文件系统库
 - 折叠表达式(args + ...)

## C++ 20
	gcc10 完全支持,gcc8实验性支持

- 约束模板参数(requires)
- 模块 代替头文件(import module)： 提高编译效率，增强符号隔离
- 协程(coroutines): co_await 异步支持
- constexper 虚函数
## C++ 23
	gcc12部分支持
	
- this显示传递
- 多维下标运算符：`operator[]`支持多参数（`m[1,2]`）
- std::expected: 增强错误处理
