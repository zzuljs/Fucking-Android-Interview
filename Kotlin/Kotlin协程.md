# ☕ 基础原理类

## 什么是 Kotlin 协程？基本使用方式有哪些？它与线程有什么区别？

Kotlin协程（Coroutine）是Kotlin提供的一种轻量级异步编程解决方案，用于简化异步任务、并发任务的编写，通过挂起函数（suspend）的方式，让异步代码写起来像同步代码一样，提升可读性和可维护性  

```kotlin
// 启动协程
import kotlinx.coroutines.*

fun main() = runBlocking { // 创建一个协程作用域
    launch {
        delay(1000L) // 挂起1秒，不阻塞主线程
        println("Hello, Coroutine!")
    }
    println("Hello, World!")
}

// 使用suspend函数
suspend fun fetchData(): String {
    delay(1000L) // 模拟异步操作
    return "Data"
}

fun main() = runBlocking {
    val data = fetchData()
    println(data)
}

// 使用async/await获取结果
fun main() = runBlocking {
    val deferred = async {
        fetchData()
    }
    val result = deferred.await()
    println(result)
}

```

| 特性   | 协程 Coroutine      | 线程 Thread           |
| ---- | ----------------- | ------------------- |
| 概念   | 轻量级协作任务单元         | 操作系统级别的资源单元         |
| 启动成本 | 极低（数千协程可运行在一个线程上） | 高（每个线程分配独立栈空间，几 MB） |
| 切换成本 | 极低（只需保存协程状态）      | 高（上下文切换涉及线程状态、寄存器）  |
| 是否阻塞 | 默认不阻塞线程，支持挂起      | 默认阻塞                |
| 控制方式 | 协作式挂起、恢复          | 抢占式调度               |
| 可取消性 | 支持结构化取消，容易管理      | 难取消或控制              |

## 什么是挂起函数（suspend function）？它和普通函数的本质区别是什么？

suspend函数是Kotlin协程中的核心概念，是指可以被挂起并在稍后恢复执行的函数、用来表示异步、非阻塞式操作  

编译器会把挂起函数转换成一个状态机（State Machine）+Continuation接口，以便在挂起点记录当前状态，并在恢复时继续执行  

普通函数像是一站到底的高铁，中途无法停止，挂起函数像是自驾车，可以根据需求停靠、出发  

```kotlin
suspend fun loadData() {
    withContext(Dispatchers.IO) {
        // 切换到 IO 线程池
        val data = fetchFromNetwork()
    }

    withContext(Dispatchers.Main) {
        // 回到主线程更新 UI
        updateUI()
    }
}
```

## 协程是如何实现挂起和恢复的？底层原理知道多少？

协程挂起和恢复，核心是`suspend+Continuation`构建的可挂起、可恢复的状态机模型，线程切换由`Dispatcher`控制，协程本身不占线程，执行逻辑可以被调度器迁移到任意线程执行  

通俗点来说，Kotlin协程是一个“自带快照能力的迷你线程”，当执行的supend时：  
- 自动保存当前状态（局部变量、resume点）
- 主动暂停，控制权交给调度器  
- 等待外部信号（如网络返回）时恢复  
- 在目标线程中执行恢复逻辑，从中断点继续走  

```kotlin
suspend fun foo() {
    println("A")
    delay(1000)
    println("B")
}

// 编译器转换
fun foo(continuation: Continuation<Unit>): Any {
    when (continuation.state) {
        0 -> {
            println("A")
            continuation.state = 1
            return delay(1000, continuation) // 暂停点：返回到调用者
        }
        1 -> {
            println("B")
            return Unit
        }
    }
}
```

## 协程和线程、线程池之间的关系？是如何调度执行的？

协程是运行在线程之上的轻量执行单元，借助协程调度器 Dispatcher 将挂起/恢复逻辑映射到底层线程池的线程中执行，从而避免线程阻塞并提高并发效率  

```kotlin
runBlocking {
    launch(Dispatchers.IO) {
        // 1. 从线程池中取出线程A
        val result = fetchFromNetwork() // delay 挂起
        // 2. delay挂起，线程A空闲，调度器回收线程
        // 3. 1秒后，线程池中某个线程（可能是线程B）恢复协程执行
        println(result)
    }
}

```
实际流程：
- `launch(Dispatchers.IO)`请求协程调度到IO线程池  
- 调度器从线程池中找一个可用线程，启动协程  
- 如果遇到挂起点（如delay），挂起当前协程，线程释放  
- 一段时间后，调度器选择一个线程恢复挂起的协程  
- 恢复后，协程从挂起点继续执行

