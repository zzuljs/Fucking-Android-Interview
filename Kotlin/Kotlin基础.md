# 🧱 一、基础语法与类型系统

## 1 Kotlin 与 Java 的主要语法差异有哪些？

* Kotlin 支持简洁语法，非必写类型、分号等
* 支持定义高级函数（高阶、拓展等）
* 实现维度类型安全，比如可空性检查
* 数据类、密封类，单例、委托等高级特性

## 2 Kotlin 中有哪些基础类型？和 Java 有何不同？

* Int, Long, Short, Byte, Float, Double, Boolean, Char, String
* Kotlin 中的基本类型是对象，非 JVM 基本类型，通过 boxing/unboxing 和编译器优化处理

## 3 `val` 和 `var` 有什么区别？

* `val`：一次性分配，即 final 类型值
* `var`：可复值的变量

## 4 Kotlin 如何处理可空类型？

* 通过在类型后加 `?`，表示可为 null
* `?.` 表示安全调用，`!!` 表示确定不为 null
* `?:` 表示后处理或默认值（Elvis 操作符）
* `let` 配合 `?.` 可实现类似 Optional 风格的安全操作

## 5 Kotlin空安全实现原理

Kotlin空安全是一种类型系统机制，通过显式地区分可空类型和非空类型，在编译阶段预防空指针异常（NullPointerException, NPE）

通过编译器类型检查和字节码注入空值检查指令（CheckNotNull），在代码逻辑违反空安全规则时抛出异常。
Kotlin空安全会引入运行时开销，但可以忽略不计。

与Java `@Nullable` 最大的不同，Java运行时仍需手动判空。

## 6 `is` 和 `as` 用来做什么？

* `is` 用于类型检查，支持智能类型推断
* `as` 是类型强制转换，`as?` 是安全转换（失败时返回 null）

## 7 Kotlin 如何进行字符串模板？

* 直接使用 `"Hello, $name!"`，可以用括号 `\${}` 进行补充操作

## 8 Kotlin 数组和集合有什么区别？

* 数组是定长序列，通过 `Array<T>` 创建
* 集合有 List/Set/Map 等类型，分为 immutable (`listOf`) 和 mutable (`mutableListOf`)
* 集合提供多种便捷操作函数，如 map/filter/reduce 等

## 9 Kotlin 的类型系统支持关系类型吗？

* 支持「关系类型」模型：

  * 通过 `in` / `out` 关键字实现「变量定义时的协、逆、无关」性
  * `out T` 表示协变（只能输出），`in T` 表示逆变（只能输入）
* 例如：`List<out Number>` 表示可以是 List<Int> 或 List<Double>

## 10 Kotlin 支持委托机制吗？

* 是的，Kotlin 支持两种委托：

  * **属性委托：** `by` 关键字实现 lazy 、ObservableProperty 等
  * **类委托：** `class X: Y by z` 表示将接口 Y 的实现委托给对象 z

## 11 Kotlin 中的关键字 inline / crossinline / noinline 有什么用？

* `inline`：转换为相当于直接代码应用，降低间接负担
* `noinline`：用于阻止 inline 函数中某些参数被 inline
* `crossinline`：阻止 lambda 中使用 return 打断外部函数

---

# 👮‍♂️ 二、面向对象特性（OOP）

## 1 Kotlin 中类的默认修饰符是什么？

* 默认是 `public final`，即 **公共且不可被继承**，需要加 `open` 才可被继承

## 2 Kotlin 支持多类继承吗？

* 不支持类多继承，**支持接口多继承**；多接口冲突需要显式指明

## 3 Kotlin 中如何定义抽象类和接口？

* 抽象类：`abstract class A`，可包含抽象和实现方法
* 接口：`interface B`，支持方法默认实现，但不支持字段

## 4 什么是 data class？能自动生成什么？

* 数据类，通过 `data class User(val name: String)` 定义
* 生成 `toString()` 、`equals()` 、`hashCode()` 、`copy()` 、`componentN()`

