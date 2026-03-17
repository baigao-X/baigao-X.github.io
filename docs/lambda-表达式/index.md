# Lambda 表达式<no value>

匿名函数


## 0.1. 基础语法
``
``` cpp
[=] () mutable throw() -> int
{
	int n = x + y;
	y = x;
	y = n;
	return n;
}

```

### 0.1.1. `[=]` capture子句

在C++规范中也称为Lambda引导

- `[]` : 表示不访问封闭范围内的变量
- `[=]` : 表示通过值不过所有变量
- `[$]` : 表示通过引用捕获所有变量
- `[&total, factor]` : 通过引用捕获total, 通过值捕获factor变量
- `[&, factor]` : 通过引用捕获total, 通过值捕获factor变量
- `[=, &total]` : 通过引用捕获total, 通过值捕获factor变量
`

### 0.1.2. `()` 参数列表(可选)

也称为Lambda声明符
``` cpp
auto y = [] (auto first, auto second)
{
    return first + second;
};
```

### 0.1.3. `mutable` 规范(可选)

 **`mutable`** 规范允许在 在Lambda表达式中对一个值引用的值进行修改（但是不会反映到外部）

### 0.1.4. `throw()` exception-specification(可选)

### 0.1.5. `int` trailing-return-type(可选)

如果 Lambda 体仅包含一个返回语句，则可以省略 Lambda 表达式的 return-type 部分。
在表达式未返回值的情况下。 如果 lambda 体包含单个返回语句，编译器将从返回表达式的类型推导返回类型。 否则，编译器会将返回类型推导为 **`void`**。
``` cpp
auto x1 = [](int i){ return i; }; // OK: return type is int
auto x2 = []{ return{ 1, 2 }; };  // ERROR: return type is void, deducing
                                  // return type from braced-init-list isn't valid
```

### 0.1.6. Lambda 体

普通函数和 lambda 表达式的主体均可访问以下变量类型：
- 从封闭范围捕获变量，如前所述。
- 参数。
- 本地声明变量。
- 类数据成员（在类内部声明并且捕获 **`this`** 时）。
- 具有静态存储持续时间的任何变量（例如，全局变量）。


# 1. 
C++ 实际转换成一个没有数据成员的类。因此占用的大小是1