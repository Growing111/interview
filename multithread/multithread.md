<h1>多线程</h1>


### 内存模型的三大特性
+ 原子性： Java 内存模型保证了 read、load、use、assign、store、write、lock 和 unlock 操作具有原子性，例如对一个 int 类型的变量执行 assign 赋值操作，这个操作就是原子性的
+ 可见性：可见性指当一个线程修改了共享变量的值，其它线程能够立即得知这个修改。Java 内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值来实现可见性的。
   主要有三种实现可见性的方式：
     + volatile (可以保证可见性，并不保证原子性)
     + synchronized，对一个变量执行 unlock 操作之前，必须把变量值同步回主内存。
     + final，被 final 关键字修饰的字段在构造器中一旦初始化完成，并且没有发生 this 逃逸（其它线程通过 this 引用访问到初始化了一半的对象），那么其它线程就能看见 final 字段的值。
+ 有序性：
   在本线程内观察，所有操作都是有序的。在一个线程观察另一个线程，所有操作都是无序的，无序是因为发生了指令重排序。
   实现有序行的方式：
   + volatile 关键字通过添加内存屏障的方式来禁止指令重排，即重排序时不能把后面的指令放到内存屏障之前、
   + 也可以通过 synchronized 来保证有序性，它保证每个时刻只有一个线程执行同步代码，相当于是让线程顺序执行同步代码

### 线程的五种基本状态
+ 1.新建状态（New）：当线程对象对创建后，即进入了新建状态，如：Thread t = new MyThread(); 
+ 2.就绪状态（Runnable）：当调用线程对象的start()方法，线程即进入就绪状态。处于就绪状态的线程，只是说明此线程已经做好了准备，随时等待CPU调度执行，并不是说执行了t.start()此线程立即就会执行；
+ 3.运行状态（Running）：当CPU开始调度处于就绪状态的线程时，此时线程才得以真正执行，即进入到运行状态。注：就 绪状态是进入到运行状态的唯一入口，也就是说，线程要想进入运行状态执行，首先必须处于就绪状态中； 
+ 4.阻塞状态（Blocked）：处于运行状态中的线程由于某种原因，暂时放弃对CPU的使用权，停止执行，此时进入阻塞状态，直到其进入到就绪状态，才有机会再次被CPU调用以进入到运行状态。根据阻塞产生的原因不同，阻塞状态又可以分为三种：
  +  等待阻塞：运行状态中的线程执行wait()方法，使本线程进入到等待阻塞状态；
  +  同步阻塞 — 线程在获取synchronized同步锁失败(因为锁被其它线程所占用)，它会进入同步阻塞状态；
  +  其他阻塞 — 通过调用线程的sleep()或join()或发出了I/O请求时，线程会进入到阻塞状态。当sleep()状态超时. join()等待线程终止或者超时. 或者I/O处理完毕时，线程重新转入就绪状态。
+ 5.死亡状态（Dead）：线程执行完了或者因异常退出了run()方法，该线程结束生命周期。

### 什么是线程死锁，死锁的四个必要条件
> 死锁： 多个线程同时被阻塞，它们中的一个或者全部都在等待某个资源被释放。由于线程被无限期地阻塞，因此程序不可能正常终止

死锁必须具备以下四个条件：
+ 互斥条件：该资源任意一个时刻只由一个线程占用
+ 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
+ 不剥夺条件:线程已获得的资源在末使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源
+ 循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。

### 线程的三种创建方式
常说的是有三种创建线程的方式,但是一般都直接使用：
+ 1.实现Runnable 接口
```
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        // ...
    }
}

public static void main(String[] args) {
    MyRunnable instance = new MyRunnable();
    Thread thread = new Thread(instance);
    thread.start();
}
```
+ 2.实现 Callable 接口
```
public class MyCallable implements Callable<Integer> {
    public Integer call() {
        return 123;
    }
}

public static void main(String[] args) throws ExecutionException, InterruptedException {
    MyCallable mc = new MyCallable();
    FutureTask<Integer> ft = new FutureTask<>(mc);
    Thread thread = new Thread(ft);
    thread.start();
    System.out.println(ft.get());
}
```
+ 3.继承Thread类
```
public class MyThread extends Thread {
    public void run() {
        // ...
    }
}

public static void main(String[] args) {
    MyThread mt = new MyThread();
    mt.start();
}
```
### java线程池
> 说明：使用线程就要注意cpu和内存的使用和释放，一般推荐是用线程池的方式来使用线程

线程池核心类-ThreadPoolExecutor,ThreadPoolExecutor含有如下七个参数
+ corePoolSize：核心线程数量
+ maximumPoolSize：线程最大线程数
+ workQueue：阻塞队列
   + 直接切换（SynchronusQueue）
   + 无界队列（LinkedBlockingQueue）能够创建的最大线程数为corePoolSize,这时maximumPoolSize就不会起作用了。当线程池中所有的核心线程都是运行状态的时候，新的任务提交就会放入等待队列中
   + 有界队列（ArrayBlockingQueue）最大maximumPoolSize，能够降低资源消耗，但是这种方式使得线程池对线程调度变的更困难。因为线程池与队列容量都是有限的。所以想让线程池的吞吐率和处理任务达到一个合理的范围，又想使我们的线程调度相对简单，并且还尽可能降低资源的消耗，我们就需要合理的限制这两个数量