## 5 什么是 sealed class？适合什么场景？

* 密封类，通过 `sealed class Result` 定义
* 限制继承类只能在同一文件内，适合 `when` 全型别分析

## 6 Kotlin 如何定义 singleton？

* 使用 `object` 关键字：`object Singleton`，在 JVM 下是 thread-safe 单例

## 7 Kotlin 中的 companion object 有什么用？

* 类的伴生对象，用于实现非 static 的静态功能
* 可以定义名称，也可实现接口，适用于 Factory 等场景

## 8 Kotlin 如何实现方法重写？

* 需要先将类和方法标记为 `open`，实现继承类使用 `override` 修饰

## 9 Kotlin 中是否支持内部类？

* 是的，用 `inner class` 表示可访问外部类属性，默认为 static nested class

## 10 Kotlin 是否支持方法重载（函数重载）？

* 是的，支持同名方法持有不同参数（类型/长度）的 overload

## 11 什么是Kotlin逆变、协变

Kotlin逆变、协变是泛型编程概念，定义了父类和子类之间两种不同的类型替换。
协变：关键字 `out`，通配符 `? extends`，表示子类可以替代父类类型，仅用于读的场景，例如：

```kotlin
class Animal
class Cat: Animal

fun println(data: List<out Animal>) {
    for(e in data) {
        println(e)
    }
}

val data = arrayOfList(Cat(),Cat())
println(data)
```

常用场景：返回一个只读集合

逆变：关键字 `in`，通配符 `? super`，表示父类可以替换子类类型，仅用于写的场景，例如：

```kotlin
class Event
class ButtonEvent: Event

fun callback(e:List<in ButtonEvent>){
    e.add(Event())
}

val buttonEvent = listOf(Event())
callback(buttonEvent)
```

常用场景：事件监听、Comparator比较器

## 12 为什么协变限制写入，逆变限制读取

保证类型安全，对于协变，如果不限制写入，有可能导致类型不确定，比如：

```kotlin
val list = listOf<Cat>()
fun feedAnimal(a: List<out Animal>) {
    a.add(Dog()) // error
}
```

协变允许子类替代父类，但子类可能存在多种，如果允许写入，会导致该类型不确定，所以协变限制写，保障类型精确性
对于逆变，逆变允许父类取代子类，如果允许写，可能导致父类调用子类的方法，直接报错，所以逆变限制读，保障类型兼容性


---

# 🧬 三、函数式编程（Functional Programming）

## 1 Kotlin 中什么是高阶函数？

* 高阶函数是指以函数作为参数或返回值的函数，例如：

```kotlin
fun doSomething(action: () -> Unit) {
    action()
}
```

## 2 Kotlin 中如何定义 Lambda 表达式？

* 基本语法：`val sum = { a: Int, b: Int -> a + b }`
* 使用 `it` 表示单个参数时的默认名称

## 3 Kotlin 中的匿名函数与 Lambda 有什么区别？

* 匿名函数：`fun(x: Int): Int = x * x`
* 匿名函数可指定返回类型，可支持局部 return
* Lambda 中的 return 默认返回最近的函数（非局部 return），需配合 `crossinline` 限制

## 4 Kotlin 的闭包特性是什么？

* Lambda 表达式可以捕获外部作用域的变量形成闭包，即变量生命周期延长至函数对象

## 5 Kotlin 中的 `with`、`let`、`apply`、`run`、`also` 有何区别？

* `let`：传入 it，返回结果值，常用于 null 检查
* `run`：作为表达式块，返回结果值，作用对象为 this
* `apply`：初始化配置对象，返回对象本身
* `also`：与 apply 类似，但传入 it，常用于链式副作用
* `with`：非扩展函数，作用于指定对象，可更改上下文 this

## 6 Kotlin 如何对集合进行函数式操作？

* 集合支持 map、filter、flatMap、reduce、fold、groupBy 等操作

