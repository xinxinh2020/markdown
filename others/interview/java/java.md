[TOC]

# 基础
## 面向对象三大特性
三大特性：封装，继承和多态

- 封装：增加程序的可读性，隐藏实现细节
- 继承：解决的是代码的复用性和可维护性问题,可以从一个类派生出另一个新的类，继承父类的属性和方法
- 多态：



## 重写和重载

- 重写(Override)：子类通过重写父类方法，覆盖父类的实现

- 重载：一个类中可以同时存在多个同名的方法，只要参数的声明不一样即可



## Java的平台无关性是如何实现的

- JVM屏蔽了底层操作系统和硬件差异

- Java面向JVM编程，先编译成字节码文件，再交给JVM解释成机器码执行（先编译后解释）

## 抽象类和接口的区别

- 一个类可以实现多个接口，但只能实现一个抽象类（has a, is a）
- 接口中只能有常量(static final)，不能有变量(普通成员变量)
- JDK8中加入了接口的默认实现，目的是为了在给一个接口加入新方法时可以不影响已经实现该接口的实现类，实现向下兼容(如java.util.Collection接口)。如果多个接口有相同的默认实现方法，实现时就需要子类显式实现该方法，否则会报错，因为编译器不知道要使用哪个实现。实际使用时应该谨慎考虑接口默认方法的使用



## 元注解有哪些

- @Target : 注解修饰对象的范围
- @Retention：定义注解的保留时间，可取的值有：SOURCE，CLASS，RUNTIME
- @Documented：标记该成员可以被javadoc工具文档化
- @Inherited：标记类型是被继承的



## 反射机制

- 动态地获取信息和调用对象的方法

- 作用：判断一个对象所属的类，在运行时构造一个类的对象，判断任意类所具有的成员方法和变量，调用任意一个对象的方法（动态代理）

  

## 异常处理

### Exception和Error的区别

- Exception是程序正常运行时可以预料到可能出现的错误，并且应该被捕获并进行相应的处理。 Exception包括编译时异常和运行时异常，编译时异常要求要求在代码中进行相应的处理，运行时异常表示运行时可能出现的异常，如空指针，数组越界等，运行时异常在代码中不进行处理编译也不会报错
- Error是正常情况下不可能发生的错误，一旦出现会导致JVM处于一种不可恢复的状态，不需要捕获处理（也没办法处理），不如OOM

### 异常处理的原则

- 尽可能处理具体的异常，而不是全部都使用一个Exception
-  捕获异常后要妥善的处理，不要什么都不做，至少要打印一下日志方便问题排查
- 不要try...catch一大段，这样不利于问题的定位排查，性能也会受影响，try..catch尽可能只包住可能跑出的异常的代码
- 本模块捕获到异常不知道怎么处理时，可以抛给上层去处理，切忌默默把异常吃掉了



### NoClassDefFoundError和ClassNotFoundException的区别

- NoClassDefFoundError是代码中使用new创建对象时，编译时是能通过的，但到了运行时却找不到该类，一般是打包时少了某些包或者包被篡改了导致，它是个Error，无法被捕获处理，程序会直接挂掉
- ClassNotFoundException是通过类名反射调用来创建一个类的对象的时候找不到该类导致，一般是类名传错了，这个可以也应该在代码中捕获处理



## JIT编译器

把经常运行的热点代码直接编译成本地平台相关的机器码，并进行各种层次的优化。除了具有缓存之外，还会对代码做各种优化，如逃逸分析，方法内联，锁消除等。

### 逃逸分析

动态分析对象的作用域，如果分析到一个对象不会被方法外访问时，也就是对象不会逃逸到方法外面（通过参数传递等方式），就可以把这个对象拆成包含若干个基本类型的成员变量，这个过程也叫做标量替换。标量替换的好处就是对象可以不在堆内存进行分配，为栈上分配提供了良好的基础。