+ keepAliveTime：线程没有任务执行时最多保持多久时间终止（当线程中的线程数量大于corePoolSize的时候，如果这时没有新的任务提交核心线程外的线程不会立即销毁，而是等待，直到超过keepAliveTime）
+ unit：keepAliveTime的时间单位
+ threadFactory：线程工厂，用来创建线程，有一个默认的工场来创建线程，这样新创建出来的线程有相同的优先级，是非守护线程、设置好了名称）
+ rejectHandler：当拒绝处理任务时(阻塞队列满)的策略（AbortPolicy默认策略直接抛出异常、CallerRunsPolicy用调用者所在的线程执行任务、DiscardOldestPolicy丢弃队列中最靠前的任务并执行当前任务、DiscardPolicy直接丢弃当前任务）

当调用线程池execute() 方法添加一个任务时，线程池会做如下判断：
+ 如果有空闲线程，则直接执行该任务；
+ 如果没有空闲线程，且当前运行的线程数少于corePoolSize，则创建新的线程执行该任务；
+ 如果没有空闲线程，且当前的线程数等于corePoolSize，同时阻塞队列未满，则将任务入队列，而不添加新的线程；
+ 如果没有空闲线程，且阻塞队列已满，同时池中的线程数小于maximumPoolSize ，则创建新的线程执行任务；
+ 如果没有空闲线程，且阻塞队列已满，同时池中的线程数等于maximumPoolSize ，则根据构造函数中的 handler 指定的策略来拒绝新的任务。

##### Executors类创建了四种线程池。但是阿里代码规范手册明确不推荐使用
+ Executors.newCachedThreadPool
  + newCachedThreadPool是一个根据需要创建新线程的线程池，当一个任务提交时，corePoolSize为0不创建核心线程，SynchronousQueue是一个不存储元素的队列，可以理解为队里永远是满的，因此最终会创建非核心线程来执行任务。
    对于非核心线程空闲60s时将被回收。因为Integer.MAX_VALUE非常大，可以认为是可以无限创建线程的
+ Executors.newSingleThreadExecutor
  + newSingleThreadExecutor是单线程线程池，只有一个核心线程，用唯一的一个共用线程执行任务，保证所有任务按指定顺序执行（FIFO、优先级…）
+ Executors.newFixedThreadPool
  + 定长线程池，核心线程数和最大线程数由用户传入，可以设置线程的最大并发数，超出在队列等待
+ Executors.newScheduledThreadPool
  + 定长线程池，核心线程数由用户传入，支持定时和周期任务执行


### 线程之间的协作
+ join
+ wait() notify() notifyAll()
+ await() signal() signalAll()

### CAS
+ 乐观锁需要操作和冲突检测这两个步骤具备原子性，这里就不能再使用互斥同步来保证了，只能靠硬件来完成。硬件支持的原子性操作最典型的是：比较并交换（Compare-and-Swap，CAS）。CAS 指令需要有 3 个操作数，分别是内存地址 V、旧的预期值 A 和新值 B。当执行操作时，只有当 V 的值等于 A，才将 V 的值更新为 B

### synchronized

### volatile的原理和作用
存在问题说明：
实例化一个对象其实可以分为三个步骤：
1.分配内存空间。2.初始化对象。3.将内存空间的地址赋值给对应的引用。
但是由于操作系统可以对指令进行重排序打乱顺序。

+ 防止指令重排序
> 获取JIT（即时Java编译器，把字节码解释为机器语言发送给处理器）的汇编代码，发现volatile多加了lock addl指令，这个操作相当于一个内存屏障，使得lock指令后的指令不能重排序到内存屏障前的位置

存在问题说明：
可见性问题主要指一个线程修改了共享变量值，而另一个线程却看不到。引起可见性问题的主要原因是每个线程拥有自己的一个高速缓存区——线程工作内存。
+ 保证线程变量的可见性。MESI（缓存一致性协议）
利用缓存块（64个字节）对齐，当一个线程中的变量更换后，通知提前其他线程失效了，其他线程取变量的时候重新从内存中取值
![缓存一致性处理](src/main/resources/img-storage/mesi.png)

### ThreadLocal

### ReentrantLock

### AQS


### CyclicBarrier 和 CountDownLatch 的区别
+ CountDownLatch 简单的说就是一个线程等待，直到他所等待的其他线程都执行完成并
    且调用 countDown()方法发出通知后，当前线程才可以继续执行。
+ cyclicBarrier 是所有线程都进行等待，直到所有线程都准备好进入 await()方法之后，
    所有线程同时开始执行！
+ CountDownLatch 的计数器只能使用一次。而 CyclicBarrier 的计数器可以使用 reset()
    方法重置。所以 CyclicBarrier 能处理更为复杂的业务场景，比如如果计算发生错误，可以
    重置计数器，并让线程们重新执行一次。
+ CyclicBarrier 还提 供其他有 用的方法 ，比如 getNumberWaiting 方法 可以获得
    CyclicBarrier 阻塞的线程数量。isBroken 方法用来知道阻塞的线程是否被中断。如果被中断
    返回 true，否则返回 false。

### Semaphore
+ Semaphore 类似于操作系统中的信号量，可以控制对互斥资源的访问线程数。
Semaphore(信号量)可以指定多个线程同时访问某个资源。

<hr>