```kotlin
val result = listOf(1, 2, 3).map { it * 2 }.filter { it > 2 }
```

## 7 `takeIf` 和 `takeUnless` 的作用是什么？

* `takeIf { condition }`：如果满足条件则返回对象，否则 null
* `takeUnless { condition }`：与 takeIf 相反逻辑

## 8 什么是尾递归优化（tailrec）？

* Kotlin 支持 `tailrec` 修饰尾递归函数，编译器会将其优化为循环，避免栈溢出

```kotlin
tailrec fun factorial(n: Int, acc: Int = 1): Int =
    if (n <= 1) acc else factorial(n - 1, acc * n)
```

## 9 Kotlin 如何支持函数类型声明？

* 使用 `(参数类型) -> 返回类型` 语法定义，例如：`(Int, Int) -> Int`
* 可用于变量、参数、返回值类型

## 10 Kotlin 函数默认参数和命名参数如何使用？

* 支持为函数参数设置默认值
* 支持通过参数名指定值，提升可读性，支持跳过部分参数

```kotlin
fun greet(name: String = "Guest")

fun main() {
    greet()            // Guest
    greet(name = "Li") // Li
}
```

---

# 🔧 四、协程与并发（Coroutines & Concurrency）

## 1 Kotlin协程和Java线程的区别
- 协程是轻量级资源占用。Kotlin启动一个协程占用内存约2KB（动态增加），Java线程默认占用1MB栈内存（可以通过`-Xss`调整），单个进程支持的线程数只有几千（JVM特性决定），但是Kotlin可以轻松开启上万协程
- 更好的线程调度和切换。协程可以在没有操作系统内核介入的前提下实现执行权的调度，上下文切换仅保存程序计数器、栈指针和关键计数器，耗时小，执行比较快；Java线程依赖操作系统抢占式切换、强制中断线程，上下文切换需要保存完整线程资源（寄存器、堆栈、线程变量），耗时相对大
- 协程可以通过`suspend`非阻塞性挂起，挂起时释放底层线程资源，供其他协程复用；线程挂起是阻塞性操作，会占用线程资源，协程通过少量线程资源实现高并发IO操作，避免了资源浪费
- 结构化并发与生命周期管理。通过`CoroutineScope`实现结构化并发，父协程取消时，所有子协程自动取消，避免了内存泄漏；Java线程需要手动管理，容易内存泄漏。
- 协程通过`await/async`实现更好的异步操作，Java线程需要通过线程回调来实现切换，容易造成回调嵌套地狱，可读性差
- 更灵活的调用和线程池优化。Kotlin可以通过`Dispatchers.Main` `Dispatchers.IO`等精准控制协程执行的线程池，Java需要手动配置线程池，分配不当会导致资源争抢

## 2 协程和线程使用场景
- 协程适合高并发、网络请求、数据IO、UI异步更新、业务回调嵌套较多的场景
- 线程适合CPU密集（图像处理）、依赖`ThreadLocal`等Java组件、依赖直接控制线程优先级或者内核调度的场景

## 3 Kotlin 协程的核心概念有哪些？

* `suspend`：声明挂起函数，可在协程中挂起并恢复执行
* `CoroutineScope`：定义协程作用域，决定生命周期
* `Dispatcher`：指定协程运行的线程或线程池（如 `Dispatchers.IO`）
* `Job`：协程任务句柄，用于控制、取消、合并等

## 4 协程与线程的主要区别？

* 协程是轻量级线程，由 Kotlin 和底层调度器管理
* 启动成本更低，支持大量并发
* 非阻塞式挂起，节省资源，避免 ANR

## 5 什么是结构化并发？优点是什么？

* 子协程必须在父协程完成前结束，作用域一致
* 可通过 `CoroutineScope.launch` 保证协程不会泄漏
* 提高协程生命周期管理的可预测性与安全性

## 6 `launch`、`async`、`runBlocking` 有什么区别？

