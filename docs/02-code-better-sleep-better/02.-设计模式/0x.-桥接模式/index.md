# 0X.  桥接模式<no value>

## Pimpl惯用法
	pointer to implementation”（指向实现的指针）

由于C++语言特性，如果对外开放一个类，即使是private成员，也需要声明在公有头文件中。 
使用该惯用法可以在公有接口中完全隐藏内部细节。支持将私有成员数据和方法，从.h转移到.cpp文件。

```  mermaid
classDiagram


class Interface {
	- IMpl *impl //可以优化成智能指针
	+ Function1() void
}

class Impl {
	- int privateData
	- privateFunctiont1() void
}

Interface "1" *-- "1" Impl : Composition
```


### 优点

- 信息隐藏 
只显示需要的公有属性，让使用者更容易查看，同时降低了耦合，带来了更好的二进制兼容性  

- 加速编译  
    将实现相关头文件移入.cpp，API的引用层次降低，会导致编译时间减少。
	
- 惰性分配  
    Impl类可以在需要时再构造，而不必在接口类构造时立即构造。

### 注意点

#### 虚函数
不能在impl类中隐藏private虚方法。virtual方法必须出现在接口类中，以保证任何派生类都能覆盖它。

#### 互指
虽然可以将接口类传递给Impl类的方法，但必要时，可以在Impl类中增加指回接口类的指针，便于Impl类调用公有方法。

#### 复制
当没有为类显式定义copy构造函数、assignment运算符（operator=）时，C++编译器会默认创建（trivial版本）。但这种trivail版本的copy构造函数、assigment运算符，只能执行***浅复制***。显然不利于使用Pimpl惯用法，因为如果客户复制了接口类对象，那么2个对象就会指向同一个Impl实现类对象，析构时，就会删除同一个Impl对象2次，从而可能导致程序崩溃。
可以采用以下方案

##### 禁止复制类。  

如果不打算让用户创建接口类对象副本，可以将对象声明为不可复制。 禁止编译器生成默认copy函数，有以下几种方法：

- 将方法设为private，禁止客户调用；
- 如果使用Boost库，可以让接口类继承自boost::noncopyable；
- C++11以后，可以将方法声明为"=delete"。

##### 显式定义复制语义。  
如果希望客户能复制采用Pimpl的对象，就应该声明并定义自己的copy构造函数、assignment运算符，进行对象的深拷贝，创建Impl对象的副本。


### C语言中使用Pimpl惯用法

``` c
/* autotimer.h */
typedef struct AutoTimer *AutoTimerPtr;  

AutoTimerPtr AutoTimerCreate(); 
void AutoTimerDestroy(AutoTimerPtr ptr);
```