## 你了解 Kotlin 的协程是基于什么实现的？是语言层面的魔法还是库层实现？

协程既有编译器层面的语言魔法，也有库层面的支持，两者缺一不可  

kotlin = kt编译器 + Kotlinx.coroutines

# ⚙️ 协程调度与上下文

## CoroutineDispatcher 有哪些常用实现？它们适合什么场景？

`CoroutineDispatcher`是Kotlin协程的调度核心，它决定了协程在哪个线程上运行  

1. `Dispatchers.Default`  

- 默认调度器，协程不指定 dispatcher 就用它
- 适合 CPU 密集型 工作，比如排序、复杂算法、图片处理
- 底层使用固定大小的线程池，线程数 = CPU 核数（最小2，最大CPU数）

2. `Dispatchers.IO`  

- 专为 IO密集型任务 设计（数据库、网络、文件读写）
- 底层使用一个 可扩展线程池（最多 64 个线程）
- 会将阻塞线程的任务从 Default 迁移到 IO，防止 CPU 饥饿  

3. `Dispatchers.Main`  

- Android专属，用于UI主线程，不能执行耗时任务
```xml
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:版本号") 
```

4. `Dispatchers.Unconfined`  

- 不指定调度器，在当前线程立即执行  
- 如果协程遇到挂起点，恢复后会在挂起点线程继续执行  
- 常用于测试环境  

5. 自定义Dispatcher  

```kotlin
val myDispatcher = Executors.newFixedThreadPool(3).asCoroutineDispatcher()

launch(myDispatcher) {
    // 运行在你自定义的线程池
}
```

## 你如何将协程切换到主线程？协程是怎么在多线程间切换上下文的？

Kotlin 协程通过withContext和Dispatcher控制线程切换，利用挂起恢复机制，在多个线程间非阻塞迁移上下文，实现了比回调更优雅的多线程协作  

```kotlin
suspend fun loadData() {
    withContext(Dispatchers.IO) {
        val data = fetchDataFromNetwork() // 在 IO 线程执行
        // … 数据加载完成
    }

    withContext(Dispatchers.Main) {
        // 回到主线程更新 UI
        textView.text = "Loaded: $data"
    }
}
```

以`withContext(Dispatchers.Main)`为例，协程的线程切换流程如下：  

- 挂起当前协程，保存挂起状态  
- 将当前的Continuation封装成一个任务ResumeRunnable  
- 投递到主线程对应的调度器队列中（如Handler）  
- 主线程执行该任务，恢复协程执行  

## 什么是 CoroutineContext？你是否自定义过上下文元素？  

`CoroutineContext`是一个Map-like协程“环境容器”，可以内建多个元素（调度器、Job、异常处理），也支持自定义元素用于跨协程携带上下文数据，功能类似ThreadLocal  

```kotlin
interface CoroutineContext {
    operator fun <E : Element> get(key: Key<E>): E?
    interface Key<E : Element>
    interface Element : CoroutineContext {
        val key: Key<*>
    }
}
```

看起来有点抽象，实际上CoroutineContext有一个扩展子类叫CombinedContext，本质上是个单链表 

```kotlin
// 简化版代码
class CombinedContext(
    private val left: CoroutineContext,
    private val element: Element
) : CoroutineContext {

    override fun <E : Element> get(key: Key<E>): E? {
        // 先查当前 element
        val e = element.get(key)
        if (e != null) return e
        // 没找到就往左递归查找
        return left[key]
    }
}

// 实际举例
val ctx = CoroutineName("Jason") + Dispatchers.IO + Job()

// 实际是：
val ctx = CombinedContext(
    left = CombinedContext(
        left = CombinedContext(
            left = EmptyCoroutineContext,
            element = CoroutineName("Jason")
        ),
        element = Dispatchers.IO
    ),
    element = Job()
)
```

使用举例：

```kotlin
val ctx = Dispatchers.IO + CoroutineName("JasonJob") + CoroutineExceptionHandler { _, e ->
    println("Caught exception: $e")
}

val scope = CoroutineScope(ctx)

scope.launch {
    println("My coroutine name: ${coroutineContext[CoroutineName]}")
}
```  

### 常见`CoroutineContext`元素  

