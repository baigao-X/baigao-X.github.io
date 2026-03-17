# 模板<no value>

# 1. 关键字
- `template`
- `typename`
 如果模板定义中的名称是依赖于模板参数的限定名称，则必须使用关键字 ;如果限定名称不依赖，则为可选。
 
可由任何类型在模板声明或定义中的任何位置使用。
不允许在基类列表中使用它，除非作为模板基类的模板参数。 

``` cpp
template <typename T>
class C1 ： typename T::InnerType //错误
{}

template <typename T>
class C2 ： A<typename T::InnerType> //正确
{}
```

- `class` 等效与`typename`
``` cpp
template <typename T>
T minimum(const T& lhs, const T& rhs) {
	return lhs < rhs ? lhs : rhs;
}
```

### 1.1.1. 类型参数： 
对于函数模板，支持类型推导，因此允许不显示地指定类型。

``` cpp
int a = get_a();
int b = get_b();
//显示地指定类型进行实例化
int i = minimum<int>(a, b);

//自动推导
int i = minimum(a, b);
```
对于类型参数数量没有限制，支持省略运算符(...)

``` cpp
// 多个类型参数
template <typename T, typename U, typename V>
class Foo {
};

// ... 省略运算法
template <typename ... Arguments>
class Foo (Arguments ... args){
	//...
};

Foo<> foo1;
Foo<int> foo2;
Foo<int, bool> foo3;
```

### 1.1.2. 非类型参数(值参数): 

非类型参数作为模板参数传入时必须是常量或者常量表达式

``` cpp
template <typename T, size_t L>
class Myarray {
	T arr[L];
}

Myarray<MyClass*, 10> arr;
```

c++17之后支持使用auto进行非类型模板参数的类型推导.
``` cpp
template <auto x> 
consterpr auto constant = x;

auto v1 = constant<5>;
auto v2 = constant<true>;
```

### 1.1.3. 模板参数：
支持使用模板作为模板参数
``` cpp
template <typename T, template<typename U, int I> class Arr> 
class Myclass2{
	T t;
	Arr<T, 10> a;
	U u;         //错误，模板U不在当前作用域
}
```

### 1.1.4. 默认模板自变量:
可以理解为针对模板参数的默认参数

``` cpp
template <class T, class Allocator = allocator<T>> 
class vector;

vector<int> ints;
vector<int, MyAllocator> myInts;

```

如果每个参数都有默认参数，可以使用空尖括号来使用默认值
``` cpp
template<typename A = int, typename B = doubel>
class Bar {
	//...
}

Bar<> bar;
```

### 1.1.5. 模板特例化：
针对指定的参数构造特例的代码。

#### 1.1.5.1. 偏特化(部分专用化 partial_specialization)
只有部分模板参数是特殊指定的。

##### 1.1.5.1.1. 模板有多个类型，但只有一部分需要特例化
``` cpp
template <typename T, typename V>
class MyMap {}；

template <typename V>
class MyMap<string, V> {};
```

##### 1.1.5.1.2. 模板只有一个类型，但指针、引用、指向成员的指针或函数指针类型需要专用化。

``` cpp
template <typename T>
class MyMap {};

template <typename T>
class MyMap<T*> {};
```


#### 1.1.5.2. 全特化(完整专用化)
所有的模板参数都是特殊指定的

``` cpp
template 
class MyMap<string, int> {};
```


## 1.2. 函数模板
### 1.2.1. 显示实例化
当首次为每个类型调用函数模板时，编译器会创建一个实例化(生成定义)。

``` cpp
template <class T> 
void f(T) {}

int a = get_a();
int b = f<int>(a);
```

也可以显示地声明来显示实例化。
``` cpp
template <class T> 
void f(T) {}

template void f<int> (int);
```
使用extern关键字来防止自动实例化
``` cpp
template <class T> 
void f(T) {}

extern template void f<int> (int);
```

### 1.2.2. 实例化的顺序

当存在多个函数(包括普通函数和函数模板)符合匹配时，使用以下书优先级
1. 非函数模板
2. 完全匹配的函数模板

当存在多个模板的符合实例化时，存在一定的符合度优先顺序。
1. `const T* > T*`
2. `T* > T`

## 1.3. 类模板
类声明中定义的函数被视为内联函数，并且始终实例化。
类模板的所有函数都是***泛型函数***(无论是否是模板函数)，但不一定是***成员模板***。

类外定义需要像***函数模板***一样定义他们。
``` cpp
template <class T, int i>
class MyStack {
	public: MyStack(void);
};

tepmlate <class T, int i> 
MyStack<T, i>::MyStack(void) {

}
```

## 1.4. 成员模板
分为***成员函数模板***和***嵌套类模板***

### 1.4.1. 成员函数模板
在类或者类模板中定义的模板，被称为**成员函数模板**。
局部类不允许具有成员模板。
``` cpp 
template <class T> 
class A {
public:
	template <class U>
	void mf(const U &u);
}

template <class T> template <typename U>
void A<T>::mf(const U &u) {
}
```

成员函数模板不能是虚函数。
当使用与基类虚拟函数相同的名称声明虚拟函数时，无法替代基类中的虚拟函数。

### 1.4.2. 嵌套类模板
作为类的成员模板称为**嵌套类模板**。

#### 1.4.2.1. 普通类中的嵌套类模板
``` cpp
class A {
	template <class T> 
	struct Y {
		T m_t;
	}
	Y<int> yInt;
}
```
#### 1.4.2.2. 类模板中的嵌套类模板
``` cpp
template <class T>
class A {
	template <class U>
	class B {
	public: 
		U& Value();	
	}
}


template <class T>
template <class U>
U& A<T>::B<U>::Value() {

}
```

### 1.4.3. 模板友元
如果原始函数模板是友元，则函数模板专用化将自动为友元。
也可以只将模板的专用版本声明为友元。
``` cpp
template <class T> 
class A {
public:
	friend A<T>* combine(A<T>& a1, A<T>& a2);
}

template <class T>
A<T>* combine(A<T>& a1, A<T>& a2) {

}
```


## 1.5. 模板名称：

模板定义中有以下三种类型的名称:

本地声明的名称，包括模板本身的名称以及在模板定义中声明的任何名称。
- 模板定义之外的封闭范围中的名称。
- 在某种程度上依赖于模板自变量的名称（称为依赖名称）。
	- 模板自变量本身 `T`
	- 具有包括依赖类型的限定的限定名 `T::myType`
	- 具有非限定部分表示依赖类型时的限定名 `N::T`
	- 其基类型属于依赖类型的不变或可变类型 `const T`
	- 基于依赖类型的指针、引用、数组和函数指针类型`T*, T&, T[10], T(*)()`
	- 大小基于模板参数的数组 ` template <int arg>  class X { int x[arg] ; // dependent type  }
	- 从模板参数构造的模板类型: `T<int>， MyTemplate<T>`
  
### 1.5.1. 依赖类型的名称解析
##### 1.5.1.1.1. 使用 **`typename`** 作为模板定义中的限定名称，以告知编译器给定的限定名称标识一个类型

```  cpp
// template_name_resolution1.cpp
#include <stdio.h>
template <class T> class X
{
public:
   void f(typename T::myType* mt) {}
   //void f(T::myType* mt) {} 错误
};

class Yarg
{
public:
   struct myType { };
};

int main()
{
   X<Yarg> x;
   x.f(new Yarg::myType());
   printf("Name resolved by using typename keyword.");
}
```

 

 