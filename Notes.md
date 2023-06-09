# Note for Concurrency Thread

## 同步方式

下面是对C++中常用的6种线程间同步方式的优缺点和适合的场景的总结：

### 互斥锁（mutex）

优点：

- 实现简单，易于理解；
- 支持递归锁，即同一线程可以多次获得锁，避免死锁；
- 支持线程优先级，可以避免低优先级线程饥饿。

缺点：

- 可能会造成线程饥饿（starvation）问题，即某些线程由于竞争锁而无法访问共享资源；
- 如果锁的粒度过大，可能会影响程序的性能。

适用场景：

- 对共享资源访问的控制需要互斥；
- 适用于竞争不激烈的场景。

### 条件变量（condition_variable）

优点：

- 可以避免线程忙等待的问题，节约CPU资源；
- 支持多个线程等待同一条件变量，可以实现复杂的同步逻辑。

缺点：

- 使用条件变量需要结合互斥锁，实现起来比较复杂；
- 可能会造成死锁和优先级反转问题。

适用场景：

- 适用于多线程协作完成某个任务的场景；
- 当线程需要等待某个条件时，可以使用条件变量避免忙等待的问题。

### 原子操作（atomic）

优点：

- 简单易用，不需要加锁；
- 提供了多种操作，可以满足各种需求。

缺点：

- 不适用于复杂的同步场景；
- 可能会造成ABA问题（即在操作过程中，共享资源被修改多次，但最终结果与初始状态相同）。

适用场景：

- 适用于对共享资源进行简单的读写操作；
- 适用于不需要加锁的场景。

### 信号量（semaphore）

优点：

- 可以控制多个线程对共享资源的访问；
- 可以控制同时运行的线程数，避免系统资源被耗尽。

缺点：

- 如果使用不当，可能会造成死锁问题；
- 实现较为复杂，容易出错。

适用场景：

- 适用于需要限制同时访问某些资源的线程数的场景；
- 适用于需要等待某些事件发生后再执行的场景。

### 屏障（barrier）

优点：

- 可以保证多个线程在某个点上同步执行；
- 可以避免线程间的竞争和死锁问题。

缺点：

- 可能会影响程序的性能；
- 实现较为复杂，容易出错。

适用场景：

- 适用于需要多个线程协同完成某个任务的场景；
- 适用于需要等待某些事件发生后再执行的场景。

### 读写锁（read-write lock）

优点：

- 可以提高共享资源的读取效率；
- 支持多个线程并发读取共享资源。

缺点：

- 写操作需要独占锁，可能会降低写操作的效率；
- 实现较为复杂，容易出错。

适用场景：

- 适用于读操作远远大于写操作的场景；
- 适用于对共享资源进行频繁读取的场景。

## 测试和调试

### 设计并发代码时要考虑的地方

1. 哪些数据需要保护，防止并行访问？
2. 如何保护这些数据？
3. 在数据被保护的此刻，其他线程执行到代码何处？
4. 该线程持有的哪种同步量？
5. 其他线程持有哪些同步量？
6. 该线程中所有的操作有先后顺序吗？
7. 该线程载入的数据是否有效？是否被其他线程修改了？
8. 如果假设其他线程可能正在修改该数据，那么可能会导致什么样的后果？以及如何保证这样的事情永不发生？

### 测试并发队列，需要考虑的场景

1. 单线程测试队列的push()、pop()的基本功能正确；
2. 在空队列上，
   1. 多个线程调用push()；
   2. 多个线程调用pop()；
   3. 一个线程push()，同时另一个线程pop()；
   4. 多个线程push()，一个线程pop()；
   5. 多个线程pop()，一个线程push()；
   6. 多个线程push()，多个线程pop()；
3. 在满队列上，
   1. 同上空队列；
   2. 在特定的满队列上，多个线程pop()，但线程数多于队列大小。
4. 附加的因素考虑：
   1. 在每个场景中，多线程是什么意思（3、4、1024？）
   2. 系统是否有足够的处理核来为每个运行线程分配一个核？
   3. 测试程序需要在哪种结构的处理器上运行？
   4. 将怎样为测试的并行部分确定合适的时间顺序？

### 测试的样例代码

#### 测试一个线程调用pop()，一个线程调用push()，保证线程安全

见代码 test_concurrent_push_and_pop_on_empty_queue.cpp

