# 对象模型<no value>

	深度探索C++对象模型 阅读笔记

# 1. 对象模型


## 1.1. C++ 对象模型（The C++ Object Model）

1. vtbl（vurtual table）： 每一个class 产生出一堆指向vurtual function的指针表格。
2. vptr： 每个class 对象安插一个指针，指向对应的vtbl
3. 每一个class 所关联的type_info object 也被vtbl所指出，传统上vtbl会被补充在所有显式member的最后，但是如今也有一些编译器将它放在class object 的最前端。

![[Pasted image 20231218090038.png]]
优点： 空间和存取时间的效率
缺点：如果应用程序程序本身未改变，但所用到的class 对象 的 nonstatic data members有所修改，就得重新编译。
### 1.1.1. 可以理解为以下两种模型的优点合并
#### 1.1.1.1. 简单对象模型（A Simple Object Model）

每个slot 指向一个members 
简单，但是执行效率较低。


![[Pasted image 20231218085758.png]]

#### 1.1.1.2. 表格驱动对象模型（A Table-driven Object Model）

将 data member 和function member 区分，分为两个 table 


![[Pasted image 20231218085831.png]]





### 1.1.2. 涉及继承的对象模型

- bptr(base class table) : 指向基类的表格。 该实现，不同编译器之间有极大差异。

![[Pasted image 20231218091157.png]]


### 1.1.3. 包含虚基类的对象模型


``` cpp
class X {};
class Y: public virtual X {};
class Z: public virtual X {};
class A: public Y, public Z {};

```

class X 即使是一个空类，也会被编译器安插一个char，使得这一class的两个objects在内存中拥有独一无二的地址

![[Pasted image 20231225092415.png]]

某些编译器针对 **Empty virtual base class**优化后，继承类不再保留空基类的1字节消耗。模型改为
![[Pasted image 20231225093046.png]]



## 1.2. 内存分布
![[Pasted image 20231219125134.png]]



# 2. 准则
- 处于同一个access section的数据，必定保证以其声明顺序出现在内存布局当中。然  
而被放置在多个access sections中的各个数据，排列顺序就不一定了

- 组合（composition），而  
非继承，才是把 C 和 C++结合在一起的唯一可行方法（conversion 运算符提供了一个十分便利  
的萃取方法）：
C struct在C++中的一个合理用途，是当你要传递“一个复杂的class object的全部或部分”到  
某个C函数去时，struct声明可以将数据封装起来，并保证拥有与C兼容的空间布局。然而这项保证  
只在组合（composition）的情况下才存在。如果是“继承”而不是“组合”，编译器会决定是否应该  
有额外的data me mbers被安插到base struct subobject之中（再一次请你参考3.4节的讨论以  
及图3.2a 和图3.2b ）。


## 2.1. Static Data Members

static Data Members 将会被编译器提取到class之外，并被视为一个只在class生命范围之内可见的global变量

# 3. 构造 Constructor

## 3.1. 默认构造 Defualt Constructor
	一个不需要参数就要可以执行的构造。可以是user-declared的也可以是编译器自己产生的。

在被需要的时候，被**编译器** 产生出来。
- 注意:被需要指的是**编译器需要**，而非一个程序的需要。如一个data members 需要被初始化，这是程序需要，需要由程序员实现。
- 如果没有任何的**user-declared constructor**，那么编译器会隐式(implicitly)地声明一个 **default constructor**。 一个被隐式声明出来的 default constructor 将会是 **trivial**(无用的)。
- 如果一个class没有任何constructor，但是**内含**一个member object，而后者有default constructor。那么这个class的 **implicit default constructor**才是**nontrivial**（有用的）的。
- 仅一个**notrivial default constructor**才被认为是编译器所需要的。
- 如果一个class已经有constructor，但是**内含**一个member object，而后者有default constructor。编译器会扩展已经存在的constructors，在其中安插一些代码，在user code执行之前调用member object 的default constructors。

如以下示例：
	编译器会隐式生成Zoo的 default constructor，是 trivial的。因为对str对象初始化是程序员的需要。
	编译器会隐式生成Bar的 default constructor，是 nontrivial的。因为对foo对象调用默认构造函数是编译器的需要。
	
``` cpp
class Foo {public:Foo(), Foo(int) ...}
class Zoo {char *str;};
class Bar {public:Foo foo; char *str;};

```

- C++语言要求  以“member objects 在class中的声明顺序”来调用各个constructors。