| 元素类型                             | 作用        | 示例                                  |
| -------------------------------- | --------- | ----------------------------------- |
| `CoroutineDispatcher`            | 指定线程/线程池  | `Dispatchers.IO`、`Dispatchers.Main` |
| `Job`                            | 协程的生命周期控制 | 父子协程取消、结构化并发                        |
| `CoroutineName`                  | 协程命名（调试用） | `CoroutineName("MyTask")`           |
| `CoroutineExceptionHandler`      | 协程异常处理    | 捕获未捕获异常                             |
| 你自定义的 `CoroutineContext.Element` | 附加数据或行为   | 比如埋点 ID、日志追踪链路等                     |


### 自定义`CoroutineContext.Element`  

实现CoroutineContext.Element需要重写key成员  

假设要在协程环境里携带一个traceId:

```kotlin
data class TraceContext(val traceId: String) : CoroutineContext.Element {
    companion object Key : CoroutineContext.Key<TraceContext>
    override val key: CoroutineContext.Key<*> = Key
}

// 使用
val ctx = Dispatchers.Default + TraceContext("trace-1234")

val scope = CoroutineScope(ctx)

scope.launch {
    val trace = coroutineContext[TraceContext]
    println("Trace ID: ${trace?.traceId}")
}
```

## withContext() 和 launch() 有什么区别？用法场景如何区分？  

最大区别是launch会新建一个协程，形成多协程效果，原有的协程与新建的协程并发执行，而withContext只会切换当前协程调度器，不会新建协程，所在协程代码执行顺序不会改变  

举个例子——
对于launch：
```kotlin
fun main() = runBlocking {
    launch {
        delay(1000)
        println("launch 完成")
    }
    println("主协程继续")
}
```
执行结果：
```txt
主协程继续
launch 完成
```
对于withContext：
```kotlin
fun main() = runBlocking {
    withContext(Dispatchers.IO) {
        delay(1000)
        println("withContext 完成")
    }
    println("主协程继续")
}
```
执行结果：
```txt
withContext 完成
主协程继续
```

从源码的角度来讲，launch是调用了`CoroutineScope.launch`，新建一个CoroutineImpl子类，调度器接管  
```kotlin
public fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
): Job {
    val newContext = newCoroutineContext(context)
    val coroutine = if (start.isLazy)
        LazyStandaloneCoroutine(newContext, block) else
        StandaloneCoroutine(newContext, active = true)
    coroutine.start(start, coroutine, block)
    return coroutine
}
```

withContext本质上是个supend函数, 调用后挂起当前协程，切换CoroutineContext，运行block，最后切换回来恢复上下文  

```kotlin
// 简化版withContext
suspend fun <T> withContext(
    newContext: CoroutineContext,
    block: suspend () -> T
): T {
    // 获取当前协程上下文（包括 Job、Dispatcher、CoroutineName 等）
    val currentContext = coroutineContext

    // 合并上下文（相当于 current + new）
    val combined = currentContext + newContext

    // 如果没变，就直接执行 block
    if (newContext === EmptyCoroutineContext) {
        return block()
    }

    // 否则挂起当前协程，调度执行 block
    return suspendCoroutine { continuation ->
        // 使用新的 Dispatcher 启动任务
        combined[ContinuationInterceptor]?.dispatch(
            context = combined,
            block = Runnable {
                try {
                    val result = block()
                    continuation.resume(result)
                } catch (e: Throwable) {
                    continuation.resumeWithException(e)
                }
            }
        )
    }
}

```

# 🔁 协程构建器

## launch、async、runBlocking 有什么区别？使用场景？

| 构建器           | 返回值           | 是否阻塞线程  | 是否有返回值 | 用途              |
| ------------- | ------------- | ------- | ------ | --------------- |
| `launch`      | `Job`         | ❌非阻塞    | ❌无返回值  | 启动后台任务          |
| `async`       | `Deferred<T>` | ❌非阻塞    | ✅有返回值  | 并发计算、需要结果的任务    |
| `runBlocking` | `T`（泛型）       | ✅阻塞当前线程 | ✅有返回值  | 桥接阻塞与协程，仅用于入口场景 |

## runBlocking 在 Android 中能否用？什么时候用它是合理的？

runBlocking会阻塞主线程，所以不能在Android主线程中使用，但可以用于子线程环境  

## 协程作用域`CoroutineScope`是什么？为什么建议在适当作用域中启动协程？

