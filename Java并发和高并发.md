# 线程的状态

## 新建（New）

创建后尚未启动。

## 可运行（Runnable）

可能正在运行，也可能正在等待 CPU 时间片。

包含了操作系统线程状态中的 Running 和 Ready。

## 阻塞（Blocked）

等待获取一个排它锁，如果其线程释放了锁就会结束此状态。

## 无限期等待（Waiting）

等待其它线程显式地唤醒，否则不会被分配 CPU 时间片。

| 进入方法                                   | 退出方法                             |
| ------------------------------------------ | ------------------------------------ |
| 没有设置 Timeout 参数的 Object.wait() 方法 | Object.notify() / Object.notifyAll() |
| 没有设置 Timeout 参数的 Thread.join() 方法 | 被调用的线程执行完毕                 |
| LockSupport.park() 方法                    | LockSupport.unpark(Thread)           |

## 限期等待（Timed Waiting）

无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。

**调用 Thread.sleep() 方法使线程进入限期等待状态时，常常用“使一个线程睡眠”进行描述。**

**调用 Object.wait() 方法使线程进入限期等待或者无限期等待时，常常用“挂起一个线程”进行描述。**

睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。

阻塞和等待的区别在于，阻塞是被动的，它是在等待获取一个排它锁。而等待是主动的，通过调用 Thread.sleep() 和 Object.wait() 等方法进入。

| 进入方法                                 | 退出方法                                        |
| ---------------------------------------- | ----------------------------------------------- |
| Thread.sleep() 方法                      | 时间结束                                        |
| 设置了 Timeout 参数的 Object.wait() 方法 | 时间结束 / Object.notify() / Object.notifyAll() |
| 设置了 Timeout 参数的 Thread.join() 方法 | 时间结束 / 被调用的线程执行完毕                 |
| LockSupport.parkNanos() 方法             | LockSupport.unpark(Thread)                      |
| LockSupport.parkUntil() 方法             | LockSupport.unpark(Thread)                      |

## 死亡（Terminated）

可以是线程结束任务之后自己结束，或者产生了异常而结束。

# 使用线程

有三种使用线程的方法：

- 实现 Runnable 接口；
- 实现 Callable 接口；
- 继承 Thread 类。

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以说任务是通过线程驱动从而执行的。

## 实现 Runnable 接口

需要实现 run() 方法。

通过 Thread 调用 start() 方法来启动线程。

```java
public class MyRunnable implements Runnable {
    public void run() {
        // ...
    }
}
```

```java
public static void main(String[] args) {
    MyRunnable instance = new MyRunnable();
    Thread thread = new Thread(instance);
    thread.start();
}
```

## 实现 Callable 接口

与 Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装。

