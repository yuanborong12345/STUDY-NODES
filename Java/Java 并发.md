## Java 创建线程的方式

### Thread 类

```java
// 每次创建都会创建一个新的Thread对象，线程与任务耦合
class ThreadPractice extends Thread {
    @Override
    public void run() {
        // run()方法无返回值、无法抛出检查异常
        try {
            System.out.println("继承Thread类执行任务：" + Thread.currentThread().getName());
        } catch (InterruptedException e) {
            // 只能捕获异常，无法向上抛出
            Thread.currentThread().interrupt(); // 恢复中断状态（最佳实践）
            System.out.println("线程被中断：" + e.getMessage());
        }
    }

    // 测试调用
    public static void main(String[] args) {
        ThreadPractice thread = new ThreadPractice();
        thread.start(); // 必须调用start()启动线程，而非直接调用run()
    }
}
```

### Runnable 接口

```java
// 避免单继承机制，任务与线程解耦，但无法获取线程执行结果
class RunnablePractice implements Runnable {
    @Override
    public void run() {
        // run()方法无返回值、无法抛出检查异常
        try {
            Thread.sleep(500);
            System.out.println("实现Runnable接口执行任务：" + Thread.currentThread().getName());
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            System.out.println("任务被中断：" + e.getMessage());
        }
    }

    // 测试调用（两种方式：手动Thread执行 / 线程池执行）
    public static void main(String[] args) {
        // 方式1：手动创建Thread执行（不推荐，仅演示）
        Runnable task = new RunnablePractice();
        new Thread(task, "Runnable-Thread-1").start();

        // 方式2：线程池执行（最佳实践）
        ExecutorService executor = Executors.newFixedThreadPool(2);
        executor.submit(task);
        executor.shutdown(); // 必须关闭线程池，避免资源泄漏
    }
}
```

### Callable 接口

与Runnable接口的区别在于可抛出受检异常，并且可用Future获取返回结果；Runnable无返回结果且异常只能在方法内部处理。

```java
import java.util.concurrent.*;

// 支持返回结果、抛出检查异常，需配合线程池执行
class CallablePractice implements Callable<Integer> {
    // 泛型指定返回值类型，call()可抛出检查异常
    @Override
    public Integer call() throws Exception {
        System.out.println("实现Callable接口执行任务：" + Thread.currentThread().getName());
        Thread.sleep(500);
        // 模拟业务计算（返回结果）
        int result = 100 + (int) (Math.random() * 100);
        // 可主动抛出检查异常（无需try-catch）
        if (result > 150) {
            throw new Exception("计算结果超出阈值：" + result);
        }
        return result;
    }

    // 测试调用（必须通过线程池执行）
    public static void main(String[] args) {
        Callable<Integer> task = new CallablePractice();
        // 创建线程池（核心最佳实践）
        ExecutorService executor = Executors.newSingleThreadExecutor();
        
        // 提交任务，返回Future对象（用于获取结果/取消任务）
        Future<Integer> future = executor.submit(task);

        try {
            // 获取结果：阻塞等待，建议设置超时时间避免死等
            Integer result = future.get(2, TimeUnit.SECONDS);
            System.out.println("Callable任务返回结果：" + result);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            System.out.println("获取结果时线程被中断");
        } catch (ExecutionException e) {
            // 捕获call()方法抛出的异常
            System.out.println("任务执行异常：" + e.getCause().getMessage());
        } catch (TimeoutException e) {
            future.cancel(true); // 超时取消任务
            System.out.println("获取结果超时，任务已取消");
        } finally {
            executor.shutdown(); // 关闭线程池
        }
    }
}
```

### start() 和 run() 的区别？

start () 是开新线程异步执行，真正启动了一个线程，进入就绪状态等待CPU去调度，底层是调用native方法；run () 是普通方法同步执行，直接调用是在主线程执行。

## Java 线程的状态

Java 线程有6种状态

- NEW：新建状态，线程对象已经创建但未调用start()
- RUNNABLE：可运行状态，正在运行 /  等待CPU调度 
- BLOCKED：阻塞状态，线程在等待获取某个对象的锁
- WAITING：等待状态，线程无限期等待其他线程的通知
- TIME_WAITING：限时等待状态，线程等待一定时间自动唤醒
- TERMINATED：终止状态，线程执行完毕或者抛出异常结束

## 并行和并发

- 并发（逻辑上的同时处理）

  系统同时处理多个任务的能力，但不一定是“同时”运行，通过线程切换、时间轮片等方式，让多个任务交替执行

- 并行（物理意义上的同时执行）

  真正意义上多个任务同时执行，让任务在不同的处理器核心上执行

