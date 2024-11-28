+++
date = '2024-11-26T20:52:32+08:00'
draft = false
title = 'Cpp Thread Basic'
tags = ['C++', 'multi-thread']
description = 'C++多线程的基础概念'
summary = 'C++多线程的基础概念'
+++
# 多线程
多线程享有一个进程的地址空间，线程切换比进程切换要简单，而且线程通信也比进程通信要简单。  
C++中使用多线程可以通过`std::thread`类实现。`std::thread`位于头文件`<thread>`中。
具体可以通过以下方式实现一个线程
1. 绑定普通函数
2. 绑定lambda表达式
3. 绑定函数对象
4. 函数指针
5. 仿函数
6. `std::bind`
初始化一个线程后，线程就会开始执行，但是线程可能会在主线程结束之后还需要运行，因此需要使用`std::thread::join`方法来等待线程结束。  
也可以通过`std::thread::detach`在主线程后台运行，主线程不会等待子线程执行结束。
## 绑定普通函数
```C++
void print()
{
    std::cout << "Hello, World!" << std::endl;
}
void bindaThread()
{
    std::thread t1(print);
    t1.join();
}
```
## 绑定类成员函数
```C++
class Foo
{
    public:
    void print(){
        std::cout << "This is a class member function" << std::endl;
    }
}
void bindClassMember()
{
    Foo foo;
    std::thread t(&Foo::print, &foo);
    t.join();
}
```
## 绑定lambda
```C++
void bindLambda()
{
    std::thread t([](){
        std::cout << "Bind a lambda function" << std::endl;
    })
}
```
## 绑定仿函数
```C++
struct ClassLikeFunc
{
    void operator()()
    {
        std::cout << "Bind a class like function" << std::endl;
    }
}
void bindClassLikeFunc()
{
    std::thread t{ClassLikeFunc()};
    t.join();
}
```
如需要传参，初始化时`std::thread t(func,params)`即可。  

# 注意
1. `std::thread`不支持拷贝构造，只能通过移动构造。  因此一个线程对象可以通过`std::move()`的方式将所有权转移给其他线程
2. `std::thread`默认传参是以拷贝的方式，如果要传入引用，需要通过`std::ref()`