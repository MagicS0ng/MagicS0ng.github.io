+++
date = '2024-11-27T11:29:45+08:00'
draft = true
title = 'Cpp Thread Lock'
tags = ['C++', 'multi-thread', 'locks']
description = "C++多线程编程中的锁的使用方法"
summary = "C++多线程编程中的锁的使用方法"

+++

# mutex
`std::mutex`最基础的锁 支持
1. `std::mutex::lock`
2. `std::mutex::unlock`
# lock_guard
`std::lock_guard`可以接受一个`std::mutex`作为参数，自动加锁，超出作用域自动解锁。
但无法手动解锁
# unique_lock

`std::unique_lock` 传递一个`std::mutex`作为参数，自动加锁，超出作用域自动解锁。  
也可以手动解锁，延迟加锁。常与`std::condition_variable` 搭配使用

```C++

std::mutex mtx;
void useUnique_lock()
{
     std::unique_lock<std::mutex> lock(mtx);
     if(lock.owns_lock()) //判断是否拥有锁
     {
        // do something
     }
     else {

     }
     lock.unlock();   // 手动解锁
     if(lock.owns_lock()) //判断是否拥有锁
     {
        // do something
     }
     else {

     }
}
void deferlock()
{
    std::unique_lock<std::mutex> lock(mtx, std::defer_lock); // 此时并未加锁
}

```
# std::shared_lock
共享锁，多个线程可以同时访问，需要定义`std::shared_mutex`