## 线程池

本质上是一个线程资源管理器。核心问题是线程的生命周期问题管理问题。传统的线程每个任务都new Thread，而线程池通过预创建并复用一定数量的线程来执行提交的任务，避免频繁创建销毁线程的开销（CPU 和 内存)。

## 线程池的创建

### 不使用Excutors工具类创建线程池的原因

Excutors 工具类创建的线程池参数都是预设的，不够灵活，且存在缺陷

- **newSingleThreadPool / newFIxedThreadPool**

  使用 `LinkedBlockingQueue` 作为任务队列，且队列容量为 `Integer.MAX_VALUE`

  任务无限堆积，导致OOM内存溢出

- **newCachedThreadPool**

  核心线程数为 0，最大线程数为 `Integer.MAX_VALUE`；

  使用 `SynchronousQueue`（无容量队列），空闲线程存活时间仅 60 秒

  频繁创建线程大量的开销

- newScheduledThreadPool

  核心线程数可指定，但最大线程数为 `Integer.MAX_VALUE`，队列使用 `DelayedWorkQueue`（无界）。

## 线程池的状态

线程池内部维护了五种状态

![image-20260311211051671](.\images\image-20260311211051671.png)

RUNNING , SHUTDOWN,  STOP ,TIDYING , TERMINATED 

## 线程池的关闭

### shutDown() 和 shutDownNow() 

shutDown() 是优雅关闭，会等待所有正在执行的任务执行完成后再关闭

shutDown() 是强制性关闭，通过Interrupt() 机制立刻停止所有正在执行的任务，清空队列并返回未执行的任务列表

![image-20260311210249771](.\images\image-20260311210249771.png)

![image-20260311210307318](.\images\image-20260311210307318.png)

```java
pool.shutDown();
if(!pool.awaitTermination(60,TimeUnit.SECONDS)){
    List<Runnable> task = pool.shutDownNow();
    if(!pool.awaitTermination(60,TimeUnit.SECONDS)){
        sout("线程池无法正常关闭");
    }
}
```

## ThreadPoolExcutor 

### 七大核心参数

**corePoolSize** **（核心线程数）**，这些核心线程即使空闲也不会被回收，保证系统的响应能力。

当核心线程忙碌的时候，**workQueue（工作队列）**作为缓冲机制开始发挥作用，任务进入队列等待；

当工作队列已满，线程池才会创建非核心线程，直到达到**maximumPoolSize（最大线程数）**;

**keepActiveTime（线程空闲存活时间）、** **TimeUnit（时间单位）**

**threadFactory(线程创建工厂)**负责创建新线程

**rejectedExecutionHander （拒绝策略）**处理队列满且达到最大线程数时的拒绝策略

### 核心线程数

首先分析任务特性： CPU密集型 还是  I/O密集型

CPU 密集型通常为 CPU核心数量 + 1，因为更多的线程会导致频繁的上下文切换导致的线程开销，+1的目的是当某个线程因偶尔的 I/O 操作（如日志打印）或系统调用被阻塞时，这个 “备用线程” 能立即利用空闲的 CPU 核心，避免 CPU 空转。**用 1 个备用线程填补 CPU 核心的短暂空闲**，避免因线程偶尔阻塞导致 CPU 空转；

IO 密集型为 CPU核心数量 * (2 - 3)，具体还得是压测

### 工作队列选型

| 队列类型                | 核心特性                                                     | 入队 / 出队性能    | 适用场景                                                     | 注意事项                                                     |
| ----------------------- | ------------------------------------------------------------ | ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `ArrayBlockingQueue`    | 有界、基于数组、FIFO（先进先出）、创建时必须指定容量         | 高（数组结构）     | 绝大多数生产场景（CPU 密集 / IO 密集），尤其是需要严格控制任务堆积的场景 | 1. 必须指定容量（如 1000）；2. 公平锁（可选）性能略低，默认非公平 |
| `LinkedBlockingQueue`   | 可选有界 / 无界（默认无界 `Integer.MAX_VALUE`）、基于链表、FIFO | 中（链表节点创建） | 仅适合**低并发、任务量可控**的场景（需手动指定容量）         | 严禁使用默认无界模式（易 OOM）；高并发下节点创建有额外开销   |
| `SynchronousQueue`      | 无容量（零队列）、任务直接传递给线程，无缓冲                 | 极高（无存储）     | 高并发、任务处理速度极快的场景（如 `newCachedThreadPool` 原生队列） | 需配合较大的最大线程数；无缓冲，线程数不足时立即触发拒绝策略 |
| `PriorityBlockingQueue` | 无界（可自定义有界）、按优先级排序（需实现 `Comparable`）、基于堆结构 | 中（堆排序）       | 任务有优先级区分的场景（如核心任务优先执行）                 | 无界易 OOM；排序有性能开销；低优先级任务可能长期饥饿         |
| `DelayedWorkQueue`      | 无界、按延迟时间排序、仅支持 `ScheduledFutureTask`           | 中（延迟排序）     | 定时 / 延迟任务（如 `ScheduledThreadPoolExecutor` 原生队列） | 仅适配定时线程池；无界易 OOM                                 |

