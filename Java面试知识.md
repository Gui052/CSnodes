#### 1. Java中数据类型以及长度

|  类型   | 位(一个字节8位) |   封装   | 默认值 |
| :-----: | :-------------: | :------: | :----: |
|  byte   |        8        |   Byte   |   0    |
|  short  |       16        |  Short   |   0    |
|   int   |       32        | Interger |   0    |
|  long   |       64        |   Long   |   0L   |
|  float  |       32        |  Float   |  0.0F  |
| double  |       64        |  Double  |  0.0D  |
|  char   |       16        | Charater |   空   |
| boolean |        8        | Boolean  | false  |

#### 2. 面向对象四个特征

封装，抽象，继承，多态

#### 3. 装箱和拆箱

将基本类型转换为包装类，叫装箱。需要这样转换是因为`java`是面向对象的语言，基本数据类型没有面向对象的特征。

#### 4. ==和equals有什么区别

`==` 用来判断变量之间的值是否相等。如果是基本数据类型，就比较值，如果是引用类型就比较引用的内存首地址。`equals` 比较的是对象的某些特征是否一样。实际上就是调用对象的equals方法，这个方法是由程序自己定义的。

#### 5. String，StringBuilder，StringBuffer有什么不同

String是不可变的字符串  String s=new String("strings")，底层使用了不可变的数组(final char[] value)。其他两个都是可变字符串，底层使用的是可变数组(没有final修饰)

拼接字符串：String的直接`+` 会创建三个对象，所以使用其他两个效率更高(append方法)

StringBuilder是线程不安全的，效率高，StringBuffer是线程安全的，效率低

#### 6. Java中的集合

Java中集合分为value，key-value两种。储存value的有List和Set，储存key-value的是Map。

* List是有序的，可以重复的
* Set是无序的，不可以重复的。根据equals和hashcode判断，所以存在Set里面的元素，就必须重写equals和hashcode的方法。
* map是自动根据key排序的，key不能修改，其对应的value可以更改，不允许key重复

#### 7. ArrayList和LinkList的区别

ArrayList底层使用的是数组。LinkList底层使用的是链表。所以前者查询特定索引的元素比较快，后者插入删除快。所以ArrayList使用的是查询比较多的场景，LinkList使用的是插入删除比较多的场景。

#### 8. HashMap和HashTable的区别

两者都可以用来存储key-value的数据

​	HashMap可以用null作为key或者value，HashTable不行

​	HashMap是线程不安全的，效率低。

​	HashTable是现成安全的，效率高。

​	ConcurrentHashMap是线程安全的，其内部使用了多个HashMap

#### 9. 拷贝文件使用字节流还是字符流

我们拷贝的文件不知包含字符，为了考虑通用性，要使用字节流。

#### 10. 线程的实现方式

* 通过继承Thread类实现
* 通过实现Runable接口实现

Java中只有单继承，如果继承Thread就不能继承其他类了。

怎么启动：

```java
Thread thread=new Thread（继承了Thread的对象/实现了Runable的对象）
thread.setName("设置一个线程名称")
thread.start();
启动后执行run方法。
```

#### 11. 线程并发库

java通过Executor提供四种静态方法创建四种线程池

* newCachedThreadPool创建可缓存线程池 ，如果长度超过所需，可以自动回收空闲线程，如果不够则创建新的。
* **newFixedThreadPool**（常用）创建一个定长线程池，可控制线程最大并发量，超出的线程会在队列中等待
* newScheduledThreadPool创建一个定长线程池，支持定时以及周期性的执行任务
* newSingleThreadPool创建一个单线程化的线程池，保证所有线程按照一定顺序（FIFO,UFO，优先级）执行

  作用：

1. 限制线程个数，不会导致由于线程过多导致溢出。

2. 线程池不用每次需要再去创建或销毁，节约资源

3. 不需要每次创建，响应更快

   **连接池也一样**

#### 12. 常用的设计模式

* 单例模式：
  * 饱汉模式：一开始就创建
  * 饥汉模式：需要时创建，注意多线程的情况，需要进行同步操作
    1. 构造方法私有化
    2. 在自己的类中创建一个单实例
    3. 提供一个方法获取该实例的对象（创建时需要进行方法同步）
* 工厂模式：Spring IOC
  * 对象的创建交给一个工厂去创建，自己不用
* 代理模式：Spring AOP

#### 13. forward和redirect

* forward是服务器端跳转，还是原来的请求，链接没有变，效率高

* redirect是客户端的跳转，是重新发起的请求，链接改变，效率低

#### 14.session和cookie的区别

都是会话跟踪技术。session是服务端记录，cookie是客户端记录，但是session依赖于cookie，sessionID。所以登录信息放在session中，其他信息放在cookie中（购物车的实现可以放在cookie，但是cookie是可以禁用的，所以需要使用cookie+数据库的方式）