* `launch`：返回 `Job`，不关心返回值
* `async`：返回 `Deferred<T>`，需 `await()` 获取结果
* `runBlocking`：阻塞当前线程，常用于测试或 main 函数中

## 7 `withContext` 和 `launch` 的区别？

* `withContext` 是挂起函数，会切换上下文并等待结果返回
* `launch` 异步启动新协程，不阻塞调用方

## 8 如何取消协程？

* 通过 `job.cancel()` 取消协程
* 在协程中应定期检查 `isActive` 或使用 `ensureActive()` / `yield()`

## 9 什么是 `SupervisorJob`？与普通 `Job` 有何区别？

* `Job`：子协程异常会取消整个作用域
* `SupervisorJob`：子协程失败不会影响其他兄弟协程
* 常用于 `ViewModelScope` 等场景

## 10 如何处理协程中的异常？

* 使用 `try-catch` 捕获异常
* 或使用 `CoroutineExceptionHandler` 捕获未处理异常

```kotlin
val handler = CoroutineExceptionHandler { _, exception -> println(exception) }
CoroutineScope(Dispatchers.IO + handler).launch { throw IOException() }
```

## 11 `suspend` 修饰函数能否独立调用？

* 否，必须在协程作用域或另一个挂起函数中调用
* 不能直接在主线程或普通函数中调用

## 12 如何优雅地做协程任务的并发控制？

* 使用 `async` 并配合 `awaitAll()`
* 使用 `coroutineScope { }` 组合多个挂起任务
* 使用 `Mutex` 或 `Channel` 实现并发协作

---

# 🌊 五、Flow、Channel 与响应式流

## 1 什么是 Flow？与传统协程有何区别？

* Flow 是 Kotlin 提供的异步数据流，用于按顺序发射多个值（冷流）
* 与 suspend 返回单个值不同，Flow 可发射多个值
* Flow 是惰性执行的，仅在被 collect 时才启动

## 2 Flow 的基本使用方式有哪些？

* 创建 Flow：`flow { emit(value) }`
* 收集 Flow：`flow.collect { value -> ... }`
* 常用操作符：`map`、`filter`、`take`、`onEach`、`collect` 等

## 3 Flow 的异常处理方式？

* 使用 `catch {}` 操作符进行异常捕获
* 使用 `onCompletion {}` 无论成功或失败后都可触发

```kotlin
flow {
    emit(1)
    throw RuntimeException("error")
}.catch { e -> emit(-1) }
 .collect { println(it) }
```

## 4 Flow 默认在哪个线程执行？如何切换上下文？

* 默认在调用者协程上下文中运行
* 可使用 `flowOn(Dispatchers.IO)` 切换发射上下文（非收集）

## 5 冷流（cold flow）与热流（hot flow）的区别？

* 冷流：每次 `collect` 都重新执行发射逻辑（如 Flow）
* 热流：始终在运行，collect 是监听其当前状态（如 SharedFlow、StateFlow）

## 6 什么是 SharedFlow 和 StateFlow？它们的区别？

* SharedFlow：无状态热流，支持多个 collector
* StateFlow：有状态热流，始终保存最新值（类似 LiveData）
* 区别：StateFlow 有 `value` 属性和初始值，SharedFlow 可自定义 replay 缓存

## 7 如何取消 Flow 的收集？

* 通过协程作用域取消，如 `viewModelScope.cancel()`
* 或在 collect 中手动使用 `takeWhile {}` 控制

## 8 Flow 的背压策略如何控制？

* Flow 是按需消费的，不会主动推送（天然支持背压）
* 可使用 `buffer()`、`conflate()`、`collectLatest()` 控制流速与丢弃策略

## 9 什么是 Channel？与 Flow 的区别？

* Channel 是协程中的热流，支持双向通信（发送/接收）
* 通常用于生产者-消费者模式
* Flow 是声明式、响应式数据流，适合单向链式处理

## 10 如何用 Channel 实现并发消息处理？

