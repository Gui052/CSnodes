# 线程安全概念

不管何种调度方式，不需要额外的同步或者协同，都能表现正确的行为

* 原子性：互斥访问
* 可见性：一个线程对主存的修改能及时被其他线程观察
* 有序性：尽管所有线程执行顺序改变，单个线程的执行顺序不会改变

# 原子性

## Atomic包

AtomicXXX方法使用了Unsafe类的CompareAndSwap方法，就是常说的CAS，此方法将内存中的值读取，和预期的值比较，如果相等则允许更新，不相等则循环再取。

## 锁

* synchronized：依赖于JVM
* Lock：依赖特殊的CPU指令，代码实现-ReentrantLock

### synchronized

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

### 对比

* synchronized：不可中断，适合竞争激烈，可读性好
* Lock：可中断锁，多样化同步，竞争激烈时维持常态
* Atomic：竞争激烈时能维持常态，比Lock性能更好。但是一次只能同步一个值

# 可见性

导致不可见的原因：

* 线程交叉执行
* 重排序结合线程交叉执行
* 共享变量更新后的值没有在工作内存与主存间及时更新

## synchronized

* 线程解锁前必须把共享变量的最新值刷新回到主存
* 线程加锁时，将清空工作内存中共享变量的值，重新从主存中读取最新值

## volatile

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

# 有序性

如果两个线程的操作顺序无法从happen-before规则推导出来，就不能保证有序性，虚拟机可以随意进行重排序

## happen-before规则

* 程序次序规则：一个线程内，按照书写顺序执行
* 锁定规则：一个unlock操作先行发生于后面同一个锁的lock操作
* volatile变量规则：对于一个变量的写操作先行发生于后面对这个变量的读操作
* 传递规则：如果操作A先行发生于B，B又先行于C，则A先于C
* 线程启动规则：Thread对象的start()方法先行与此线程的每一个动作
* 线程中断规则：对于线程interrupt()方法的调用 先行发生于 被中断线程的代码 检测到中断事件的发生
* 线程终结规则：线程中所有操作都先行发生于线程的终止检测
* 对象终结规则：一个对象的初始化完成先行发生于它的finalize()方法的开始