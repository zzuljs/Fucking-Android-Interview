# Kotlin基础

1 Kotlin 与 Java 的主要语法差异有哪些？  
2 Kotlin 中有哪些基础类型？和 Java 有何不同？  
3 `val` 和 `var` 有什么区别？  
4 Kotlin 如何处理可空类型？  
5 Kotlin空安全实现原理  
6 `is` 和 `as` 用来做什么？  
7 Kotlin 如何进行字符串模板？  
8 Kotlin 数组和集合有什么区别？  
9 Kotlin 的类型系统支持关系类型吗？  
10 Kotlin 支持委托机制吗？  
11 Kotlin 中的关键字 inline / crossinline / noinline 有什么用？  
12 Kotlin 中类的默认修饰符是什么？  
13 Kotlin 支持多类继承吗？  
14 Kotlin 中如何定义抽象类和接口？  
15 什么是 data class？能自动生成什么？  
16 什么是 sealed class？适合什么场景？  
17 Kotlin 如何定义 singleton？  
18 Kotlin 中的 companion object 有什么用？  
19 Kotlin 如何实现方法重写？  
20 Kotlin 中是否支持内部类？  
21 Kotlin 是否支持方法重载（函数重载）？  
22 什么是Kotlin逆变、协变  
23 为什么协变限制写入，逆变限制读取  
24 Kotlin 中什么是高阶函数？  
25 Kotlin 中如何定义 Lambda 表达式？  
26 Kotlin 中的匿名函数与 Lambda 有什么区别？  
27 Kotlin 的闭包特性是什么？  
28 Kotlin 中的 `with`、`let`、`apply`、`run`、`also` 有何区别？  
29 Kotlin 如何对集合进行函数式操作？  
30 `takeIf` 和 `takeUnless` 的作用是什么？  
31 什么是尾递归优化（tailrec）？  
32 Kotlin 如何支持函数类型声明？  
33 Kotlin 函数默认参数和命名参数如何使用？  
34 Kotlin协程和Java线程的区别  
35 协程和线程使用场景  
36 Kotlin 协程的核心概念有哪些？  
37 协程与线程的主要区别？  
38 什么是结构化并发？优点是什么？  
39 `launch`、`async`、`runBlocking` 有什么区别？  
40 `withContext` 和 `launch` 的区别？  
41 如何取消协程？  
42 什么是 `SupervisorJob`？与普通 `Job` 有何区别？  
43 如何处理协程中的异常？  
44 `suspend` 修饰函数能否独立调用？  
45 如何优雅地做协程任务的并发控制？  
46 什么是 Flow？与传统协程有何区别？  
47 Flow 的基本使用方式有哪些？  
48 Flow 的异常处理方式？  
49 Flow 默认在哪个线程执行？如何切换上下文？  
50 冷流（cold flow）与热流（hot flow）的区别？  
51 什么是 SharedFlow 和 StateFlow？它们的区别？  
52 如何取消 Flow 的收集？  
53 Flow 的背压策略如何控制？  
54 什么是 Channel？与 Flow 的区别？  
55 如何用 Channel 实现并发消息处理？  
56 什么是 ViewModelScope？为何要使用它？  
57 如何在 Activity / Fragment 中感知协程生命周期？  
58 如何使用 Kotlin Flow 结合 LiveData？  
59 在 Jetpack Compose 中如何使用协程与 Flow？  
60 Android 中为何 prefer Flow over LiveData？  
61 Kotlin 协程 + Retrofit 请求方式是怎样的？  
62 如何在 Android 中优雅处理协程异常？  
63 Jetpack Compose 中如何实现 UI 与 State 解耦？  
64 协程在 Android 中的常见坑有哪些？  
65 Kotlin DSL 在 Android 中有哪些应用？  
66 Kotlin 如何调用 Java 代码？  
67 Java 如何调用 Kotlin？有哪些限制？  
68 Kotlin 如何处理 Java 的 null 值？  
69 Kotlin 如何兼容 Java 的 SAM 接口？  
70 Kotlin 中的 `@JvmStatic`、`@JvmField`、`@JvmOverloads` 有什么作用？  
71 如何使用 Kotlin 写工具类供 Java 调用？  
72 Kotlin 中 data class 转换为 Java 会生成什么？  
73 Java 的枚举与 Kotlin 的 `enum class`、`sealed class` 有何差异？  
74 Kotlin 的协程如何从 Java 调用？  
75 Kotlin 写的 API 对 Java 开发者的友好建议？  

# Kotlin协程 Coroutine

1 什么是 Kotlin 协程？基本使用方式有哪些？它与线程有什么区别？  
2 什么是挂起函数（suspend function）？它和普通函数的本质区别是什么？  
3 协程是如何实现挂起和恢复的？底层原理知道多少？  
4 协程和线程、线程池之间的关系？是如何调度执行的？  
5 async、await、join的区别是什么？  
6 Job、Deferred区别是什么？  
7 你了解 Kotlin 的协程是基于什么实现的？是语言层面的魔法还是库层实现？  
8 CoroutineDispatcher 有哪些常用实现？它们适合什么场景？  
9 你如何将协程切换到主线程？协程是怎么在多线程间切换上下文的？  
10 什么是 CoroutineContext？你是否自定义过上下文元素？  
11 withContext() 和 launch() 有什么区别？用法场景如何区分？  
12 launch、async、runBlocking 有什么区别？使用场景？  
13 runBlocking 在 Android 中能否用？什么时候用它是合理的？  
14 协程作用域`CoroutineScope`是什么？为什么建议在适当作用域中启动协程？  
15 你了解 supervisorScope 和 coroutineScope 的区别吗？  
16 协程中的异常是如何传播的？如何捕获？  
17 什么是结构化并发（Structured Concurrency）？有什么优点？  
18 你了解 SupervisorJob 和 Job 的区别吗？何时用它？  
19 CoroutineExceptionHandler 是如何工作的？在哪些场景有效？  
20 如何取消协程？Job.cancel() 的原理是什么？  
21 isActive、ensureActive()、yield() 这些函数你用过吗？各自作用？  
22 挂起函数执行过程中如果不检查取消，会发生什么？如何改进？  
23 你在项目中是如何使用协程的？是否与 ViewModel、LiveData、Flow 结合使用？  
24 viewModelScope 和 lifecycleScope 分别是什么？为何推荐使用它们？  
25 你用过 LiveData + Coroutine 的组合吗？如何处理生命周期？  
26 协程在 Activity 被销毁后仍在运行，会发生什么问题？如何避免？  
27 你有一个网络请求 + 本地缓存的场景，怎么用协程实现？需要考虑并发和取消。  
28 如何用 async 并发执行两个耗时任务，并收集它们的结果？  
29 启动多个协程，如果其中某个失败，其他协程怎么办？如何控制它们的取消策略？  
30 怎么写一个能自动超时的协程操作？你用过 withTimeout() 吗？  
31 协程为什么可能导致内存泄漏？你遇到过吗？  
32 为什么说协程虽然轻量，但不是免费线程？过度使用有什么风险？  