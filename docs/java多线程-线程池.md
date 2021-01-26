###ThreadPoolExecutor 的构造函数 (https://blog.csdn.net/xiaojin21cen/article/details/87359148)
corePoolSize ：线程池中核心线程数的最大值
maximumPoolSize ：线程池中能拥有最多线程数
workQueue：用于缓存任务的阻塞队列
keepAliveTime ：表示空闲线程的存活时间。
TimeUnit unit ：表示keepAliveTime的单位。
```
public ThreadPoolExecutor(int corePoolSize,
    int maximumPoolSize,
    long keepAliveTime,
    TimeUnit unit,
 BlockingQueue<Runnable> workQueue,
 ThreadFactory threadFactory,
RejectedExecutionHandler handler) 

```




### corePoolSize、maximumPoolSize、workQueue 三者关系

我们现在通过向线程池添加新的任务来说明着三者之间的关系。

（1）如果没有空闲的线程执行该任务且当前运行的线程数少于 corePoolSize ，则添加新的线程执行该任务。

（2）如果没有空闲的线程执行该任务且当前的线程数等于 corePoolSize ，同时阻塞队列(workQueue)未满，则将任务入队列，而不添加新的线程。

（3）如果没有空闲的线程执行该任务且阻塞队列已满同时池中的线程数小于maximumPoolSize ，则创建新的线程执行任务。

（4）如果没有空闲的线程执行该任务且阻塞队列已满同时池中的线程数等于maximumPoolSize ，则根据构造函数中的 handler 指定的策略来拒绝新的任务。

handler 拒绝策略

handler ：表示当 workQueue 已满，且池中的线程数达到 maximumPoolSize 时，线程池拒绝添加新任务时采取的策略。

为了解释 handler 的作用，我们在上述情况下做另一种假设。
假设线程池这个单位招满临时工，但新任务依然继续增加，线程池从上到下，从里到外真心忙的不可开交，
阻塞队列也满了，只好拒绝上级委派下来的任务。怎么拒绝是门艺术，handler 一般可以采取以下四种取值。
策略
 ```
ThreadPoolExecutor.AbortPolicy()	    抛出RejectedExecutionException异常
ThreadPoolExecutor.CallerRunsPolicy()	由向线程池提交任务的线程来执行该任务，当触发拒绝策略时，只要线程池没有关闭，就由提交任务的当前线程处理。
ThreadPoolExecutor.DiscardPolicy()	    抛弃当前的任务
ThreadPoolExecutor.DiscardOldestPolicy()	抛弃最旧的任务（最先提交而没有得到执行的任务)
```
CallerRunsPolicy、DiscardPolicy、DiscardOldestPolicy 处理器针对被拒绝的任务并不是一个很好的处理方式。

    CallerRunsPolicy 在非线程池以外直接调用任务的run方法，可能会造成线程安全上的问题；
    DiscardPolicy 默默的忽略掉被拒绝任务，也没有输出日志或者提示，开发人员不会知道线程池的处理过程出现了错误；
    DiscardOldestPolicy 中e.getQueue().poll()的方式好像是科学的，但是如果等待队列出现了容量问题，大多数情况下就是这个线程池的代码出现了BUG。

最科学的的还是 AbortPolicy 提供的处理方式：抛出异常，由开发人员进行处理。