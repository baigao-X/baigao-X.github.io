# 01 . Gmock 遇到的问题与解决方案<no value>

## 1.  对于异步回调的测试
可以在mock中改为同步的方法，用于测试回调接口的正常调用

## 2. 无法mock static 类型的函数
google是这么回答的： 如果你需要mock一个静态函数，说明你的程序模块过于紧耦合(灵活性不够、重用性不够、可测试性不够)。 
可采用的方法是定一个小接口，然后通过这个接口来调用static函数

## 3. 无法mock非虚函数
### 方案一： 模板类 
[官方示例 ](https://github.com/google/googletest/blob/master/googlemock/docs/CookBook.md)
- 缺点： 需要改造生产代码

### 方案二： 
重新定义一个mock类B,该类不继承被测类，但是在mock类B中，需要实现和A中同样的函数接口。在测试代码中通过mock类B对象，来达到测试目的。
	-缺点： 需要维护一套专门的测试代码。
### 方案三： HOOK