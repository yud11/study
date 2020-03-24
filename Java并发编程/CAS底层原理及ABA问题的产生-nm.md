### 前言
> CAS（compare and swap）它是一条CPU并发原语.
> 它的功能是判断内存某个位置的值是否为预期值,如果是则更新为新的值,这个过程是原子的

### 原子包装类和CAS
> java.util.concurrent.atomic存在许多原子包装类，它们都是在基本数据类型包装类的基础上实现了原子性，这里我们拿AtomicInteger来举例  

```java
//首先我们来看一下AtomicInteger的getAndIncrement()方法，该方法的作用是Integer类型的数据自增，等同于num++  
//此方法在原本自增操作的基础上加上了原子性的特征

 // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();

/**
     * Atomically increments by one the current value.
     *
     * @return the previous value
     */
    public final int getAndIncrement() {
        return unsafe.getAndAddInt(this, valueOffset, 1);
    }
```

> getAndIncrement()方法底层调用的是unsafe对象的getAndAddInt()方法，我们进入getAndAddInt()方法中，可以看到最终的实现是compareAndSwapInt()方法，我们再往下深入会发现该方法是Unsafe类中被native关键字修饰的方法。

> **native**即Java本地接口（JNI,Java Native Interface），是一种编程框架，使得Java虚拟机中的Java程序可以调用本地应用/或库，也可以被其他程序调用。 本地程序一般是用其它语言（C、C++或汇编语言等）编写的, 并且被编译为基于本机硬件和操作系统的程序

```java

public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

        return var5;
    }
    
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```
> 首先我们来分析getAndAddInt()方法，该方法传入三个参数分别是
> * Object var1：当前对象
> * long var2：内存偏移量（即内存地址）
> * int var4：自增值1
>
> 接下来程序进入do while循环并调用方法getIntVolatile()将当前对象和内存地址作为参数传入，该方法根据某个具体对象和该对象的内存地址取出该对象具体的值。得到具体的值var5，此后程序进入while语句中进行判断  
> compareAndSwapInt(Object var1, long var2, int var4, int var5)中的四个参数分别代表
> * Object var1：当前对象
> * long var2：内存地址
> * int var4：期望值
> * int var5：修改后的值
>
> 首先由于compareAndSwapInt是一个native修饰的方法，也就是说该方法是是直接调用操作底层资源执行响应的任务。由于该方法是一种系统原语,原语属于操作系统用于范畴,是由若干条指令组成,用于完成某个功能的一个过程,并且原语的执行必须是连续的,在执行过程中不允许中断,也即是说该方法是一条原子指令,不会造成所谓的数据不一致的问题。  
> 该方法根据当前对象和当前对象的内存地址取出当前对象的值，并和自己设定的期望值进行比较，如果该值等于期望值，那么进行更新，这便是CAS（比较和交换）的核心概念。如果说这里判断失败返回flase，那么继续循环重新取值，直到取到的值等同于我们定义的期望值，这样可以保证不管有多少个线程进入num++的操作，每个线程永远都会取到最新值并进行更新，从而保证了数据的原子性。

### CAS的缺点
* 由于使用了do while循环，如果多次比对不正确，循环次数变多开销变大，效率变低，降低了并发性
* CAS是对原子包装类的操作，所以只能保证一个共享变量的原子性
* 会引出ABA问题

### 关于ABA问题的探讨

#### 什么是ABA问题
>CAS的核心思想就是通过对象内存地址取值并和期望值进行比较，但是取出并进行比较的过程中，该共享变量可能会被其他线程操作，虽然可能最终期望值和取出的值对比是一致的，但实际上现在的共享变量已经不等于原来的共享变量了。  
>通过一个简单的例子说明。A买了10个苹果放在了桌子上，B跑过来拿走了5个苹果吃掉了，然后又买了5个苹果放在了桌子上，这时候A过来算苹果数量一数是10个，那么A就想当然的认为这10个苹果依然是之前的10个苹果，但实际上在这个过程中B已经对这些苹果进行了替换。

#### ABA问题的代码体现

