# 针对面试简历的知识点
# Kotlin基础
## Kotlin空安全
Kotlin空安全是一种类型系统机制，通过显式的区分可空类型和非空类型，在编译阶段预防空指针异常(NullPointerException, NPE)<br>

通过编译器类型检查和字节码注入空值检查指令(CheckNotNull)，在代码逻辑违反空安全规则时抛出异常
Kotlin空安全会引入运行时开销，但可以忽略不计

与Java @Nullable最大的不同，Java运行时仍需手动判空

## 什么是Kotlin逆变、协变
Kotlin逆变、协变是泛型编程概念，定义了父类和子类之间两种不同的类型替换
协变：关键字 `out`，通配符 `? extend`, 表示子类可以替代父类类型，仅用于读的场景，例如:

```kotlin
class Animal
class Cat:Animal

fun println(data:List<out Animal>){
    for(e:data){
        println(e)
    }
}

val data = arrayOfList(Cat(),Cat())
println(data)

```

常用场景：返回一个只读集合

逆变：关键字`in`, 通配符 `? super`, 表示父类可以替换子类类型，仅用于写的场景，例如

```kotlin
class Event
class ButtonEvent:Event

fun callback(e:List<in ButtonEvent>){
    e.add(Event())
}

val buttonEvent = listOf(Event())
callback(buttonEvent)

```

常用场景：事件监听、Compare比较器

## 为什么协变限制写入，逆变限制读取
保证类型安全，对于协变，如果不限制写入，有可能导致类型不确定，比如

```kotlin
val list = listOf<Cat>()
fun feedAnimal(a:List<out Animal>){
    a.add(Dog()) // error
}
```

协变允许子类替代父类，但子类可能存在多种，如果允许写入，会导致该类型不确定，所以协变限制写，保障类型精确性
对于逆变，逆变允许父类取代子类，如果允许写，可能导致父类调用子类的方法，直接报错，所以逆变限制读，保障类型兼容性

## 什么是Kotlin协程
### Kotlin协程和Java线程的区别
- 协程是轻量级资源占用。Kotlin启动一个协程占用内存约2kb（动态增加），Java线程默认占用1MB栈内存（可以通过`-Xxs` 调整），单个进程支持的线程数只有几千（JVM特性决定），但是Kotlin可以轻松开启上万协程
- 更好的线程调度和切换。协程可以在没有操作系统内核介入的前提下实现执行权的调度，上下文切换仅保存程序计数器、栈指针和关键计数器，耗时小，执行比较快；Java线程依赖操作系统抢占式切换、强制中断线程，上下文切换需要保存完整线程资源（寄存器、堆栈、线程变量），耗时相对大
- 协程可以通过`supend`非阻塞性挂起，挂起时释放底层线程资源，供其他协程复用；线程挂起是阻塞性操作，会占用线程资源，协程通过少量线程资源实现高并发IO操作，避免了资源浪费
- 结构化并发与生命周期管理。通过`CoroutineScope`实现结构化并发，父协程取消时，所有子协程自动取消，避免了内存泄漏；Java线程需要手动管理，容易内存泄漏。
- 协程通过`awit\async`实现更好的异步操作，Java线程需要通过线程回调来实现切换，容易造成回调嵌套地狱，可读性差
- 更灵活的调用和线程池优化。Kotlin可以通过`Dispatchers.Main` `Dispatchers.IO` 等精准控制协程执行的线程池，Java需要手动配置线程池，分配不当会导致资源争抢

### 协程和线程使用场景
- 协程适合高并发、网络请求、数据IO、UI异步更新、业务回调嵌套较多的场景
- 线程适合CPU密集（图像处理）、依赖`ThreadLocal`等Java组件、依赖直接控制线程优先级或者内核调度的场景

### 协程

# AIDL