- class 继承一个virtual function，编译器将会合成一个 default constructor。 用与生成 vtbl和初始化vptr。
- class继承自一个继承串链，其中有一个或多个virtual base classes，编译器将会合成一个default constructor。用于初始化bptr。

## 3.2. 拷贝构造 copy constructor
	MyClass(const MyClass& that);

- 若没有一个 **explicit  copy constructor**,编译器会默认生成一个 **implicitly declared** 或**implicitly defined**。
- 决定一个 copy constructor 是否为 trivial 的标准在于 class 是否展现出所谓的“**bitwise copy semantics（位逐次拷贝)**"。 在以下场景，一个class 不会展现出 bitwise copy semantics，因此编译器需要取生成一个  **nontrivial copy constructor**
	 1. 当class **内含或继承**一个object，该对象存在一个copy constructor（无论是user -declared的还是编译器合成的）
	 2. 当一个class声明了一个或多个 virtual functionis时。（因为一个基类对象可能以他的继承类对象来调用copy constructor）
	 3. 当一个class 派生自一个继承串链，其中有一个或多个 virtual base classed时。（因为一个基类对象可能以他的继承类对象来调用copy constructor）
	
## 3.3. 移动构造 copy constructor
	MyClass(MyClass&& that);
## 3.4. 赋值运算符 assignment operator
	MyClass& operator=(const MyClass& that);
## 3.5. 移动赋值运算符 move assignment operator
	MyClass& operator=(MyClass&& that);


## 3.6. 成员初始化队列（member initialization list）
	本质不是一组函数调用。
	编译器会根据initalizaion list ，用适当的顺序在constructor之间安插初始化操作（并且在任何explicit user code 之前）。
	list 中members的初始化顺序，由定义顺序决定，而非initialization list中的顺序。（因此部分编译器在当initialization list 中顺序和定义顺序不相符时，会提供警告）

以下情况必须通过初始化队列进行初始化：
	1. 初始化一个reference member
	2. 初始化一个const member
	3. 调用一个base clase 的constructor，而它拥有一组参数时
	4. 调用一个member clase 的constructor，而它拥有一组参数时

在初始化队列中初始化一个object往往会在constructor中调用更有效率

如
``` cpp
class Word {
	String _name;
	int _cnt;

public:
	Word() {
		_name = 0;
		_cnt = 0;
	}
}
======== 经过编译器转换
	Word() {
		_name.String::String(); //调用String 的 default constructor
		String temp = String(0); //产生一个临时性对象
		_name.String::operator=(temp);  //“memberwise”地拷贝_name

		temp.String::~String();   //销毁临时性对象

		_cnt = 0;
	}
```
而
``` cpp
	Word::Word() :_name(0) {
		_cnt = 0;
	}
======== 经过编译器转换
	Word() {
		_name.String::String(0); //调用String 的 constructor
		_cnt = 0;
	}
```
## 3.7. conversion 转换运算符：
	事实上关键词explicit 之所以被导入这个语言  ，就是为了给程序员提供一种方法，使他们能够制止“单一参数的constructor”被当做一个convers  ion运算符



# 4. 编译器优化

## 4.1. 临时性object 策略

### 4.1.1. 参数的转换

``` cpp
void foo(X x0);

X xx;
// ... 
for (xx);
======== //编译器会转换成
//将入参的class X object 转换成 class X reference
//并导入一个临时object，传递他的reference作为参数
void f(X &x0);
X xx;

X __temp0;
__tempx.X::X(xx); //copy constructor
for(__tmep0)
```
### 4.1.2. 返回值的转换  NRV(Named Return Value)优化
	 一般情况下能优化执行效率
	 缺少一个copy constructor，将会无法触发NRV优化
	 因此如果你的编译器提供NRV优化，而且你希望开启他，那么需要提供一个 copy constructor，即使他是 trival的

``` cpp

X bar() {
	X xx;
	//.... 处理xx
	return xx;
}

======== //编译器会转换成
//将入参的class X object 转换成 class X reference
//并导入一个临时object，传递他的reference作为参数
void bar (X& __result) {
	X xx;
	xx.X::X(); //调用default constructor
	//.... 处理xx
	__result.X::xx(xx); //调用 copy constructor 调用操作
	return;
}
```

# 5. 多态：

- 编译器在（1 ）初始化及（2 ）指定（assignment）操作（将一个cla  
ss object指定给另一个class object）之间做出了仲裁。编译器必须确保如果某个object含有一个  或一个以上的vptrs，那些vptrs的内容不会被base c lass object初始化或改变