```kotlin
val channel = Channel<Int>()
CoroutineScope(Dispatchers.Default).launch {
    for (msg in channel) println("Received $msg")
}
channel.send(1)
channel.send(2)
channel.close()
```

---

# 📱 六、Kotlin 在 Android 开发中的应用

## 1 什么是 ViewModelScope？为何要使用它？

* ViewModelScope 是 Android Jetpack 提供的协程作用域，用于 ViewModel 生命周期范围内启动协程
* 可自动在 ViewModel 销毁时取消协程，避免内存泄漏

```kotlin
viewModelScope.launch {
    val data = repository.loadData()
    _state.value = data
}
```

## 2 如何在 Activity / Fragment 中感知协程生命周期？

* 使用 lifecycleScope（绑定到生命周期）
* 使用 repeatOnLifecycle / lifecycle.repeatWhenActive 块收集 Flow 更加安全

```kotlin
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.stateFlow.collect { render(it) }
    }
}
```

## 3 如何使用 Kotlin Flow 结合 LiveData？

* 使用 `asLiveData()` 将 Flow 转换为 LiveData

```kotlin
val liveData = myFlow.asLiveData()
```

* 或使用 `liveData { emitSource(...) }` 构建响应式数据流

## 4 在 Jetpack Compose 中如何使用协程与 Flow？

* 使用 `LaunchedEffect` 启动协程
* 使用 `collectAsState()` 从 StateFlow 中获取状态

```kotlin
val uiState by viewModel.stateFlow.collectAsState()
LaunchedEffect(Unit) {
    viewModel.loadData()
}
```

## 5 Android 中为何 prefer Flow over LiveData？

* Flow 支持更多操作符，具有挂起能力，背压支持更好
* 可用于 Repository 层、数据转换链路
* 结合生命周期的收集方式更灵活（repeatOnLifecycle）

## 6 Kotlin 协程 + Retrofit 请求方式是怎样的？

* Retrofit 接口方法支持 `suspend` 修饰
* 可结合 ViewModelScope、错误处理、Flow 转换等使用

```kotlin
interface ApiService {
    @GET("/users")
    suspend fun getUsers(): List<User>
}
```

## 7 如何在 Android 中优雅处理协程异常？

* ViewModelScope 中添加 `CoroutineExceptionHandler`
* Flow 中使用 `catch {}` 和 `onCompletion {}` 处理

## 8 Jetpack Compose 中如何实现 UI 与 State 解耦？

* 将状态托管至 ViewModel 中的 `StateFlow`
* 使用 `collectAsState()` 实时更新 UI，遵循单向数据流（Unidirectional Data Flow）

## 9 协程在 Android 中的常见坑有哪些？

* 忘记取消协程导致内存泄漏（未使用正确作用域）
* 在 UI 线程执行阻塞操作（未使用 IO Dispatcher）
* 使用 ViewModelScope 之外的作用域处理数据回调

## 10 Kotlin DSL 在 Android 中有哪些应用？

* Gradle 构建脚本（build.gradle.kts）
* Jetpack Compose UI 代码本身即是 DSL
* Anko（已废弃）等库也曾使用 Kotlin DSL 构建 UI

---

# 🔁 七、Kotlin 与 Java 的互操作性

## 1 Kotlin 如何调用 Java 代码？

* Kotlin 可以无缝调用 Java 类、方法、字段，无需额外配置
* 自动映射 getter/setter、重载方法等

## 2 Java 如何调用 Kotlin？有哪些限制？

* Kotlin 编译器会生成 JVM 字节码，Java 可调用生成的方法
* 可用 `@JvmName`、`@JvmStatic`、`@JvmOverloads` 提供兼容性
* 注意 Kotlin 顶层函数、对象、默认参数需要特殊处理

## 3 Kotlin 如何处理 Java 的 null 值？

* Java 类型在 Kotlin 中为平台类型（T!），需开发者自己管理 null 安全
* 可用 `@Nullable`、`@NotNull` 注解帮助推断

