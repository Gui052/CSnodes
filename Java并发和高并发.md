# 线程安全性

不管何种调度方式，不需要额外的同步或者协同，都能表现正确的行为

* 原子性：互斥访问
* 可见性：一个线程对主存的修改能及时被其他线程观察
* 有序性：尽管所有线程执行顺序改变，单个线程的执行顺序不会改变

## 原子性

### Atomic包

AtomicXXX方法使用了Unsafe类的CompareAndSwap方法，就是常说的CAS，此方法将内存中的值读取，和预期的值比较，如果相等则允许更新，不相等则循环再取。

### 锁

* synchronized：依赖于JVM
* Lock：依赖特殊的CPU指令，代码实现-ReentrantLock

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



# 线程封闭

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

# 线程安全--并发容器J.U.C

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