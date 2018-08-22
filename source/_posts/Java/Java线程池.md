---
title: Java 线程池
categories:
  - Java
tags:
  - ExecutorService
  - Executors
abbrlink: bc557e1a
date: 2018-08-17 16:47:17
---



## 线程池作用

- CPU资源隔离
- 减少上下文切换
- 减少线程创建/关闭的资源开销
- 更好并发控制
- 更好生命周期控制

## 设计时注意事项

- 任务混杂
- 任务依赖
- 饥饿死锁
- 慢操作



<!-- more -->



## 使用时注意事项

### 线程池参数

- 核心池大小（core pool size）
- 最大池的大小（maximum pool size）
- 存活时间（keep-alive time）
- 缓冲队列（work queue）

### 任务队列（BlockingQueue）

- 无限队列
  - LinkedBlockingQueue
    - `newSingleThreadExecutor`
    - `newFixedThreadPool`
- 有限队列 

  - ArrayBlockingQueue
  - LinkedBlockingQueue(int capacity)
  - 饱和策略
    - `setRejectedExecutionHandler`
    - `ThreadPoolExecutor.AbortPolicy`
      - 中止策略（默认）
      - 抛出`RejectedExecutionException`
      - 调用者捕获后，自行实现逻辑
  - ThreadPoolExecutor.CallerRunsPolicy 
    - 不丢弃任务
    - 不抛出异常
    - 把任务退回调用者线程执行（同步调用）
  - ThreadPoolExecutor.DiscardOldestPolicy 
    - 遗弃最旧的任务
    - 选择本应该接下来就要执行的任务
      - 会尝试再次提交
    - 如果使用优先级队列，则丢弃优先级最高的元素
    - *配合 SynchronousQueue 使用，可以实现任务提交并等待的效果*
  - ThreadPoolExecutor.DiscardPolicy 
    - 遗弃策略
    - 放弃这个任务
  - 可以结合`Semaphore`使用 
    - 限制任务注入率（injection rate）

  - FIFO
- 同步移交（synchronous handoff） 
  - 直接传递给其他线程
  - SynchronousQueue
    - `newCachedThreadPool`

### 线程工厂

- 设置异常处理 `UncaughtExceptionHandler`

- 设置线程名称
- 优先级（不建议）
- 守护线程（不建议）
- 增加额外的计数器
- 增加额外的统计信息
- `Executors.privilegedThreadFactory()`，使用安全策略创建线程 



### 执行策略

- 执行任务，不关心结果
  - void execute(Runnable command)
- 执行任务，需要对结果进行处理
  - Future submit(Callable task)
  - Future submit(Runnable task)
  - Future submit(Runnable task, T result)
- 执行一批任务，需要全部成功完成
  - List<Future> invokeAll(Collection<? extends Callable<T>> tasks)
  - List<Future> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit)
- 执行一批任务，只需要有一个成功完成即可
  - T List<Future> invokeAny(Collection<? extends Callable<T>> tasks)
  - T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit)
- 执行一批任务，需要逐个处理结果
  - CompletionService
    - ExecutorCompletionService