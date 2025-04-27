# 1 Java中==和equals的区别，hashCode什么作用  

Java的数据类型分为基本类型和引用类型，对于基本类型，==表示值比较，对于引用类型，==比较的是内存中的存放地址，对象是放在堆中的，栈中存放的是对象的引用地址，具体来说，==比较的是栈中的值，如果要比较对堆中的数据，那么需要重写equals方法  

默认情况下，equals方法主要是用于判断对象的内存地址引用是不是同一地址，即比较栈内存中的地址值（这个地址指向堆中的对象），此时equals与==是等效的，参见根类Object.equals的实现    
```java
// Object.java  equals内部调用了==
public boolean equals(Object obj) {
    return (this == obj);
}
``` 

重写equals方法需要满足自反性、对称性、传递性和一致性，同时需要重写hashCode，确保相等对象有相同哈希码（Java规范的要求）  

hashCode主要用于哈希表相关操作  

几个例外情况：  

1. 常量池优化  
```java
String s1 = "hello"; // 指向字符串常量池中的对象
String s2 = "hello";
System.out.println(s1 == s2); // true（常量池优化，同一对象）
```

2. 自动装箱和缓存  
包装类在-128和127之间的值会被缓存：  
```java
Integer a = 127;
Integer b = 127;
System.out.println(a == b); // true（缓存对象）

Integer c = 128;
Integer d = 128;
System.out.println(c == d); // false（超出缓存范围，新建对象）
```

# 2 int和Integer的区别  

Java为每个基本数据类型引入对应的包装类，从Java5引入自动装箱、拆箱机制，使得基础类型和对应的包装类能够自动转换   

int是基本类型，初始值默认0，Integer是包装类，初始值默认是null  


# 3 Java多态的理解  

Java多态有两种形式：编译时多态和运行时多态  

编译时多态通过方法重载实现，编译器根据方法签名（方法名+参数类型+参数数量）在编译阶段确定调用具体哪个方法  

运行时多态通过方法重写和向上转型实现，JVM在运行时根据对象的实际类型动态绑定方法，动态绑定的底层机制是虚方法表，每个类在加载时会为每个方法分配一个虚方法表，表中记录了方法实际的内存入口地址，调用方法时，JVM会根据实际调用类型查找虚方法表    

# 4 String、StringBuffer、StringBuilder的区别  

String:字符串常量，使用字符串拼接时是两个不同的空间  
StringBuffer：字符串变量，线程安全，字符串拼接直接在原字符串添加  
StringBuilder：字符串变量，非线程安全，字符串拼接直接在原字符串添加  

Builder执行效率高于Buffer、高于String  
String是一个不可变产量，所以每次+=赋值都会创建一个新的对象  
Builder和Buffer都是可变的，拼接字符串直接在原字符串后添加，但是因为Buffer是线程安全的，所以效率上地狱Builder  
JDK 1.6之后String做了优化，+拼接字符串时，底层调用了Builder，在效率不敏感的场景，String和Builder没有太大差异  
Buffer线程安全都是通过给方法加锁来实现的，这是早期JDK实现线程安全的主要方式，现在可以考虑使用`ThreadLocal<StringBuilder>` 或者对特定代码块加锁  

# 5 什么是内部类、内部类的作用  

一般外部类是没有private、protected修饰，但是内部类可以，所以内部类可以很好的隐蔽实现细节，控制访问权限，又能够拥有外部类的全部访问权限  

# 6 抽象类和接口的区别、应用场景  

抽象类使用abstract class定义，既可以有抽象方法，也可以有其他方法，既可以有静态属性，也可以有非静态属性  

接口用interface定义，属性用public final static修饰，通常可以缺省，JDK 8之后，可以使用默认方法  

抽象类是一种类别，具有构造方法，接口是一种行为规范，没有构造方法，也不持有数据，抽象类也可以没有任何抽象方法，但是抽象类无法直接实例化，必须由子类实现继承后才能实例化  

抽象类只能单继承，接口可以被多个类实现，接口和接口之间可以单继承  

抽象类的抽象方法可以由public、protected、default修饰，接口方法必须是public  

从设计思想上来看，抽象类描述的是对象的本质属性（持有数据）和行为，适合构建层级关系，接口描述对象能做什么，而非描述对象是什么，不持有任何数据  

# 7 泛型中extends和super的区别  

1. <? extends T> 上界通配符  

表示可接收T类型以及其子类，生产者模式，只读不写
```java
List<? extends Number> numbers = new ArrayList<Integer>(); // 合法
Number num = numbers.get(0); // 安全读取

// numbers.add(10); // 编译错误！无法确定具体子类型
```

2. <? super T> 下界通配符  

表示可接收T及其父类类型，消费者模式，可写可读（必须是Object类型）
```java
List<? super Integer> integers = new ArrayList<Number>(); // 合法
integers.add(10); // 安全写入（Integer是Number的子类）

Object obj = integers.get(0); // 只能读取为Object
// Integer num = integers.get(0); // 编译错误！可能实际是Number或其他父类
```


# 8 静态属性、方法能否被重写  

不能，子类继承父类后，静态属性和方法都被隐藏，无法重写

# 9 进程和线程的区别  

进程是资源分配的基本单位，线程是处理器调度的基本单位  

进程拥有独立的地址空间，线程没有独立的地址空间，同一个进程内多个线程共享其资源  

线程属于进程，当进程退出时，所有线程都会被强制清除  


# 10 final/finally/finalize的区别  



# 11 什么是序列化、为什么要序列化  


# 12 Serizlizable、Parcelable的区别  


# 13 静态内部类的设计意图  


# 14 成员内部类、静态内部类、方法内部类（局部内部类）和匿名内部类的区别和应用场景    


# 15 闭包和局部内部类的区别  


# 16 String转换成Integer的方式和原理  


# 17 对象拷贝中深拷贝和浅拷贝的区别  


# 18 Enumeration和Iterator的区别  

