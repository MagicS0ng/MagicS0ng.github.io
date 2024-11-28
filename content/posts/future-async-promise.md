+++
date = '2024-11-27T21:11:39+08:00'
draft = false
title = 'Future Async Promise'
tags = ['future', 'async', 'promise', 'multi-thread', 'C++']
show_toc = true
ShowCodeCopyButtons=true
ShowReadingTime=true
ShowBreadCrumbs=true
description="C++中std::future, std::packaged_task, std::promise的使用" 
summary = "C++中std::future, std::packaged_task, std::promise的使用"
[cover]
   image = "https://img-s-msn-com.akamaized.net/tenant/amp/entityid/BB1msG0X.img"
+++

# std::async

```C++
#include <chrono>
#include <future>
#include <iostream>
#include <string>
#include <thread>
// async 获取异步返回值
std::string fetchDataFromDB(std::string query) {
  std::cout << "Subthread suspended is fetching data from DB..." << std::endl;
  std::this_thread::sleep_for(std::chrono::seconds(4));
  return "Data: " + query;
}
void use_async() {
  // 异步获取数据
  std::future<std::string> fetchedData =
      std::async(std::launch::async, fetchDataFromDB, "query");
  // 主线程正常进行
  for (int i = 0; i < 10; i++) {
    std::cout << "Main thread is doing something else x" << i + 1 << " time(s)."
              << std::endl;
  }
  // get() 是阻塞的， 只有数据返回后才会继续执行
  std::string data = fetchedData.get();
  std::cout << data << std::endl;
}
int main()
{
    use_async();
    return 0;
}
```

## async 的启动模式

1. `std::launch::async` 立即执行
2. `std::launch::defer` 调用`std::future::get`时才会执行
3. `std::launch::deferred|std::launch::async` 不同的环境执行结果不同

# std::future

1. `std::future::get` 调用时会阻塞，调用后`std::future`就会失效
2. `std::future::wait` 阻塞调用，如果任务完成就会返回，没有则继续等待，可以多次调用，但是一旦调用了`std::future::get` 因为对象`std::future`的失效，就会异常

```C++

std::string fetchDataFromDB(std::string query) {
  std::cout << "Subthread suspended is fetching data from DB..." << std::endl;
  std::this_thread::sleep_for(std::chrono::seconds(4));
  return "Data: " + query;
}
void use_async_future_wait() {
  // 异步获取数据
  std::future<std::string> fetchedData =
      std::async(std::launch::async, fetchDataFromDB, "query");
  // 主线程正常进行
  for (int i = 0; i < 10; i++) {
    std::cout << "Main thread is doing something else x" << i + 1 << " time(s)."
              << std::endl;
  }
  // get() 是阻塞的， 只有数据返回后才会继续执行
  std::cout << "first call `fetchedData.get()` " << std::endl;
  fetchedData.wait();
  //   std::string data = fetchedData.get();
  //   std::cout << data << std::endl;
  for (int i = 10; i < 20; i++) {
    std::cout << "Main thread is doing something else x" << i + 1 << " time(s)."
              << std::endl;
  }
  std::cout << "call `future::wait()` again " << std::endl;
  fetchedData.wait();
}
```

# std::packaged_task

`std::packaged_task`可以将异步任务包装成`std::future`，运行在另一个线程上，可以捕获任务的返回值或异常
使用：

1. 创建一个`std::packaged_task` 对象，包装一个任务
2. 调用`std::packaged_task::get_future` 获取一个`std::future`对象
3. 在另一个线程上调用`std::packaged_task`的`operator()`执行任务
4. 调用`std::future::get`获取返回值或异常

```C++
int my_task() {
  std::this_thread::sleep_for(std::chrono::seconds(4));
  std::cout << "Subthread suspended is running task..." << std::endl;
  return 42;
}
void use_packaged_task() {
  std::packaged_task<int()> task(my_task);
  for (int i = 0; i < 10; i++) {
    std::cout << "Main thread is doing something else x" << i + 1 << " time(s)."
              << std::endl;
  }
  std::cout << "Call `task.get_future()`" << std::endl;
  auto res = task.get_future();
  std::cout << "Get result from `my_task()`" << std::endl;
  // 需要开辟一个子线程处理`std::packaged_task` 必须使用`std::move` std::packaged_task只支持移动语义
  std::thread t(std::move(task)); 
  t.detach();
  std::cout << res.get();
}
```

# std::promise
`std::promise`也可以在线程中获取异步返回值，保存在`std::future`中，在另外一个线程中获取这个值或异常。  
 与`std::packaged_task`不同的是，`std::packaged_task`绑定的是一个函数，也就是只有等待函数执行完才能拿到结果，  
 而`std::promise` 可以在函数中的任意一步定义，拿到值之后可以直接在其他线程中得到值。  
 `std::promise`也是只支持移动操作。