``` java
/**
 * 控制台输出
 *    T1	 initial value is 10
 *    T2	 initial value is 10
 *    true
 *
 * @Description: ABA问题
 * @Author: Niem
 * @Date: 2020/3/2-22:55
 */
 
public class ABADemo {
    public static void main(String[] args) {
        //创建原子包装类对象
        AtomicInteger atomicInteger = new AtomicInteger(10);

        new Thread(()->{
            //获取atomicInteger的值
            System.out.println(Thread.currentThread().getName()+"\t initial value is "+atomicInteger.get());
            //暂停一秒确保T2线程可以获得和T1相同的值
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) { e.printStackTrace(); }
            //如果期望值是10，那么就把该值更新为5
            atomicInteger.compareAndSet(10,5);
            //如果期望值是5，那么就把该值更新为10
            atomicInteger.compareAndSet(5,10);
        },"T1").start();
        new Thread(()->{
            //获取atomicInteger的值
            System.out.println(Thread.currentThread().getName()+"\t initial value is "+atomicInteger.get());
            //暂停3秒，确保T1的CAS操作完成
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) { e.printStackTrace(); }
            //如果期望值是10，那么就把该值更新为15,如果返回值是true，代表CAS操作成功，但实际上对象已经被操作替换过了
            System.out.println(atomicInteger.compareAndSet(10, 15));
        },"T2").start();
    }
}
```

#### 解决方案
> 如果我们让原子包装类对象带上版本信息，给对象带上标记，那么在进行比较的时候就不光要比对内容是否相同，还需要比对版本是否一致。  
> 所以我们可使用AtomicStampedReference类来实现这一功能，该类是在原子引用类的基础上加上了时间戳的概念。

> 从下列代码中，我们得出当版本比对不一致时，就算值是相同的，也是无法进行CAS操作的，时间戳原子引用类从版本的角度解决了ABA的问题

```java
/**
 * @Description: ABA问题解决
 * 
 * 控制台输出
 * T1     initial version is 1	 the initial value is 10
 * T2	 initial version is 1	 the initial value is 10
 * The current version of T1 is 3 ,the latest value is 10
 * the result of CAS: false
 * The current version of T2 is 3 ,the latest value is 10
 * 
 * @Author: Niem
 * @Date: 2020/3/2-23:16
 */
public class ABADemoSolve {
    public static void main(String[] args) {
        //创建时间戳原子引用类对象,指定版本为1
        AtomicStampedReference<Integer> reference = new AtomicStampedReference<Integer>(10,1);

        new Thread(()->{
            //获取atomicInteger的值
            System.out.println(Thread.currentThread().getName()+"\t initial version is "+reference.getStamp()+"\t the initial value is "+reference.getReference());
            //获取初始版本号
            int stamp = reference.getStamp();
            //暂停一秒确保T2线程可以获得和T1相同的值
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) { e.printStackTrace(); }
            //如果期望值是10，那么就把该值更新为5,并且把版本号加1
            reference.compareAndSet(10,5,stamp,stamp+1);
            //如果期望值是5，那么就把该值更新为10
            reference.compareAndSet(5,10,reference.getStamp(),reference.getStamp()+1);
            System.out.println("The current version of T1 is "+reference.getStamp()+" ,the latest value is "+reference.getReference());
        },"T1").start();
        new Thread(()->{
            //获取atomicInteger的值
            System.out.println(Thread.currentThread().getName()+"\t initial version is "+reference.getStamp()+"\t the initial value is "+reference.getReference());
            //获取初始版本号
            int stamp = reference.getStamp();
            //暂停3秒，确保T1的CAS操作完成
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) { e.printStackTrace(); }
            //如果期望值是10，那么就把该值更新为15,如果返回值是true，代表CAS操作成功，但实际上对象已经被操作替换过了
            System.out.println("the result of CAS: "+reference.compareAndSet(10, 15,stamp,stamp+1));
            System.out.println("The current version of T2 is "+reference.getStamp()+" ,the latest value is "+reference.getReference());
        },"T2").start();
    }
}
```