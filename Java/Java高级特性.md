# 1 哪些情况下对象会被垃圾回收机制处理掉  

1. 引用链不可达  

对象在内存中，从任何GC Roots出发，无法通过引用链访问到它，这个对象就是待GC的垃圾对象  

比如说方法体内new的对象，在方法执行完之后，自动回收  

```java
void test() {
    Object obj = new Object(); // obj是局部变量，指向堆中的对象
}
// test()执行完后，obj出栈，堆上Object没有任何引用了 => 引用链断了 => 可以GC
```

也可以主动赋null，区别在于主动null可以更快的GC，无需等方法体执行完，适合大内存（大图片、大文件等）占用及时释放  

```java
public void loadLargeImage() {
    byte[] imageData = loadFromDisk(); // 加载一个很大的图片文件到内存
    
    processImage(imageData); // 使用imageData做点什么

    imageData = null; // **主动断开引用**

    // 后面还有很多逻辑，比如其他小数据的处理
    doOtherThings();
}

```

优化点：如果频繁new对象，可以考虑把对象缓存起来，小对象直接存，如果是Bitmap这种大对象，可以考虑用LRUCache策略，目的是减少频繁GC带来的性能损耗

2. 强引用断开后，弱引用、软引用、虚引用的回收  

- 强引用：正常new出来的对象，只要存在，就不回收  
- 弱引用（WeakReference）：GC时发现弱引用，直接回收  
- 软引用（SoftReference）：内存紧张时回收  
- 虚引用（PhantomReference）：任何时候都能回收，仅用于监听回收事件  

```java
import java.lang.ref.WeakReference;

public class Example3 {
    public static void main(String[] args) {
        Object obj = new Object();
        WeakReference<Object> weakRef = new WeakReference<>(obj);

        obj = null; // 断开强引用

        System.gc(); // weakRef指向的对象很快会被回收
        System.out.println(weakRef.get()); // 可能输出null
    }
}

```

3. 对象存在于可回收的数据结构里  

如WeakHashMap

4. 类卸载时，静态字段回收  

一个类的静态字段只有在类卸载时才可能被回收，通常自定义ClassLoader加载类并且让ClassLoader没有引用时，类才卸载

5. 内存溢出时，紧急回收  

当系统内存不足时，GC会努力回收那些能回收的对象，尽可能避免OOM（OutOfMemoryError）  

# 2 常见的编码方式有哪些  

编码的原因：计算机存储信息的最小单元是一个字节8个bit，所以能表示的字符范围是0~255，显然无法表示所有的字符，所以需要一个编码规则定义多个字节所代表的字符编码  

ASCII码：128个字符，用一个字节的低7位表示，0-31是控制字符，32-126是打印字符  

UTF-16：定义了Unicode字符在计算机中的存储方法，定长两个字节共16位  

UTF-8：UTF-16定长2个字节有点浪费空间，所以UTF-8采用变长字节，最小1个字节8位，最多6个字节（后被限制到4个字节）  


# 3 UTF-8编码一个中文占几个字节，int型占几个字节  

一个汉字通常3~4字节，int型占用与编码方式无关，Java中固定4个字节  

# 4 静态代理和动态代理的区别，什么场景使用  



# 5 Java的异常体系  
# 6 Java虚拟机中的静态分派和动态分配  
# 7 修改对象A的equals方法的签名，那么使用HashMap存放这个对象实例中，会调用哪个equals方法  
# 8 Java中多态的底层实现机制是什么  
# 9 如何将一个Java对象序列化到文件中  
# 10 说说你对Java反射的理解  
# 11 说说你对Java注解的理解  
# 12 说说你对依赖注入的理解  
# 13 说一下泛型原理，并举例说明  
# 14 String为什么要设计成不可变的  

1. 安全性

JVM用String存储类名、方法名，不可变性保证类加载时不会被篡改  
使用String存储敏感信息，如文件路径、网络链接、密码等，避免恶意修改  

2. 线程安全  

不可变对象天然线程安全，不需要同步即可在多线程环境下共享，提升性能  

3. 性能优化  

字符串常量池：JVM通过常量池复用相同字符串，避免内存开销  
String作为哈希表键值对，hashCode仅需计算一次即可  


# 15 类加载流程的双亲委托机制  