```C++
void set_value(std::promise<int> prom)
{
    std::cout << "Subthread suspended is setting value..." << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(4));
    prom.set_value(42);
    std::cout << "promise has set value to 42" << std::endl;
}
void use_promise()
{
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();
    std::thread t(set_value, std::move(prom)); // 开辟的子线程会运行，但只有通过`std::future`对象才能获取到`std::promise`中的值
    std::cout << "Main thread is doing something else..." << std::endl;
    std::cout << "Waiting for subthread to set value "<< std::endl;
    std::cout << "get value for subthread: " << fut.get() << std::endl; 
    t.join();
}
// 使用promise 捕获异常
void promise_one_exception(std::promise<void> promise) {
  try {
    throw std::runtime_error("Oops, Error");
  } catch (...) {
    promise.set_exception(std::current_exception());
  }
}
void use_promise_exception() {
  std::promise<void> promise;
  std::future<void> future = promise.get_future();
  std::thread t(promise_one_exception, std::move(promise));
  std::cout << "Main thread is doing something else..." << std::endl;
  std::cout << "Waiting for subthread to set value " << std::endl;
  future.wait();
  try {
    future.get();
  } catch (std::exception &e) {
    std::cout << "Caught exception: " << e.what() << std::endl;
  }
  t.join();
}
// 搭配std::shared_future
void function(std::promise<int> &&prom) {
  std::this_thread::sleep_for(std::chrono::seconds(10));
  prom.set_value(10);
}
void threadfunction(std::shared_future<int> future) {
  try {
    int result = future.get();
    std::cout << "Result: " << result << std::endl;
  } catch (std::future_error &e) {
    std::cout << "future error: " << e.what() << std::endl;
  }
}
void use_shared_future() {
  std::promise<int> prom;
  std::shared_future<int> future = prom.get_future();
  std::thread t1(function, std::move(prom));
  std::thread t2(threadfunction,
                 future); // 不可以通过 std::move(future)的方式传递
  std::thread t3(threadfunction, future);
  t1.join(), t2.join(), t3.join();
}
```
# last but not least
C++线程池的一种实现
```C++
#include <condition_variable>
#include <cstddef>
#include <functional>
#include <future>
#include <iostream>
#include <memory>
#include <mutex>
#include <queue>
#include <thread>
#include <utility>
#include <vector>

class ThreadPool {
  using TASK = std::packaged_task<void()>;

private:
  static std::shared_ptr<ThreadPool> _instance;
  std::atomic<int> _thread_count;
  std::atomic<bool> _stop;
  std::mutex _mtx;
  std::condition_variable _cond;
  std::vector<std::thread> _workers;
  std::queue<TASK> _tasks;

private:
  void work_thread() {
    while (!this->_stop.load()) {
      TASK task;
      {
        std::unique_lock<std::mutex> lock(this->_mtx);
        this->_cond.wait(lock, [this] {
          return this->_stop.load() || !this->_tasks.empty();
        });
        if (this->_tasks.empty())
          return;
        task = std::move(this->_tasks.front());

        this->_tasks.pop();
      }
      this->_thread_count--;
      task();
      this->_thread_count++;
    }
  }
  void start() {
    for (int i = 0; i < _thread_count; ++i) {
      _workers.emplace_back(&ThreadPool::work_thread, this);
    }
  }
  void stop() {
    _stop.store(true);
    _cond.notify_all();

    for (auto &t : _workers) {
      if (t.joinable()) {
        std::cout << "Join thread with id = " << t.get_id() << std::endl;
        t.join();
      }
    }
  }
  ThreadPool(size_t thread_count = 4) : _stop(false) {
    if (thread_count <= 1)
      _thread_count = 1;
    else
      _thread_count = thread_count;
    start();
  }
  ThreadPool(const ThreadPool &other) = delete;
  ThreadPool &operator=(const ThreadPool &) = delete;

public:
  ~ThreadPool() { stop(); }
  static std::shared_ptr<ThreadPool> GetInstance() {
    std::once_flag s_flag;
    std::call_once(s_flag, [&]() {
      if (_instance == nullptr)
        _instance = std::shared_ptr<ThreadPool>(new ThreadPool(4));
    });
    return _instance;
  }
  template <class F, class... Args>
  auto commit(F &&f, Args &&...args) -> std::future<decltype(f(args...))> {
    using RETURN_TYPE = decltype(f(args...));
    if (_stop.load())
      return std::future<RETURN_TYPE>{};
    auto task = std::make_shared<std::packaged_task<RETURN_TYPE()>>(
        std::bind(std::forward<F>(f), std::forward<Args>(args)...));
    std::future<RETURN_TYPE> res = task->get_future();
    {
      std::unique_lock<std::mutex> lck(_mtx);
      _tasks.emplace([task]() { (*task)(); });
    }
    _cond.notify_one();
    return res;
  }
  auto available_threads() const { return _thread_count.load(); }
};
std::shared_ptr<ThreadPool> ThreadPool::_instance = nullptr;
int main() {
  auto res = ThreadPool::GetInstance()->commit(
      [](int a, int b) { return a + b; }, 10, 20);
  std::cout << res.get() << std::endl;
};

```
注意,在`GetInstance()`中，只能使用`std::shared_ptr<ThreadPool>(new ThreadPool);`的方式构造 因为`make_shared`无法访问私有构造函数。