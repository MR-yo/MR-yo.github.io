---
layout: post
---

# Objective-C block 

### 1.block 类型


> * _NSConcreteStackBlock:
> </br>只用到外部局部变量、成员属性变量，且没有强指针引用的block都是StackBlock。
StackBlock的生命周期由系统控制的，一旦返回之后，就被系统销毁了。
>
> * _NSConcreteMallocBlock:
> </br>有强指针引用或copy修饰的成员属性引用的block会被复制一份到堆中成为MallocBlock，没有强指针引用即销毁，生命周期由程序员控制
> 
> * _NSConcreteGlobalBlock:
> </br>没有用到外界变量或只用到全局变量、静态变量的block为_NSConcreteGlobalBlock，生命周期从创建到应用程序结束



### 2.block 内部改变外部的值
C 语言中的变量有
* 自动变量
* 函数参数
* 静态变量
* 静态全局变量
* 全局变量

block 捕获外部变量时

* 自动变量是以值传递方式传递到Block的构造函数里面去的
* 静态全局变量，全局变量由于作用域的原因，于是可以直接在Block里面被改变。他们也都存储在全局区。
* 静态变量传递给Block是内存地址值，所以能在Block里面直接改变值

__block 的原理
* 改变变量的存储方式
