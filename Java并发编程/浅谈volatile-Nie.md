### 前言
> volatile是jvm提供的一种轻量的同步机制，它相较于synchronize性能更优，在某些场合比synchronize更加适用，它具有三种特性分别是
> * 保证可见性
> * 不保证原子性
> * 禁止指令重排

### volatile特性一：可见性
#### 理解可见性的抽象概念
如下图所示，有三个线程 T-1，T-2，T-3，主内存中存在一个对象A，对象A的age属性数值为24，现在定义一个方法，将对象A的age数值从24改为30，这三个线程去主内存中请求克隆该数值的副本到工作内存（工作内存即为jvm为每一条线程创建的独立的存储区域）中，此时T-1线程此时将age=24copy了一个副本出来到自己的工作内存中，并执行方法将age数值修改为了30，此时需要执行数据的写回，也就是将修改后的age数值重新写回主内存中，假设T-1线程首先完成了这个操作，那么在数据写回主内存后，T-1需要避免此时还未完成age数值变更的T-2，T-3线程再次进行重复将数据写回主内存中的操作，所以需要通知T-2，T-3线程，此时主内存中的age数值已经从24变更为30了。而这个通知的机制即是**可见性**的抽象体现。

![image](https://note.youdao.com/yws/api/personal/file/880CFB7FF2C649AA800E14E8FDC3C2B3?method=download&shareKey=2e58bdeb06841d475d262f2190f559c7)

---

#### 代码实例体现可见性

>主线程中新建的线程thread执行了方法ageTo30()使age数值从24更新为了30，但是主线程确没有获取到age数值的变化，在这个例子中，没有使用关键字volatile，可见性没有体现出来
```java
class Counter{
    //定义变量age=24
    int age = 24;
    //定义更新方法ageTo30
    public  void  ageTo30(){
        age = 30;
    }
}
public class VolatileDemo {
    public static void main(String[] args) {

        Counter counter = new Counter();

        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+" is come in");
            try {
                Thread.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            //当前线程执行ageTo30方法将age从24更新至30
            counter.ageTo30();
            System.out.println(Thread.currentThread().getName()+" change age from 24 to "+counter.age);
        },"AAA").start();
        //如果age=24的话，让主线程在这里挂起
        while (counter.age==24){
        }
        //没有输出以下信息说明主线程没有得到age数值更新到30的信息
        System.out.println(Thread.currentThread().getName()+" thread is over!");
    }
}
//控制台输出的内容
/**
 AAA is come in
 AAA change age from 24 to 30
*/
```

> 此时我们使用volatile修饰属性age，再次执行相同的操作，控制台输出主线程的信息，可以看出，新建的线程修改了age属性并把这个信息通知给了主线程使其得知了age属性数值的变化，从而体现出了volatile的可见性。

```java
class Counter{
    //定义变量age=24
    volatile int age = 24;
    //定义自增方法ageTo30
    public  void  ageTo30(){
        age = 30;
    }
}
public class VolatileDemo {
    public static void main(String[] args) {

        Counter counter = new Counter();

        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+" is come in");
            try {
                Thread.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            counter.ageTo30();
            System.out.println(Thread.currentThread().getName()+" change age from 24 to "+counter.age);
        },"AAA").start();
        //如果age=24的话，让主线程在这里挂起
        while (counter.age==24){
        }
        System.out.println(Thread.currentThread().getName()+" thread is over!");
    }
}
//控制台输出的内容
/**
 AAA is come in
 AAA change age from 24 to 30
 main thread is over!
*/
```

---

### volatile特性二：不保证原子性

#### 理解原子性

原子性指的是某一个或者某一些操作的集合是一个完整的个体，是不可分割和加塞的，这在volatiile中的体现也是相同的，volatile不保证原子性即不能保证一个操作流程的完整，也不能保证该操作流程不能被加塞。

#### 代码示例

> 从以下代码中我们可以看出，volatile并不具备原子性的特征，我们期望输出的数值应该是20000，但是在这里结果却输出了18484，而且每执行一次都会出现不一样的数值，这说明在线程执行的过程中，出现了重复写的情况。

```java
class Counter{
    //定义变量age=0
    volatile int age = 0;
    //定义自增方法ageTo30
    public  void  agePlus(){
        age ++;
    }
}
public class VolatileDemo {
    public static void main(String[] args) {
        //new Counter对象
        Counter counter = new Counter();
        //循环建立20个线程
        for (int i = 0; i < 20; i++) {
            new Thread(()->{
                //循环1000次agePlus方法
                for (int j = 0; j < 1000; j++) {
                    counter.agePlus();
                }
            },String.valueOf(i)).start();
        }
        //判断20个线程分别执行1000次自增方法是否结束
        while (Thread.activeCount()>2){
            Thread.yield();
        }
        System.out.println("current age is:"+counter.age);
    }
}
//控制台输出的内容
/**
 current age is:18484
*/
```

#### 为什么会出现重复写的情况？

> 让我们从程序执行的步骤开始分析，查看源码的字节码文件
> 我们可以发现，代码"age++"的操作在底层指令中可以分成4个指令
> * getfield
> * iconst_1
> * iadd
> * putfield
>
> 这4个指令组成了一个操作集合，由于volatile不具备原子性，那么就可以会出现A线程和B线程都进入了agePlus方法中，假设此时age=1，此时A线程执行完指令iadd，age的数值变为2，并执行putfield指令将数据写回主内存，但此时B线程执行到了iadd执行，由于同一时间只能有一个线程往主内存中更新age的数值，所以B线程在iadd指令这里挂起了，当A线程将age更新为2后，B内存接着执行putfield的指令将age数值再次更新为了2，所以出现了重复写的情况。在这种情况下线程A是来不及通知其他线程age数值已经更新为2的。因为age++不同于age = 30，它是由4个指令组成的操作，会出现线程加塞的情况。

```java
  com.thread.Counter();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: aload_0
       5: iconst_0
       6: putfield      #2                  // Field age:I
       9: return

  public void agePlus();
    Code:
       0: aload_0
       1: dup
       2: getfield      #2                  // Field age:I
       5: iconst_1
       6: iadd
       7: putfield      #2                  // Field age:I
      10: return
}
````

#### 如何解决
* 方式一，给agePlus方法加上synchronize修饰，同一时间只能有一个线程去操作age数值
* 方式二，使用Integer的包装类：AtomicInteger,并调用该类的自增方法getAndIncrement()，该类可以保证Integer类型数据的原子性
```java
public class Counter {
        //定义变量age=0
        AtomicInteger age = new AtomicInteger(0);
        //定义自增方法ageTo30
        public  void  agePlus(){
            age.getAndIncrement();
        }
}
```

---

### volatile特性三：禁止指令重排

#### 什么是指令重排？

计算机在执行程序时,为了提高性能,编译器和处理器常常会做指令重排,一把分为以下3种

![image](https://note.youdao.com/yws/api/personal/file/B2813B9EB59946AD996F5FD13402C5BB?method=download&shareKey=2a9bba0db60806525db89772521529e7)

单线程环境里面确保程序最终执行结果和代码顺序执行的结果一致。
处理器在进行重新排序是必须要考虑指令之间的<font color=blue>数据依赖性</font>。
多线程环境中线程交替执行,由于编译器优化重排的存在,两个线程使用的变量能否保持一致性是无法确定的,结果无法预测


#### 单例模式在多线程下出现指令重排

> 这是一个标准的懒汉式单例模式，在单线程的前提下，控制台正常输出



```java
/**
控制台输出：
true
true
true
*/
public class OrderSort {
    private static OrderSort orderSort = null;
    private OrderSort(){
        System.out.println(Thread.currentThread().getName()+" new OrderSort Object!!");
    }
    public static OrderSort getOrderSort(){
        if (orderSort==null){
            orderSort = new OrderSort();
        }
        return orderSort;
    }
    public static void main(String[] args) {
        System.out.println(OrderSort.getOrderSort() == OrderSort.getOrderSort());
        System.out.println(OrderSort.getOrderSort() == OrderSort.getOrderSort());
        System.out.println(OrderSort.getOrderSort() == OrderSort.getOrderSort());
    }
}
```

> 如果场景变成了多线程的环境，懒汉式就无法维持单例模式的特性了，在多线程的环境下可能会出现获取多次实例对象的情况，且每次输出的对象数量随机

```java
/**
 * 控制台输出：
 * 0 new OrderSort Object!!
 * 3 new OrderSort Object!!
 * 4 new OrderSort Object!!
 * 2 new OrderSort Object!!
 * 1 new OrderSort Object!!
 */
public class OrderSort {
    private static OrderSort orderSort = null;
    private OrderSort(){
        System.out.println(Thread.currentThread().getName()+" new OrderSort Object!!");
    }
    public static OrderSort getOrderSort(){
        if (orderSort==null){
            orderSort = new OrderSort();
        }
        return orderSort;
    }
    public static void main(String[] args) {
        //创建10个线程执行获取实例的静态方法
        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                OrderSort.getOrderSort();
            },String.valueOf(i)).start();
        }
    }
}
```

#### 为什么会出现线程安全问题，如何解决？

假设现在有两个线程分别是T-1和T-2，当T-1执行到orderSort = new OrderSort()这一步时，此时T-2执行orderSort==null进行非空判断，由于T-1并没有完成orderSort引用对象的赋值，所以T-2线程判断orderSort依然为null，那么理所应当T-1和T-2都可以获得对应的实例对象，最终获得两个对象实例，懒汉式失去单例模式特性。

#### 解决方式：DCL(double check lock)单例模式

> 控制台输出只得到了一个对象实例，看似是解决了懒汉式的线程安全问题，然而DCL模式下的懒汉式依然是线程不安全的

``` java
/**
 * 控制台输出：
 * 0 new OrderSort Object!!
 */
