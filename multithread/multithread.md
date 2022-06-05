<h2>多线程</h2>

<h3>CyclicBarrier 和 CountDownLatch 的区别</h3>
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
    
<hr>