## 4 Kotlin 如何兼容 Java 的 SAM 接口？

* Kotlin 支持 SAM 转换，可将 Lambda 传给 Java 函数式接口

```kotlin
val thread = Thread { println("Hello from thread") }
```

## 5 Kotlin 中的 `@JvmStatic`、`@JvmField`、`@JvmOverloads` 有什么作用？

* `@JvmStatic`：将伴生对象的方法编译为静态方法供 Java 调用
* `@JvmField`：避免生成 getter/setter，暴露字段本身
* `@JvmOverloads`：生成多个重载版本支持 Java 调用 Kotlin 的默认参数

## 6 如何使用 Kotlin 写工具类供 Java 调用？

* 建议写为 `object` 或顶层函数 + `@JvmStatic`

```kotlin
object Utils {
    @JvmStatic
    fun toast(msg: String) { ... }
}
```

## 7 Kotlin 中 data class 转换为 Java 会生成什么？

* 会生成 `equals()`、`hashCode()`、`toString()`、`copy()`、组件函数（`componentN`）等
* Java 中可使用 `getXXX()` 调用属性

## 8 Java 的枚举与 Kotlin 的 `enum class`、`sealed class` 有何差异？

* `enum class`：具备固定常量列表
* `sealed class`：可携带数据，支持模式匹配（`when`），更灵活

## 9 Kotlin 的协程如何从 Java 调用？

* 可通过 `GlobalScope.launch {}` 启动协程
* 更推荐在 Java 中使用挂起函数封装为 callback 或 future 接口

## 10 Kotlin 写的 API 对 Java 开发者的友好建议？

* 合理使用 `@JvmXxx` 注解提升可调用性
* 避免默认参数、伴生对象暴露过多内容
* 提供纯 Java 接口包装类以简化使用

---

# ⚠️ 八、Kotlin 常见语法陷阱与最佳实践

## 1 忘记处理空安全问题导致 NPE

* Kotlin 默认非空，使用 Java 平台类型或 `!!` 时需特别小心
* 尽量避免 `!!` 运算符，使用 `?.`、`?:`、`let` 更安全

## 2 使用 lateinit 却未初始化导致崩溃

* `lateinit` 不能用于原始类型，使用前必须确保初始化
* 考虑改用 `by lazy` 或 `var? = null`

## 3 默认参数在 Java 调用中无法兼容

* 使用 `@JvmOverloads` 提供多个重载版本给 Java 调用

## 4 Lambda 中 return 的非预期行为

* `return` 默认是非局部返回，可能提前退出外部函数
* 如需局部返回，使用 `return@label` 明确指定

## 5 惰性初始化不当造成线程安全问题

* `by lazy` 默认是线程安全的（`LazyThreadSafetyMode.SYNCHRONIZED`）
* 如对性能有要求可设置为 `LazyThreadSafetyMode.NONE`

## 6 避免滥用扩展函数

* 扩展函数并不会真正修改类本身
* 不要用扩展函数替代真正需要继承或多态的地方

## 7 可变集合传递给不可信代码

* Kotlin 默认集合是不可变（只读视图），`MutableList` 需谨慎传递
* 尽量使用接口类型如 `List`、`Set` 等

## 8 data class 中包含引用类型属性的 `copy()` 使用风险

* `copy()` 只是浅拷贝，深层对象仍共享引用
* 如涉及深层结构，需手动实现复制逻辑

## 9 类型擦除带来的泛型陷阱

* Kotlin 泛型在运行时被擦除，`T::class` 不可直接使用
* 使用 `reified` + `inline` 保留泛型类型信息

```kotlin
inline fun <reified T> Gson.fromJson(json: String): T {
    return this.fromJson(json, T::class.java)
}
```

## 10 忽略协程作用域管理导致内存泄漏

* 使用 `GlobalScope` 可能导致任务泄漏或无法取消
* 尽量绑定作用域到生命周期（如 ViewModelScope、lifecycleScope）
