# C++ 20 协程实现 Coroutine<no value>

协程就是一段可以**挂起(suspend)** 和 **恢复(resume)** 的程序(函数)
c++20以上推荐阿里的雅兰亭库(yalantinglibs)


# 1. 例程

``` cpp
#define __cpp_lib_coroutine

#include <iostream>
#include <coroutine>
#include <future>
#include <chrono>
#include <thread>

using namespace std::chrono_literals;

void Fun() {
    std::cout << 1 << std::endl;
    std::cout << 2 << std::endl;
    std::cout << 3 << std::endl;
    std::cout << 4 << std::endl;
}

struct Result {
    struct promise_type {
        std::suspend_never initial_suspend() {
            return {};
        }

        std::suspend_never final_suspend() noexcept {
            return {};
        }

        Result get_return_object() {
            return {};
        }

        void return_void() {

        }

        //    void return_value(int value) {
        //
        //    }

        void unhandled_exception() {

        }
    };
};

struct Awaiter {
    int value;

    bool await_ready() {
        return false;
    }

    void await_suspend(std::coroutine_handle<> coroutine_handle) {
        std::async([=](){
            std::this_thread::sleep_for(1s);
            coroutine_handle.resume();
        });
    }

    int await_resume() {
        return value;
    }
};

Result Coroutine() {
    std::cout << 1 << std::endl;
    std::cout << co_await Awaiter{.value = 1000} << std::endl;
    std::cout << 2 << std::endl;
    std::cout << 3 << std::endl;
    co_await std::suspend_always{};
    std::cout << 4 << std::endl;

    co_return;
};

//Result Coroutine(int start_value) {
//  std::cout << start_value << std::endl;
//  co_await std::suspend_always{};
//  std::cout << start_value + 1 << std::endl;
//
//  co_return 100;
//};

int main() {
    Coroutine();
    return 0;
}
```


# 2. 协程的返回值类型

如果一个函数的返回值满足协程的规则(即能够实例化模板类型 **\_Coroutine_traits** )，就会被编译成协程。
可以参见[[#1. 例程 | 例程]]中Result的定义

- 该_Ret 类型应该具有一个名为 **promise_type** 的类型。该类型可以直接定义在_Ret内部也可以通过using指向已经存在的其他外部类型。
- promise_type内部可以定义以下接口

### 2.1.1. std::suspend_never initial_suspend() 
 会在协程函数一开始执行，可以通过该接口来控制协程是否直接挂起
 
### 2.1.2. std::suspend_never final_suspend() noexcept 
 会在协程函数结束或者抛出异常后执行，可以通过该接口来清理资源。
 也可以返回一个 *等待体* 来使当前协程挂起。 

### 2.1.3. return_void()
与 *return_value* 定义互斥
定义返回值，这样协程内部就可以通过 *co_return* 来推出协程体。(即co_return 调用的就是_Ret::promise_type::return_void())

### 2.1.4. return_value(int value)
与 *return_void* 定义互斥
定义返回值，这样协程内部就可以通过*co_return value*来推出协程体。(即co_return 调用的就是_Ret::promise_type::return_value))


## 2.2. 关键字

### 2.2.1. co_await 
 表示当前函数(协程)被挂起，将调度权交给调用该协程函数的函数。
协程会在一开始执行的时候就使用`operator new` 来开启一块内存来存放这些信息这被称为 **协程的状态(coroutine state)**。
当协程挂起时，编译器会将参数移动或复制一份(取决于自身的复制构造和移动构造的定义)保存到协程的状态中。

该表达式的操作对象被称为 **等待体(awaiter)**,等待体需要实现以下三个函数来共协程在挂起和恢复时分别调用。 参见[[#1. 例程 | 例程]] 中的Awaiter实现
标准库中也已经实现了[struct suspend_never](https://en.cppreference.com/w/cpp/coroutine/suspend_never) 和[struct suspend_always](https://en.cppreference.com/w/cpp/coroutine/suspend_always)两个等待体

#### 2.2.1.1. await_ready
``` cpp
bool await_ready()
```

用于判断协程是否已经具备继续运行的条件。
- 返回 true: 总是不挂起
- 返回 flase: 总是挂起
因此可以通过一个可变变量来返回真正的状态。

#### 2.2.1.2. await_suspend
``` cpp
??? await_suspend(std::coroutine_handle<> coroutine_handle) 
```
- coroutine_handle: 协程的handle，用来表示当前协程，用于在适当的时机来调用resume来恢复执行当前协程
- 返回值 没有固定的类型
	- 返回 void 或者 true： 表示当前协程挂起后，将执行权还给当初调用或者恢复当前协程的函数
	- 返回false，恢复当前协程。 （注意也会存在挂起过程，只不过又立即恢复）。
	- 返回其他协程的coroutine_handle对象，这时候返回的coroutine_handle对应的协程被恢复执行
	- 抛出异常，此时当前协程和恢复执行，并在当前协程中抛出异常。

#### 2.2.1.3. await_resume:

``` cpp
??? await_resume();
```

协程恢复执行之后，等代替的await_resume函数被调用。（包括await_realy 返回 true的场景也会调用）
返回值将作为[[#2.2.1. co_await | co_await]] 表达式的返回值
	

### 2.2.2. co_yeild
 co_await的马甲，和co_await一样，同时会传递一个值

### 2.2.3. coroutine_handle.resume()
恢复指定协程的调度

### 2.2.4. co_return
协程退出