原则：

1. 必须使用有界队列，避免OOM问题

2. 队列容量看业务压测决定

3. 选型

   普通 FIFO 任务 → ArrayBlockingQueue（首选）/ LinkedBlockingQueue（次选）；

   高并发短任务 → SynchronousQueue；

   优先级任务 → PriorityBlockingQueue（自定义有界）；

   定时 / 延迟任务 → DelayedWorkQueue（仅适配 ScheduledThreadPoolExecutor）

### 拒绝策略

| 拒绝策略类名          | 核心行为                                                     | 适用场景                                                     |
| --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `AbortPolicy`（默认） | 直接抛出`RejectedExecutionException`异常，终止任务提交       | 核心业务场景（需感知任务提交失败，及时告警 / 处理）          |
| `CallerRunsPolicy`    | 由**提交任务的线程**（如主线程）直接执行该任务               | 非核心业务、低并发场景（降低并发压力，避免任务丢失）         |
| `DiscardPolicy`       | 静默丢弃新任务，无任何异常、无任何提示                       | 非核心、可丢失的任务（如日志收集、非实时统计）               |
| `DiscardOldestPolicy` | 丢弃任务队列中**最旧的任务**（队列头部），尝试重新提交当前任务 | 任务有先后顺序、允许丢弃旧任务（如消息消费，新任务比旧任务更重要） |

## CountDownLatch

核心作用是**让一组线程等待另一组线程完成全部任务后，再继续执行**（类似 “倒计时门闩”：倒计时归 0，门闩打开）。它的核心特点是**一次性使用**（倒计时归 0 后，无法重置) 

```java
package com.yuan.ThreadTest;

import java.util.concurrent.CountDownLatch;

public class StartupInitDemo {
    private static final int INIT_COMPONENT_COUNT = 3;
    private static final CountDownLatch latch = new CountDownLatch(INIT_COMPONENT_COUNT);
    
    // 初始化数据库连接池
    private static void initDatabase() {
        new Thread(() -> {
            try {
                System.out.println("开始初始化数据库连接池...");
                Thread.sleep(1500); // 模拟初始化耗时
                System.out.println("数据库连接池初始化完成");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                latch.countDown(); // 倒计时-1
            }
        }, "数据库初始化线程").start();
    }

    // 初始化Redis客户端
    private static void initRedis() {
        new Thread(() -> {
            try {
                System.out.println("开始初始化Redis客户端...");
                Thread.sleep(1000); // 模拟初始化耗时
                System.out.println("Redis客户端初始化完成");
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                latch.countDown(); // 倒计时-1
            }
        }, "Redis初始化线程").start();
    }

    public static void main(String[] args) throws InterruptedException {
        // 启动所有初始化线程
        initDatabase();
        initRedis();

        // 主线程等待所有初始化完成
        System.out.println("主线程等待所有组件初始化...");
        latch.await(); // 阻塞，直到倒计时归0

        // 所有组件初始化完成，服务启动
        System.out.println("===== 所有组件初始化完成，服务启动成功 =====");
    }
}

```

最典型的应用是

**程序启动初始化(等待其他加载完成，主线程再去执行)**

**多任务并行汇总（主任务切片为多个子任务，最后汇总）**

**高并发测试（阻塞请求，统一触发请求）**





## 线程优先级

线程优先级是线程调度的一个参考因素，取值1-10，默认为5；使用setPriority() 设置线程优先级。**优先级只是调度器的一个建议，并非强制性的规则**

## 多线程通信协作 + 同步互斥实现

| 概念     | 核心目标                                                  | 场景举例                     |
| -------- | --------------------------------------------------------- | ---------------------------- |
| 互斥     | 保证同一时间只有一个线程访问共享资源（避免竞争冲突）      | 多线程抢票、修改同一个计数器 |
| 同步     | 让线程按预定的顺序执行（比如 A 线程执行完，B 线程再执行） | 生产者生产完，消费者再消费   |
| 通信协作 | 线程间传递状态 / 数据，触发对方执行 / 等待                | 生产者通知消费者有新数据     |