```java
public class MyCallable implements Callable<Integer> {
    public Integer call() {
        return 123;
    }
}
```

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    MyCallable mc = new MyCallable();
    FutureTask<Integer> ft = new FutureTask<>(mc);
    Thread thread = new Thread(ft);
    thread.start();
    System.out.println(ft.get());
}
```

## 继承 Thread 类

同样也是需要实现 run() 方法，因为 Thread 类也实现了 Runable 接口。

当调用 start() 方法启动一个线程时，虚拟机会将该线程放入就绪队列中等待被调度，当一个线程被调度时会执行该线程的 run() 方法。

```java
public class MyThread extends Thread {
    public void run() {
        // ...
    }
}
```

```java
public static void main(String[] args) {
    MyThread mt = new MyThread();
    mt.start();
}
```

## 实现接口 VS 继承 Thread

实现接口会更好一些，因为：

- Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口；
- 类可能只要求可执行就行，继承整个 Thread 类开销过大。

# 基础线程机制

## Executor

Executor 管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

主要有三种 Executor：

- CachedThreadPool：一个任务创建一个线程；
- FixedThreadPool：所有任务只能使用固定大小的线程；
- SingleThreadExecutor：相当于大小为 1 的 FixedThreadPool。

```java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < 5; i++) {
        executorService.execute(new MyRunnable());
    }
    executorService.shutdown();
}
```

## Daemon

守护线程是程序运行时在后台提供服务的线程，不属于程序中不可或缺的部分。

当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程。

main() 属于非守护线程。

使用 setDaemon() 方法将一个线程设置为守护线程。

```java
public static void main(String[] args) {
    Thread thread = new Thread(new MyRunnable());
    thread.setDaemon(true);
}
```

## sleep()

Thread.sleep(millisec) 方法会休眠当前正在执行的线程，millisec 单位为毫秒。

sleep() 可能会抛出 InterruptedException，因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。线程中抛出的其它异常也同样需要在本地进行处理。

```java
public void run() {
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

## yield()

对静态方法 Thread.yield() 的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。

```java
public void run() {
    Thread.yield();
}
```

# 线程之间的协作

当多个线程可以一起工作去解决某个问题时，如果某些部分必须在其它部分之前完成，那么就需要对线程进行协调。

## join()

在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。

## wait()、notify()、notifyAll()

调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。

它们都属于 Object 的一部分，而不属于 Thread。

只能用在同步方法或者同步控制块中使用，否则会在运行时抛出 IllegalMonitorStateException。

使用 wait() 挂起期间，线程会释放锁。这是因为，如果没有释放锁，那么其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。

public class WaitNotifyExample {

```java
public synchronized void before() {
    System.out.println("before");
    notifyAll();
}

public synchronized void after() {
    try {
        wait();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("after");
}
```
**wait() 和 sleep() 的区别** 

- wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
- wait() 会释放锁，sleep() 不会。

## await()、signal()、signalAll()

java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。

相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。

使用 Lock 来获取一个 Condition 对象。

public class AwaitSignalExample {

```java
private Lock lock = new ReentrantLock();
private Condition condition = lock.newCondition();

public void before() {
    lock.lock();
    try {
        System.out.println("before");
        condition.signalAll();
    } finally {
        lock.unlock();
    }
}

public void after() {
    lock.lock();
    try {
        condition.await();
        System.out.println("after");
    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        lock.unlock();
    }
}
```
# 线程安全性

不管何种调度方式，不需要额外的同步或者协同，都能表现正确的行为

* 原子性：互斥访问
* 可见性：一个线程对主存的修改能及时被其他线程观察
* 有序性：尽管所有线程执行顺序改变，单个线程的执行顺序不会改变

## 原子性

### Atomic包

AtomicXXX方法使用了Unsafe类的CompareAndSet方法，就是常说的CAS，此方法将内存中的值读取，和预期的值比较，如果相等则允许更新，不相等则循环再取。

### 锁

* synchronized：依赖于JVM
* Lock：依赖特殊的CPU指令
  * ReentrantLock--java代码实现

#### synchronized

* 修饰代码块：大括号括起来的，作用于实例对象
* 修饰方法：整个方法，作用于该类的实例对象
* 修饰静态方法：整个静态方法，作用于类对象
* 修饰类：括号括起来的部分，作用于类对象

下面代码通过线程池模拟两个调用对象，同时对一份代码进行调用。结果说明，同步只对方法有用，不同实例对象之间互不影响

```Java
@Slf4j
public class SynchronizedExample {

    //修饰一个代码块
    public void test1(int j) {
        synchronized (this) {
            for (int i = 0; i < 10; i++) {
                log.info("test1-{}-{}", i, j);
            }
        }
    }
    //修饰一个方法
    public synchronized void test2() {
        for (int i = 0; i < 10; i++) {
            log.info("test2-{}", i);
        }
    }

    public static void main(String[] args) {
        SynchronizedExample example1 = new SynchronizedExample();
        SynchronizedExample example2 = new SynchronizedExample();

        ExecutorService executorService = Executors.newCachedThreadPool(); //声明一个线程池
        //调用两个线程池同时执行
        executorService.execute(()->{
            example1.test1(1);
        });
        executorService.execute(()->{
            example2.test1(2);
        });
    }
}
```

```Java
@Slf4j
public class SynchronizedExample2 {
    //如果子类调用test1的方法，是没有synchronized作用的，需要自己添加，因为这个不是属于方法声明的一部分。
    //修饰一个代码块
    public static void test1(int j) {
        synchronized (SynchronizedExample2.class) {
            for (int i = 0; i < 10; i++) {
                log.info("test1-{}-{}", i, j);
            }
        }
    }
    //修饰一个方法
    public static synchronized void test2(int j) {
        for (int i = 0; i < 10; i++) {
            log.info("test2-{}-{}", i, j);
        }
    }

    public static void main(String[] args) {
        SynchronizedExample2 example1 = new SynchronizedExample2();
        SynchronizedExample2 example2 = new SynchronizedExample2();

        ExecutorService executorService = Executors.newCachedThreadPool(); //声明一个线程池
        //调用两个线程池同时执行
        executorService.execute(()->{
            example1.test1(1);
        });
        executorService.execute(()->{
            example2.test1(2);
        });
    }
}
```

#### 对比

* synchronized：不可中断，适合竞争激烈，可读性好
* Lock：可中断锁，多样化同步，竞争激烈时维持常态
* Atomic：竞争激烈时能维持常态，比Lock性能更好。但是一次只能同步一个值

## 可见性

导致不可见的原因：

* 线程交叉执行
* 重排序结合线程交叉执行
* 共享变量更新后的值没有在工作内存与主存间及时更新

### synchronized

* 线程解锁前必须把共享变量的最新值刷新回到主存
* 线程加锁时，将清空工作内存中共享变量的值，重新从主存中读取最新值

### volatile

通过加入<a style="color:red;">内存屏障</a>和<a style="color:red;">禁止重排序</a>优化来实现

* 对volatile变量写操作时，会在写操作后加入一条store屏障命令，将本地内存的共享变量刷新回主存
* 对volatile变量读操作时，会在读操作前加入一条load屏障指令，从主存中读取共享变量

一般用于两个线程检查

```Java
volatile boolean inited = false；
//线程1
context = loadContext();
inited = true;

//线程2
while(!inited){
    sleep();
}
doSomething;
```

## 有序性

如果两个线程的操作顺序无法从happen-before规则推导出来，就不能保证有序性，虚拟机可以随意进行重排序

### happen-before规则

* 程序次序规则：一个线程内，按照书写顺序执行
* 锁定规则：一个unlock操作先行发生于后面同一个锁的lock操作
* volatile变量规则：对于一个变量的写操作先行发生于后面对这个变量的读操作
* 传递规则：如果操作A先行发生于B，B又先行于C，则A先于C
* 线程启动规则：Thread对象的start()方法先行与此线程的每一个动作
* 线程中断规则：对于线程interrupt()方法的调用 先行发生于 被中断线程的代码 检测到中断事件的发生
* 线程终结规则：线程中所有操作都先行发生于线程的终止检测
* 对象终结规则：一个对象的初始化完成先行发生于它的finalize()方法的开始

# 安全发布对象

* 发布对象：使一个对象能够被当前范围之外的代码所使用
* 对象逸出：一种错误发布。当对象还没有构造完就被其他线程可见

## 安全发布对象的四种方法

### 1.在静态初始化函数中初始化一个对象的引用

### 2.将对象的引用保存到volatile类型域或者AtomicRerence对象中

### 3.将对象的引用保存到某个正确的够着对象的final类型域中

### 4.将对象的引用保存到一个由锁保护的域中

# 不可变对象

* 不可变对象需要满足的条件
  * 创建后其状态不可修改
  * 对象所有域都是final类型
  * 对象是正确创建的（在创建之前，this引用没有逸出）

## final关键字：

* 修饰类：不能被继承-参考String类型
* 修饰方法：锁定不被继承修改。一个类的private会被隐式显示为final
* 修饰变量：基本类型：初始化之后不能修改；引用类型：初始化之后不能指向另外一个对象

```Java
private final static Integer a = 1;
    private final static String b = "2";
    private final static Map<Integer, Integer> map = Maps.newHashMap();
    static {
        map.put(1, 2);
        map.put(2, 3);
        map.put(4, 5);
    }

    public static void main(String[] args) {
        //a=2;
        //b="2";
        //map=newHashMap();
        map.put(1, 3);
        log.info("{}", map.get(1));//尽管map引用不可变，但是里面的值可以
    }
```



## Collections.unmodifiableXXX

对应的有Collection，List，Set，Map等，只要将这些传入，即可不能被修改。

```Java
static {
        map.put(1, 2);
        map.put(2, 3);
        map.put(4, 5);
        map = Collections.unmodifiableMap(map);
    }

    public static void main(String[] args) {
        map.put(1, 3);  //抛出异常，不能让值被更改
        log.info("{}", map.get(1));
    }
```



## Guava：ImmutableXXX

对应的有Collection，List，Set，Map等，都带有初始化方法

```Java
 private final static ImmutableList<Integer> list = ImmutableList.of(1, 2, 3);
    private final static ImmutableSet set = ImmutableSet.copyOf(list);
    private final static ImmutableMap<Integer, Integer> map = ImmutableMap.of(1, 2, 3, 4);
    private final static ImmutableMap<Integer, Integer> map2 = ImmutableMap.<Integer,Integer>builder()
            .put(1, 2)
            .put(3, 4)
            .build();
```



# 线程安全手段

将对象封闭到一个线程里

## 堆栈封闭

局部变量，无并发问题

## ThreadLocal线程封闭

特别好的封闭方法。内部使用Map，key是线程名称，value是需要封闭的对象。

# 线程不安全的类或写法

线程不安全------->线程安全

* StringBuilder -----> StringBuffer（内部由synchronized关键字修饰）
* SimpleDataFormat ----> JodaTime
* ArrayList , HashSet , HashMap等Collections
* 先检查再执行：`if(condition(a)){handle(a);}`

# 线程安全--同步容器

* ArrayList->Vector, Stack
* HashMap->HashTable(key,value不能为null)
* Collections.synchronizedXXX(List, set, Map)

编写代码时需要注意的：

1. 在使用同步容器的时候，也会出现线程不安全的情况（比如两个线程分别对Vector实行get和remove操作，有可能会出现get的时候数组越界，因为get的索引从vector的size获取，可能size被读取之后紧接着发生remove，导致数据不一致）。写的时候一定要注意
2. 在使用foreach和iterator的时候，不要做数据的更新删除操作，需要记录下来之后再处理。for循环可以。多线程下解决解决办法：使用同步关键字进行同步，同时还可以使用同步容器和其他相关的并发容器。

# 线程安全--并发集合J.U.C

在java.util.concurrent包下的类

* ArrayList ----> CopyOnWriteArrayList 

  思想：1. 读写分离；2.最终一致性；3.使用时另外开辟空间解决并发冲突

* HashSet, TreeSet -----> CopyOnWriteArraySet,  ConcurrentSkipListSet

  ConcurrentSkipListSet：支持自然排序的，基于Map集合，多线程线程安全，但是批量操作不能保证原子性，只能保证每一次操作时是原子性的。不允许null。

* HashMap, TreeMap  ---->  ConcurrentHashMap,  ConcurrentSkipListMap

  ConcurrentHashMap：不允许null。针对读操作做了大量优化。

  ConcurrentSkipListMap：内部使用跳表实现。key是有序的，支持更高的并发。

# 安全共享对象的策略--总结

* 线程限制：由线程独占并且只能被此线程修改
* 共享只读：在没有额外同步的情况下，可以被多线程访问，但不允许修改
* 线程安全对象：在内部通过同步机制保证线程安全，所以其他线程无需额外的同步就可以通过公共接口访问它。
* 被守护对象：只能通过获得特定的锁来访问。

# J.U.C--AQS（AbstractQueuedSynchronizer）

* 基于Node实现的FIFO队列，可以用来构建锁和其他同步控件的基础框架
* 利用一个int类型表示状态
* 使用方法是继承
* 子类通过继承并通过实现它的方法管理其状态的方法操纵状态
* 可以同时实现排它锁和共享锁模式（独占，共享）

## AQS同步组件

### CountDownLatch

使用计数器进行初始化，该计数器是原子性操作，同一时刻只有一个线程能执行操作。调用该类的`await()`方法的线程一直处于阻塞状态，直到其他线程调用`countDown()`使计数器的值为0时，所有调用`await()`方法 的线程会继续执行。这个计数器不能直接重置为0，如果要这样也可以自己实现。

```Java
@Slf4j
public class CountDownLacth {
    private static int threadCount = 200;
    public static void main(String[] args) throws InterruptedException {
        ExecutorService exec = Executors.newCachedThreadPool();
        final CountDownLatch countDownLatch = new CountDownLatch(threadCount);
        for (int i = 0; i < threadCount; i++) {
            final int threadNum = i;
            exec.execute(()->{
                try {
                    test(threadNum);
                } catch (InterruptedException e) {
                    log.info("{}", e);
                }finally {
                    countDownLatch.countDown();//每一次完成之后都要减一
                }
            });
        }
        //countDownLatch.await(10,TimeUnit.MILLISECONDS); //传入参数等待10ms之后没有完成也不再等了
        countDownLatch.await(); //调用await，要等待所有之前的线程完成后完成
        log.info("finish");
        exec.shutdown();//关闭线程池
    }

    private static void test(int threadNum) throws InterruptedException {
        Thread.sleep(100);
        log.info("{}", threadNum);
        Thread.sleep(100);

    }
}
```



### Semaphore

```Java
@Slf4j
public class SemaphoreExample1 {
    private static int threadCount = 200;
    public static void main(String[] args) throws InterruptedException {

        ExecutorService exec = Executors.newCachedThreadPool();

        final Semaphore semaphore = new Semaphore(10); //当前有多少个许可

        for (int i = 0; i < threadCount; i++) {
            final int threadNum = i;
            exec.execute(()->{
                try {

                    semaphore.acquire(); //获取许可。可以选择参数，一次性拿走多少个许可
                    test(threadNum);
                    semaphore.release(); //释放许可

                } catch (InterruptedException e) {
                    log.info("{}", e);
                }
            });
        }
        exec.shutdown();
    }

    private static void test(int threadNum) throws InterruptedException {
        log.info("{}", threadNum);
        Thread.sleep(1000);

    }
}
```



### CyclicBarrier

允许一组线程相互等待，直到某个公共等待点。线程调用`await()`时计数加一并且进入等待，一直到特定计数时候，所有线程可以继续执行。计数器可以进行重置，进行重复使用。

```Java
@Slf4j
public class CyclicBarrierExample1 {
    private static CyclicBarrier barrier = new CyclicBarrier(5);//设定计数器为5

    public static void main(String[] args) throws InterruptedException {

        ExecutorService exec = Executors.newCachedThreadPool();

        for (int i = 0; i < 10; i++) {
            final int threadNum = i;
            Thread.sleep(1000);
            exec.execute(()->{
                try {
                    rece(threadNum);
                } catch (Exception e) {
                    log.error("except", e);
                }
            });
        }
        exec.shutdown();
    }

    private static void rece(int threadNum) throws Exception {
        Thread.sleep(1000);
        log.info("{} is ready", threadNum);
        barrier.await(); //加计数器,可以加入参数，表示等待时间
        log.info("{} is continue",threadNum);
    }
}
```



### ReentrantLock

可重入锁。

* 和synchronized的区别

  * 可重入性：区别不大
  * 锁的实现：sync基于JVM实现，Reen是JDK实现的。
  * 性能区别：sync引入了轻量锁，自旋锁和偏向锁后，性能和Reen差不多了。官方建议使用sync
  * 功能区别：sync方便一些。reen需要手动释放锁，所以最好在finally里面写。reen锁的灵活度比sync好

* ReentrantLock独有功能

  * 可指定公平锁和非公平锁
  * 提供一个Condition类，可以分组唤醒需要唤醒的线程
  * 提供能够中断等待锁线程的机制。lock.lockInterruptibly()

  **ReentrantLock使用一种自旋锁，循环调用CAS操作加锁，避免线程进入内核态的阻塞状态**

### Condition



### FutureTask



# 线程池-ThreadPoolExecutor

Thread类：不好控制，所以使用线程池

使用线程池的好处：

* 重用存在的线程，减少系统开销
* 有效控制并发，提高资源利用率
* 提供定期执行，定时执行，单线程和并发等控制

参数：

* corePoolSize：核心线程数量
* maximunPoolSize：线程最大线程数
* workQueue：阻塞队列，储存等待执行的任务。

关系：如果运行的线程数小于corePoolSize，直接创建新线程处理任务，尽管线程池中有空闲线程。如果线程池中的数量大于等于corePoolSize但是小于maximunPoolSize，只有workQueue满了才创建新线程处理任务。如果corePoolSize和maximunPoolSize相同，则线程池大小固定，如果workQueue没有满，则将任务放入。如果workQueue已满，则使用拒绝策略去处理任务。

# 死锁

四个条件：

* 互斥条件
* 请求和保持条件
* 不剥夺条件
* 环路等待条件

避免死锁：

* 加锁顺序：代码加锁的顺序
* 加锁时间：使用ReentrantLock
* 死锁检测：检测到了就回退，设置随机线程优先级

# 多线程并发的最佳实践

* 使用本地变量
* 使用不可变类
* 最小化锁的作用域范围：S=1/(1-a+a/n)  a是并行比例，n是并行处理节点个数
* 使用线程池的Executor，而不是直接使用new Thread
* 宁可使用同步也不要使用线程的wait和notify
* 使用BlockingQueue实现生产-消费模式
* 使用并发集合而不是加了锁的同步结合
* 使用Semaphore创建有界的访问
* 宁可使用同步代码块，也不使用同步方法
* 避免使用静态变量

 