# 1 类加载流程的双亲委托机制

# 2 Java虚拟机中的静态分派和动态分配  

# 3 JVM内存模型  

```
┌──────────────────────────┐
│        程序计数器         │ ➜ 每个线程私有，记录当前执行字节码地址
├──────────────────────────┤
│         Java虚拟机栈      │ ➜ 每个线程私有，存储方法调用栈帧
├──────────────────────────┤
│          本地方法栈       │ ➜ 每个线程私有，支持 native 方法调用
├──────────────────────────┤
│            堆            │ ➜ 所有线程共享，存储对象实例（GC 管理）
├──────────────────────────┤
│         方法区（元空间）  │ ➜ 所有线程共享，类信息、静态变量、常量池等
└──────────────────────────┘
```

## 程序计数器（Program Counter Register）
每个线程独有，线程私有   
存储当前线程所执行的 字节码指令地址（偏移量）  
是唯一一个不会 OOM（OutOfMemoryError）的区域  

## Java 虚拟机栈（JVM Stack）
每个线程独有，线程私有  
每个方法调用时，都会创建一个 栈帧（Stack Frame）  
局部变量表（int a, Object o）  
操作数栈   
动态链接  
方法返回地址等   

异常或栈太深会报：`StackOverflowError`,`OutOfMemoryError`

## 本地方法栈（Native Method Stack）
与 JVM 栈类似，但用于 native 方法调用

不使用 Java 字节码，而是与平台有关的 native 代码（如 C）

## 堆（Heap）

所有线程共享

存储 对象实例、数组

垃圾回收器（GC）的主要管理区域

按生命周期可进一步分为：

新生代（Young Gen）：Eden + Survivor（S0/S1）  
老年代（Old Gen）  
大对象区  

报错：`OutOfMemoryError: Java heap space`

## 方法区（Method Area）—— Java 8 起称为 元空间（Metaspace）
所有线程共享

存储：

类的元信息（结构、字段、方法）

静态变量

常量池（运行时常量）

Java 7 之前：方法区在 堆中，叫 PermGen（永久代）

Java 8 之后：移出堆内存，使用 本地内存（称为 Metaspace）

报错：

`OutOfMemoryError: Metaspace`