### 基于wait/notify的生产者消费者

```java
package com.yuan.ThreadTest;

import java.util.LinkedList;
import java.util.Queue;

// 基于wait/notify的生产者消费者
public class WaitNotifyDemo {
    // 共享队列（仓库）
    private static final Queue<Integer> queue = new LinkedList<>();
    private static final int MAX_CAPACITY = 5; // 仓库最大容量

    // 生产者：生产数据，仓库满则等待
    static class Producer implements Runnable {
        @Override
        public void run() {
            synchronized (queue) {
                while (true) { // 用while而非if，防止虚假唤醒
                    if (queue.size() == MAX_CAPACITY) {
                        try {
                            System.out.println("仓库满，生产者等待...");
                            queue.wait(); // 释放锁，进入等待
                        } catch (InterruptedException e) {
                            Thread.currentThread().interrupt();
                        }
                    }
                    // 生产数据
                    int data = (int) (Math.random() * 100);
                    queue.offer(data);
                    System.out.println(Thread.currentThread().getName() + " 生产：" + data + "，仓库大小：" + queue.size());
                    queue.notifyAll(); // 唤醒消费者
                    try {
                        Thread.sleep(500); // 模拟生产耗时
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                }
            }
        }
    }

    // 消费者：消费数据，仓库空则等待
    static class Consumer implements Runnable {
        @Override
        public void run() {
            synchronized (queue) {
                while (true) {
                    if (queue.isEmpty()) {
                        try {
                            System.out.println("仓库空，消费者等待...");
                            queue.wait(); // 释放锁，进入等待
                        } catch (InterruptedException e) {
                            Thread.currentThread().interrupt();
                        }
                    }
                    // 消费数据
                    int data = queue.poll();
                    System.out.println(Thread.currentThread().getName() + " 消费：" + data + "，仓库大小：" + queue.size());
                    queue.notifyAll(); // 唤醒生产者
                    try {
                        Thread.sleep(800); // 模拟消费耗时
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                }
            }
        }
    }

    public static void main(String[] args) {
        new Thread(new Producer(), "生产者1").start();
        new Thread(new Consumer(), "消费者1").start();
    }
}
```

### 进阶：Condition 接口（基于 Lock 锁）

```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class ConditionDemo {
    private static final Queue<Integer> queue = new LinkedList<>();
    private static final int MAX_CAPACITY = 5;
    private static final ReentrantLock lock = new ReentrantLock();
    // 两个Condition：分别管理生产者和消费者的等待
    private static final Condition producerCond = lock.newCondition();
    private static final Condition consumerCond = lock.newCondition();

    static class Producer implements Runnable {
        @Override
        public void run() {
            while (true) {
                lock.lock(); // 获取锁
                try {
                    while (queue.size() == MAX_CAPACITY) {
                        System.out.println("仓库满，生产者等待...");
                        producerCond.await(); // 生产者等待
                    }
                    int data = (int) (Math.random() * 100);
                    queue.offer(data);
                    System.out.println(Thread.currentThread().getName() + " 生产：" + data + "，仓库大小：" + queue.size());
                    consumerCond.signal(); // 只唤醒消费者
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    lock.unlock(); // 释放锁（必须在finally中）
                }
            }
        }
    }

    static class Consumer implements Runnable {
        @Override
        public void run() {
            while (true) {
                lock.lock();
                try {
                    while (queue.isEmpty()) {
                        System.out.println("仓库空，消费者等待...");
                        consumerCond.await(); // 消费者等待
                    }
                    int data = queue.poll();
                    System.out.println(Thread.currentThread().getName() + " 消费：" + data + "，仓库大小：" + queue.size());
                    producerCond.signal(); // 只唤醒生产者
                    Thread.sleep(800);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    lock.unlock();
                }
            }
        }
    }

    public static void main(String[] args) {
        new Thread(new Producer(), "生产者1").start();
        new Thread(new Consumer(), "消费者1").start();
    }
}
```



## 当线程进入 synchronized 方法 A 后，能否进入同一个对象的 synchronized 方法 B？

可以。synchronized 在Java中基于对象监视器实现，每个对象都有一个内置锁。当同一个线程再次请求已持有的锁时，只需增加重入次数，无需重新竞争锁；只有当线程退出所有同步方法 / 代码块，重入次数归 0 时，才会释放锁。

对于对象的`synchronized`方法来说：

- 每个对象有且仅有一把「对象锁」；
- 线程进入任意一个`synchronized`非静态方法，都会获取该对象锁；
- 只要线程没释放这把锁，就能无障碍进入该对象的其他`synchronized`非静态方法。

## 