缺点是技术还不是很成熟，分析比较耗时，如果没有分析到对象是没有逃逸的，那就得不偿失了。



## 值传递和引用传递

值传递：传递的是对象的副本，即使副本被修改，对象本身也不会受到影响

引用传递：传递的是对象的引用，因此外部对引用的对象的修改会影响所有引用

值传递可以认为是将数据在栈上进行传递，方法调用的参数实际上都是值传递，只不过基本数据对象在栈上传递的就是对象本身，而引用数据类型在栈上传递的是对象的引用



## StringBuffer和StringBuilder的区别

StringBuffer是线程安全的，StringBuilder是非线程安全的，但StringBuilder效率高一点

## 对Java泛型的理解

泛型是一个很重要的特性，它实际上就是参数化数据类型，在不创建新数据类型的情况下，通过泛型参数控制生成具体的数据类型。比较多的应用是在集合中。泛型是在编译期生效的，方便编译器进行类型检查，运行时会去泛型化，也就是说泛型实际上只是一颗语法糖，实际在运行阶段JVM并不关心你这个数据类型是普通类型还是通过泛型生成的，所以也对JVM的性能不会有影响。

golang被人吐槽最多的一点之一就是不支持泛型，不知道为什么，一开始开发golang的几个大佬并不喜欢用泛型，可能他们更习惯用C语言。但是有意思的时，golang开发组内部也有许多人是支持使用泛型的，所以在其中几位大佬淡出核心开发组以后，泛型的支持就被提上了日程，传说中2.0版本会支持泛型。

## 序列化和反序列化的过程

序列化是把对象转换成字节序列从而可以保存到硬盘上或者发送到网络上的过程，反序列化是把字节序列还原成对象的过程

## equals和hashCode的区别

## Java和C++的区别有哪些

最主要的就是指针和内存管理

## 静态和非静态的区别

## equals和==的区别

==比较两个引用是否相同，equals默认实现和==相同，重写后可以用来通过对象的属性判断两个对象是否相等

# 三大集合



## HashMap和Hashtable的区别

Hashtable是线程安全的，对put和remove等操作进行了synchronized同步，而HashMap是非线程安全的，但性能比Hashtable好。

Hashtable不允许null作为键或者者，但HashMap可以。



## fast-fail

集合在进行迭代的时候如果有线程（包括本身或其他线程）对集合进行了增删操作，在迭代的时候会抛出一个ConcurrentModificationException异常。集合中是通过一个modCount字段来判断判断进行迭代时集合是否被改动过，一旦有对集合的增删操作都好记录到这个字段，这样在通过迭代器遍历时一旦检测到这个字段发生了变动，就可以抛出异常，而不用遍历到被修改的数据导致后续不可以预知的错误，所以这个机制叫fast-fail



## HashMap的底层实现

JDK8之前使用的是数组+链表的方式，JDK8及其以后使用数组+链表+红黑树的方式实现。数组用来存放Hash值，链表用来存放所有hash值相同的键值对（实际上就是使用了外拉链法来处理Hash冲突）。每个数组元素及其对应的外拉链表称为一个桶（bucket），链表长度过长时会影响查询效率，因此在链表长度超过一定长度时（TREEIFY_THRESHOLD= 8），链表会被改造成红黑树。

HaspMap的初始容量为16，加载因子为0.75，扩容增量为2倍。扩容时需要将所有元素重新放到新的bucket数组中，这个过程叫做rehashing。

使用懒汉式创建hash表，只有真正用到的时候才创建Hash表

常用处理Hash冲突的方法有拉链法（Java使用），线性探测再散列法等

 HashMap不保证顺序。

## 哪些类适合作为HashMap的键

String，Interger这样的包装类很适合用来做HashMap的键，因为他们时final的类并且重写了equals和hashCode方法，final类型可以防止这些方法被修改。

## 一致性Hash算法（Todo）

- 