`CoroutineScope`是协程的生命周期管理者，它定义了启动协程的上下文环境（所在线程、调度器）、协程何时会被取消（与Job绑定）、子协程如何传播取消事件（结构化并发核心）  

1. 生命周期管理，防止内存泄漏  

如果随意使用`GlobalScope.launch`启动协程，除非手动取消，否则会一直运行，容易导致内存泄漏，所以需要结合具体场景使用协程，比如ViewModel、Lifecycle 
```kotlin
viewModelScope.launch {
    fetchData()
}
```
当ViewModel被销毁的时候，协程也会被取消  

2. CoroutineScope支持结构化并发  

一个作用域管理的所有子协程，父协程取消，所有子协程都会被取消

```kotlin
//runBlocking取消，子协程都会取消
runBlocking {
    launch {
        // 子协程1
    }
    launch {
        // 子协程2
    }
}
```

3. 异常传播  

结构化作用域中，所有子协程的异常都可以向上传播给父协程，方便统一处理  

## 你了解 supervisorScope 和 coroutineScope 的区别吗？

最大的区别在于子协程异常传播处理上：supervisorScope子协程异常不会相互影响，coroutinescope会影响彼此  

```kotlin
suspend fun testCoroutineScope() = coroutineScope {
    launch {
        delay(100)
        throw RuntimeException("Child A failed")
    }

    launch {
        delay(1000)
        println("Child B finished")  // ❌永远不会执行
    }
}

suspend fun testSupervisorScope() = supervisorScope {
    launch {
        delay(100)
        throw RuntimeException("Child A failed")
    }

    launch {
        delay(1000)
        println("Child B finished")  // ✅依然可以正常完成
    }
}
```

# 🔐 异常处理与结构化并发

## 协程中的异常是如何传播的？如何捕获？

1. launch启动协程：异常默认向上传播  

如果没有try-catch，传给父协程（作用域）  
如果没有父协程，比如使用了GlobalScope，传给CoroutineExceptionHandler，如果Handler未处理，程序直接崩溃  

2. async启动的协程：异常被封装Deferred，await时抛出  

3. 所有协程异常都可以通过try-catch捕获  

4. launch异常可以通过CoroutineExceptionHandler捕获  

## 什么是结构化并发（Structured Concurrency）？有什么优点？

结构化并发可以理解为：让协程像普通函数、变量一样，受作用域控制，避免资源泄漏，错误可控 

```kotlin
// ViewModelScope结构化管理协程
viewModelScope.launch {
    val user = async { loadUser() }
    val posts = async { loadPosts() }
    show(user.await(), posts.await())
}

// 网络请求聚合
suspend fun loadData() = coroutineScope {
    val a = async { fetchA() }
    val b = async { fetchB() }

    combine(a.await(), b.await())
}
```

你了解 SupervisorJob 和 Job 的区别吗？何时用它？

CoroutineExceptionHandler 是如何工作的？在哪些场景有效？

# 🔄 挂起函数与取消机制
如何取消协程？Job.cancel() 的原理是什么？

isActive、ensureActive()、yield() 这些函数你用过吗？各自作用？

什么是 cooperative cancellation？为什么它重要？

挂起函数执行过程中如果不检查取消，会发生什么？如何改进？

# 📱 与 Android 框架结合
你在项目中是如何使用协程的？是否与 ViewModel、LiveData、Flow 结合使用？

viewModelScope 和 lifecycleScope 分别是什么？为何推荐使用它们？

你用过 LiveData + Coroutine 的组合吗？如何处理生命周期？

协程在 Activity 被销毁后仍在运行，会发生什么问题？如何避免？

# 🧪 实践场景题（带思维能力考察）
你有一个网络请求 + 本地缓存的场景，怎么用协程实现？需要考虑并发和取消。

如何用 async 并发执行两个耗时任务，并收集它们的结果？

启动多个协程，如果其中某个失败，其他协程怎么办？如何控制它们的取消策略？

怎么写一个能自动超时的协程操作？你用过 withTimeout() 吗？

# 💥 陷阱与高阶
协程为什么可能导致内存泄漏？你遇到过吗？

为什么说协程虽然轻量，但不是免费线程？过度使用有什么风险？

协程和 Flow 配合时，怎么处理 backpressure（背压）？

你是否实现过自己的 suspend 函数？怎么让一个普通回调函数变成挂起函数？