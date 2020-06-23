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

  

# 并发



## CAS

- CAS：compare and swap
- CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。 如果内存位置的值与预期原值相匹配，那么处理器会自动将该位置值更新为新值 。否则，处理器不做任何操作。无论哪种情况，它都会在 CAS 指令之前返回该 位置的值。（在 CAS 的一些特殊情况下将仅返回 CAS 是否成功，而不提取当前 值。）CAS 有效地说明了“我认为位置 V 应该包含值 A；如果包含该值，则将 B 放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可。”
- CPU(汇编层面)实现的功能，不是Java本身的机制
- CAS操作失败表示内存中的值已经被其他线程修改，需要使用被修改后的值来重新做计算，计算后再把新值通过CAS写到内存中（可以在一个循环中不断尝试）
- Java中的CAS是通过jni调用C语言和汇编代码实现的
- volatile只能保证内存可见性，但不能保证操作的原子性，比如自增操作就不是一个原子性操作，分为读数据，加1，写数据的几步，读的时候读到的数据可以保证是最新的，但是加一后写到内存中时，并不能保证内存中的数据还是之前的数据，所以在写到内存中的时候需要一个CAS操作，先看看内存中的值是不是我之前从内存中读出来的值，如果是，才写进去，如果不是，会写失败并返回最新的值，从而可以根据最新的值重新计算后在进行一次CAS



## volatile

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





