## ConcurrentHashMap

参考文章：[ConcurrentHashMap底层实现原理((JDK1.7&1.8))](https://blog.csdn.net/zhang18024666607/article/details/92796400?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522159209615419724835857705%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=159209615419724835857705&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_click~default-2-92796400.first_rank_ecpm_v3_pc_rank_v2&utm_term=concurrenthashmap)

### JDK 1.7实现

使用分段锁机制，降低锁的粒度

ConcurrentHashMap由多个Segment组成，每个Segment持有一个锁，对同一个Segment中的元素进行访问需要进行加锁操作

默认的concurrencyLevel，即Segment的数量是16

Size操作：先尝试通过不加锁的方式的多次计算size的值（最多三次），比较前后两次计算的结果，相同则认为当前没有元素加入，计算结果是准确的，尝试失败后改用加锁的方式，把每个Segment锁起来计算它们的size

### JDK 1.8实现

使用数组+链表+红黑树

put的时候，如果没有hash冲突，就直接无锁CAS插入，如果存在hash冲突，就要锁住链表或红黑树的头节点

链表的长度大于8（TREEIFY_THRESHOLD）时会自动转换成红黑树

比JDK1.7的实现粒度更细，直接使用synchronized来加锁而不用可重入锁

## ConcurrentLinkedQueue



## TreeMap

- 底层使用红黑树实现

- 它是有序的，按照键来排序

- 实现键有序的两种办法：键对象实现Comparable接口或创建TreeMap的时候传入一个Comparator:

  ```java
  // 方式一：定义该类的时候，就指定比较规则
  class User implements Comparable{
      @Override
      public int compareTo(Object o) {
          // 在这里边定义其比较规则
          return 0;
      }
  }
  public static void main(String[] args) {
      // 方式二：创建TreeMap的时候，可以指定比较规则
      new TreeMap<User, Integer>(new Comparator<User>() {
          @Override
          public int compare(User o1, User o2) {
              // 在这里边定义其比较规则
              return 0;
          }
      });
  }
  ```

- Comparable的好处是实现比较简单，但比较规则修改时需要修改键对象的源码
- Comparator的好处是比较灵活，不需要改动键对象的代码

## ArrayList和LinkedList的区别

- ArrayList底层使用的是一个动态数组，LinkedList底层使用的是一个双向链表
- ArrayList的随机存取效率比较高，LinkedList的增删效率比较高
- ArrayList必须预留一定空间，LinkedLIst必须花费一些空间来存储节点信息和节点指针信息
- 它们都是线程不安全的，线程安全的List可以使用CopyOnWriteArrayList



## HashSet和TreeSet的区别

- HashSet底层实际上是一个HashMap，只不过它只用到了HashMap的key,所有键值对的值都是同一个Object对象

  ```java
      private transient HashMap<E,Object> map;
  
      // Dummy value to associate with an Object in the backing Map
      private static final Object PRESENT = new Object();
  ```

- 和HashSet一样，TreeSet底层实际上是一个TreeMap

  ```java
      private transient NavigableMap<E,Object> m;
      private static final Object PRESENT = new Object();
  
      TreeSet(NavigableMap<E,Object> m) {
          this.m = m;
      }
  
      public TreeSet() {
          this(new TreeMap<E,Object>());
      }
  
      public boolean add(E e) {
          return m.put(e, PRESENT)==null;
      }
  ```

  

- HashSet是通过键的Hash值和equals方法来保证唯一性的，而TreeSet是通过Comparable或Comparator接口



## LinkedHashMap和LinkedHashSet

- LinkedHashMap除了具有HashMap的性质之外，还可以记录插入顺序或访问顺序（二选一）

- LinkedHashMap的Entry继承了HashMap.Node,它还有before,after两个指针，通过一个双向链表保证各个元素插入的顺序，创建entry时，把entry链接到tail后面，本质上，它是一个将所有Entry节点链入一个双向链表的HashMap

  ```java
      static class Entry<K,V> extends HashMap.Node<K,V> {
          Entry<K,V> before, after;
          Entry(int hash, K key, V value, Node<K,V> next) {
              super(hash, key, value, next);
          }
      }
  
  		// 创建entry时，把entry链接到tail后面
      Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
          LinkedHashMap.Entry<K,V> p =
              new LinkedHashMap.Entry<K,V>(hash, key, value, e);
          linkNodeLast(p);
          return p;
      }
  
      private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
          LinkedHashMap.Entry<K,V> last = tail;
          tail = p;
          if (last == null)
              head = p;
          else {
              p.before = last;
              last.after = p;
          }
      }
  ```

- LinkedHashMap可以实现LRU缓存算法（访问顺序）

- LinkedHashSet 底层使用LinkedHashMap实现，两者的关系类似与HashMap和HashSet的关系

  
  
  
  
## CopyOnWriteArrayList/CopyOnWriteArraySet

  参考：https://ifeve.com/java-copy-on-write/

适用于读多写少的场景，对写操作加锁，对读操作不加锁，只能保证数据的最终一致性，但不能保证数据的实时一致性。

# 并发



## CAS

- CAS：compare and swap
- CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。 如果内存位置的值与预期原值相匹配，那么处理器会自动将该位置值更新为新值 。否则，处理器不做任何操作。无论哪种情况，它都会在 CAS 指令之前返回该 位置的值。（在 CAS 的一些特殊情况下将仅返回 CAS 是否成功，而不提取当前 值。）CAS 有效地说明了“我认为位置 V 应该包含值 A；如果包含该值，则将 B 放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可。”
- CPU(汇编层面)实现的功能，不是Java本身的机制
- CAS操作失败表示内存中的值已经被其他线程修改，需要使用被修改后的值来重新做计算，计算后再把新值通过CAS写到内存中（可以在一个循环中不断尝试）
- Java中的CAS是通过jni调用C语言和汇编代码实现的
- volatile只能保证内存可见性，但不能保证操作的原子性，比如自增操作就不是一个原子性操作，分为读数据，加1，写数据的几步，读的时候读到的数据可以保证是最新的，但是加一后写到内存中时，并不能保证内存中的数据还是之前的数据，所以在写到内存中的时候需要一个CAS操作，先看看内存中的值是不是我之前从内存中读出来的值，如果是，才写进去，如果不是，会写失败并返回最新的值，从而可以根据最新的值重新计算后在进行一次CAS



## volatile

解决内存可见性问题。多线程一般会把内存中的数据读到cpu的缓存中，并发读的时候就会有问题。
Java的内存模型规定，所有变量都必须存储在主存之中，每个线程都有自己的工作内存（类似于高速缓存），线程对变量的操作必须在工作内存中进行，不能直接对主存进行操作。
普通共享变量不能保证内存可见性，变量被修改后什么时候从工作空间写入到主存中是不确定的。
volatile修饰的共享变量可以保证内存可见性，当这些变量被修改时，会保证修改的值立即写入到内存中，并且让其他线程中该变量的缓存无效，其他线程要访问该变量时，只能去主存中重新读取。
加锁也能保证内存可见性，因为加锁之后可以保证同一时间只有一个线程会执行同步代码，释放锁的时候会把工作内存中的共享变量写入到主存中。

- volatile保证了内存可见性，但不能保证操作的原子性，如++操作本身就不是一个原子操作
- 要保证数据的原子性操作，可以使用AtomicXXX开头的包装类，如AtomicInteger，这些类的修改数据操作都是使用CAS操作进行的，CAS操作失败会一直循环重试到成功为止，通过CAS的方式来保证原子性（同步）性能会比加锁要好

## synchronized和ReentrantLock的区别

- synchronized和ReentrantLock都是悲观锁
- synchronized 竞争锁时会一直等待，ReentrantLock 可以尝试获取锁，并得到获取结果
- synchronized 获取锁无法设置超时，ReentrantLock 可以设置获取锁的超时时间
- synchronized 无法实现公平锁，ReentrantLock 可以满足公平锁，即先等待先获取到锁
- synchronized 是 JVM 层面实现的；ReentrantLock 是 JDK 代码层面实现
- synchronized 在加锁代码块执行完或者出现异常，自动释放锁；ReentrantLock 不会自动释放锁，需要在 finally{} 代码块显示释放
- synchronized 控制等待和唤醒需要结合加锁对象的 wait() 和 notify()、notifyAll()；ReentrantLock 控制等待和唤醒需要结合 Condition 的 await() 和 signal()、signalAll() 方法

## 乐观锁和悲观锁

参考博文：[什么是悲观锁和乐观锁](https://blog.csdn.net/striveb/article/details/84826921?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522159223717419195239817828%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=159223717419195239817828&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-84826921.first_rank_ecpm_v3_pc_rank_v2&utm_term=%E4%B9%90%E8%A7%82%E9%94%81+%E6%82%B2%E8%A7%82%E9%94%81)

悲观锁总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁（**共享资源每次只给一个线程使用，其它线程阻塞，用完后再把资源转让给其它线程**），Java中`synchronized`和`ReentrantLock`等独占锁就是悲观锁思想的实现

乐观锁总是假设最好的情况，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号机制和CAS算法实现。**乐观锁适用于\**多读\**的应用类型，这样可以提高吞吐量**，在Java中`java.util.concurrent.atomic`包下面的原子变量类就是使用了乐观锁的一种实现方式**CAS**实现的

**乐观锁适用于写比较少的情况下（多读场景）**，可以加大系统的吞吐量。但如果是多写的情况，一般会经常产生冲突，这就会导致上层应用会不断的进行retry，这样反倒是降低了性能，所以**一般多写的场景下用悲观锁就比较合适。**



## 乐观锁的缺点

- ABA问题
- 自旋CAS失败次数过多增加CPU负载
- 最对单个变量有效，但JDK1.5之后提供了AtomicReference来保证对象修改的原子性，可以把多个变量放到一个对象中，把多个共享变量合并成一个共享变量来进行CAS操作



## NIO

阻塞io->轮询io->使用selector

我们可以创建多个通道，那么我们怎么知道哪个通道有事件就绪呢？有两种方法，一种是去轮询，一种是使用回调。轮询我们懂知道，说最简单但性能也比较差的，linux内核有个poll命令，就说去轮询哪些io(即文件描述符fd)已经就绪了。大概在linux2.6之后，人们觉得poll性能太差了，于是又提出了一个epoll的概念，它使用事件注册的方式来管理多个fd，fd注册到epoll以后，epoll不需要去轮询它的状态，而是fd就绪以后给epoll发送事件，epoll就知道有fd就绪了。
jdk 1.5以后的nio利用了epoll这个机制，nio可以用一个selector专门来管理所有通道的状态，适合并发度很高但每个并发io的数据量又不大的场景。实际上调用selector的select()后也是会阻塞的，所以在并发度不高的情况下使用nio不一定性能就比传统io要好，但是在并发度很高但每个并发io的数据量又不大的场景就很好用了。

## 阻塞/非阻塞，同步/异步
阻塞/非阻塞是指用户线程发起io请求后，需不需要等待结果返回后才能继续执行。
阻塞非阻塞是线程的一种状态。同步异步指的是被调用者结果返回时通知线程的一种机制。

# 高级特性

## 反射机制

```java
public interface Hello {
    String sayHello(String hi);
}

public class HelloImpl implements Hello{
    public String sayHello(String hi) {
        System.out.println("hello " + hi);
        return "hello world";
    }
}

public class ReflectMain {
    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        HelloImpl hello = new HelloImpl();
        Method method = Hello.class.getMethod("sayHello", String.class); // 通过反射得到接口的方法对象
        Object str = method.invoke(hello,"zhansan"); // 通过反射对象调用接口
        System.out.println("return " + str);
    }
}
```



## 代理模式

1. 访问控制：A不能被直接访问，通过代理来控制访问
2. 功能增强

相关视频：[动态代理&反射机制](https://www.bilibili.com/video/BV1HZ4y1p7F1?p=1)

### 静态代理

优点：简单，容易理解

缺点：目标类增加，代理类也要增加；接口中方法改变了，所有实现类都要变化

代码示例：

```java
// 目标对象和代理对象都要实现的接口
public interface UsbSell {
    float sell(int amount);
}

// 目标对象
public class UsbKingFactory implements UsbSell{
    public float sell(int amount) {
        return 85.0f * amount;  // 一个Usb出厂价85.0块钱
    }
}

// 代理类
public class Taobao implements UsbSell{
    private UsbKingFactory factory = new UsbKingFactory(); //代理了目标对象
    public float sell(int amount) {
        float price = factory.sell(amount);
        // 此处可以添加增强功能
        return price + amount * 10; // 每个U盘赚10块钱中间差价
    }

    public static void main(String[] args) {
        Taobao taobao = new Taobao();
        float price = taobao.sell(1);
        System.out.println("Buy a usb by taobao costing :" + price);
    }
}
```



### 动态代理

使用JDK的反射机制，创建代理对象的能力（目标对象能不能动态创建？），而不用创建类文件。

有两种实现：

1. JDK动态代理，使用JDK中的反射包中的相关类和接口来实现动态代理
2. Cglib（Code Generation Library）动态代理，是第三方类，实现原理是继承，在MyBatis和Spring中都有所使用

#### 基于接口的动态代理：Proxy

newProxyInstance创建一个代理对象，该对象实现了第二个参数传进去的接口(所以可以强制转型为相应接口)，第三个参数指定一个InvocationHandler的实现类，该实现类编写的invoke方法中的代码逻辑指定了如何代理目标接口的方法（需要注意的是，目标接口的实现类一般保存在InvocationHandler的实现类中）。

示例代码：

```java

// 实现了InvocationHandler
public class UsbSellProxy implements InvocationHandler {
    private UsbSell usb;
    public UsbSellProxy(UsbSell usb){
        this.usb = usb;
    }

    // 该方法代理了newProxyInstance中指定的接口中的方法，当调用了newProxyInstance返回的代理对象上的方法时，这些方法会被当作参数传到invoke方法的method参数中，这些方法的参数会被传到args参数中，invoke方法的处理逻辑一般是进行一些处理之后，调用被代理的目标对象相应的方法。所以一般需要给InvocationHandler传一个被代理接口的实现类（不然代理什么呢哈哈）
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("in proxy");
        return method.invoke(usb,args);
    }

    // 
    public UsbSell getProxy(){
        return (UsbSell) Proxy.newProxyInstance(UsbKingFactory.class.getClassLoader(), UsbKingFactory.class.getInterfaces(),this);
    }

    public static void main(String[] args) {
        UsbSellProxy proxy = new UsbSellProxy(new UsbKingFactory());
        UsbSell usbSell = proxy.getProxy();
        usbSell.info();
        usbSell.sell(10);
    }
}
```





#### 基于继承的动态代理：Cglib





# JVM

## 内存模型

### 线程私有
- 程序计数器，每个线程都有一块独立的内存（但很小）来存放该线程所执行的字节码的行号指示器。
- 虚拟机栈，每个线程都有一个，生命周期和线程相同。每个方法在执行的同时都会创建一个栈帧压入到该线程的栈中。栈帧中的局部变量表的大小在进入方法前就已经是完全确定的，在运行期间不会发生变化。
- 本地方法栈，和虚拟机栈类似，只不过虚拟机栈是为虚拟机执行Java方法（字节码）服务，而本地方法栈是为Native方法服务。有些虚拟机直接就把本地方法栈和虚拟机栈合二为一了。


### 线程共享

- Java堆，存放对象实例。
- 方法区，存放类信息，常量，静态变量等。HotSpot用户常称之为永久代。

## 对象的创建
- 当虚拟机遇到一条new指令时，会去常量池中定位到一个类的引用，并检查这个类是否已经加载，没有的话要先加载。
- 然后为新生对象分配内存。给对象分配内存也是一门学问，根据使用的GC收集器不同，内存中分为规整和不规整两种情况。规整的内存分配只需要移动指针的位置就可以了(指针碰撞)，如果是不规整的，就需要维护一个列表来记录哪些内存是未使用的。分配内存的时候还需要考虑并发地创建对象时的线程不安全问题，TLAB为解决这个问题的方案之一。
- 将内存空间初始化为0
- 设置对象的对象头，保存一些信息，如GC分代年龄，类的元数据，对象的哈希码等
- 执行<init>方法

## TLAB
本地线程分配缓冲（Thread Local Allocation Buffer），解决并发地创建对象时，两个线程都去修改堆内存指针（空闲内存和已使用内存的分界线）导致的问题。


## 垃圾回收机制
### 哪些内存需要回收
- 引用计数法：无法解决对象相互引用的情况，主流虚拟机没有采用这种算法
- 可达性分析算法：通过“GC ROOT”对象作为起点向下搜索，形成一条引用链，没有在引用链上的对象就是要回收掉的对象。GC ROOT对象包括以下几种：
    - 虚拟机栈中引用的对象
    - 方法区中类静态属性和常量引用的对象

### 回收时机
没有在GC引用链中的对象，会被标记，在回收前会调用对象的finalize()方法，对象可以在finalize中拯救自己，否则就会被回收掉。

### 垃圾收集算法
- 标记清除算法：会产生大量内存碎片
- 复制算法：每次使用一个Eden和一个Survivor，回收时把所有对象复制到另外一个Survivor，如果Survivor不够放，会使用老年代进行分配担保，把放不下的对象放到老年代里。经历过多次GC仍存活的对象也会被放到老年代中。

## 类加载过程
生命周期：加载，验证，准备，解析，初始化，使用，卸载
- 加载
    - 通过类的全限定名来获取定义此类的二进制字节流（并没有指定从哪里获取）
    - 把二进制流代表的类保存到方法区的运行时数据结构
    - 在内存中(按道理应该是在堆内存中的老年代？)生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口(保存的是类的元数据信息吧，比如名字？)
- 验证：确保加载的class文件的字节流是符合虚拟机要求的，可以防止一些恶意的代码导致系统奔溃
- 准备：在方法区为类变量分配内存并设置初始值
- 解析：把符号引用替换为直接引用
- 初始化：执行类构造器(<clinit>()方法)，执行静态变量赋值和静态代码块
  

### 类加载器/双亲委派
通过一个类的全限定名来获取描述此类的二进制字节流，应用程序可以自己决定如何去获取所需要的类。判断两个类是否相等，只有在这两个类是由同一个类加载器加载的前提才有可能。
系统提供的类加载器：
- 启动类加载器(Bootstrap)：加载$JAVA_HOME/lib下的类
- 扩展类加载器(Extension)：加载$JAVA_HOME/lib/ext下的类
- 系统类加载器(Application)：

### 初始化的时机
- 在遇到new或访问静态变量或调用静态方法时
- 使用java.lang.reflect包进行反射调用时
- 初始化一个类的时候，如果父类还没初始化，也要对父类进行初始化
- 虚拟机启动时，用户指定要执行的包含main方法的类
- jdk1.7 java.lang.invoke.MethodHandle实例解析到REF_getStatic的方法句柄