public class OrderSort {
    private static OrderSort orderSort = null;
    private OrderSort(){
        System.out.println(Thread.currentThread().getName()+" new OrderSort Object!!");
    }
    public static OrderSort getOrderSort(){
    //第一次判断
        if (orderSort==null){
            //对类进行加锁
            synchronized (OrderSort.class){
                //再次检查
                if (orderSort==null){
                    orderSort = new OrderSort();
                }
            }
        }
        return orderSort;
    }
    public static void main(String[] args) {
        //创建10个线程执行获取实例的静态方法
        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                OrderSort.getOrderSort();
            },String.valueOf(i)).start();
        }
    }
}
```

#### DCL缺陷&指令重排问题

DCL(双端检锁) 机制不一定线程安全,原因是有指令重排的存在，原因在于某一个线程在执行到第一次检测,读取到的instance不为null时,instance的引用对象可能没有完成初始化。
instance=new SingletonDem(); 可以分为以下步骤(伪代码)  
<font color=red size =4>重排前</font>  
memory=allocate();//1.分配对象内存空间  
instance(memory);//2.初始化对象  
instance=memory;//3.设置instance的指向刚分配的内存地址,此时instance!=null 

步骤2和步骤3不存在数据依赖关系.而且无论重排前还是重排后程序执行的结果在单线程中并没有改变,因此这种重排优化是允许的.  
<font color=red size =4>重排后</font>  
memory=allocate();//1.分配对象内存空间  
instance=memory;//3.设置instance的指向刚分配的内存地址,此时instance!=null    但对象还没有初始化完.  
instance(memory);//2.初始化对象  

但是指令重排只会保证串行语义的执行一致性(单线程) 并不会关心多线程间的语义一致性  
所以当一条线程访问instance不为null时,由于instance实例未必完成初始化,也就造成了线程安全问题.

> 这里引用周阳老师的举的一个例子：  
> * 有A同学需要转班到某班级，老师看到教室里有一个座位是空着的，于是告诉大家，这个座位分配给了A同学（内存分配-步骤1），他过1个小时就会坐到这个位置上。于是大家开始给A同学准备书本、文具（初始化对象-步骤2），过了一个小时，A同学来到这个位置坐下（引用对象指向分配的内存地址-步骤3）  
> * 如果出现了指令重排，那么由于步骤2和步骤3不存在数据上的依赖，那么就会出现，A同学提前一个小时来到座位坐下，这时候大家才给他准备书本和文具（先指向内存，再进行初始化，此时执行到第二步的A同学虽然已经和座位建立了联系，但是由于没有书本和文具是不具备上课的条件的，此时第一次判断instance!=null，于是return instance得到了一个地址不为空，但是实际内容为空的对象）

#### volatile配合DCL禁止指令重排

> volatile实现禁止指令重排的原理是“内存屏障”，内存屏障是一种cpu指令，通过在指令建插入内存屏障指令可以实现禁止前后指令重新优化排序，同时在volatile写操作之后加入内存屏障指令（store），会让工作内存中的变量强制刷新回主内存（实现了可见性）

```java
/**
 * 控制台输出：
 * 0 new OrderSort Object!!
 */
public class OrderSort {
    //添加volatile修饰符
    private static volatile OrderSort orderSort = null;
    private OrderSort(){
        System.out.println(Thread.currentThread().getName()+" new OrderSort Object!!");
    }
    public static OrderSort getOrderSort(){
        if (orderSort==null){
            //对类进行加锁
            synchronized (OrderSort.class){
                //再次检查
                if (orderSort==null){
                    orderSort = new OrderSort();
                }
            }
        }
        return orderSort;
    }
    public static void main(String[] args) {
        //创建10个线程执行获取实例的静态方法
        for (int i = 0; i < 100000; i++) {
            new Thread(()->{
                OrderSort.getOrderSort();
            },String.valueOf(i)).start();
        }
    }
}
```