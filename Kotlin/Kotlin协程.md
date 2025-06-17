☕ 基础原理类
什么是 Kotlin 协程？它与线程有什么区别？

什么是挂起函数（suspend function）？它和普通函数的本质区别是什么？

协程是如何实现挂起和恢复的？底层原理知道多少？

协程和线程、线程池之间的关系？是如何调度执行的？

你了解 Kotlin 的协程是基于什么实现的？是语言层面的魔法还是库层实现？

⚙️ 协程调度与上下文
CoroutineDispatcher 有哪些常用实现？它们适合什么场景？

Dispatchers.IO、Dispatchers.Default 和 Dispatchers.Main 有什么区别？

你如何将协程切换到主线程？协程是怎么在多线程间切换上下文的？

什么是 CoroutineContext？你是否自定义过上下文元素？

withContext() 和 launch() 有什么区别？用法场景如何区分？

🔁 协程构建器
launch、async、runBlocking 有什么区别？使用场景？

runBlocking 在 Android 中能否用？什么时候用它是合理的？

协程作用域（CoroutineScope）是什么？为什么建议在适当作用域中启动协程？

你了解 supervisorScope 和 coroutineScope 的区别吗？

🔐 异常处理与结构化并发
协程中的异常是如何传播的？如何捕获？

什么是结构化并发（Structured Concurrency）？有什么优点？

你了解 SupervisorJob 和 Job 的区别吗？何时用它？

CoroutineExceptionHandler 是如何工作的？在哪些场景有效？

🔄 挂起函数与取消机制
如何取消协程？Job.cancel() 的原理是什么？

isActive、ensureActive()、yield() 这些函数你用过吗？各自作用？

什么是 cooperative cancellation？为什么它重要？

挂起函数执行过程中如果不检查取消，会发生什么？如何改进？

📱 与 Android 框架结合
你在项目中是如何使用协程的？是否与 ViewModel、LiveData、Flow 结合使用？

viewModelScope 和 lifecycleScope 分别是什么？为何推荐使用它们？

你用过 LiveData + Coroutine 的组合吗？如何处理生命周期？

协程在 Activity 被销毁后仍在运行，会发生什么问题？如何避免？

🧪 实践场景题（带思维能力考察）
你有一个网络请求 + 本地缓存的场景，怎么用协程实现？需要考虑并发和取消。

如何用 async 并发执行两个耗时任务，并收集它们的结果？

启动多个协程，如果其中某个失败，其他协程怎么办？如何控制它们的取消策略？

怎么写一个能自动超时的协程操作？你用过 withTimeout() 吗？

💥 陷阱与高阶
协程为什么可能导致内存泄漏？你遇到过吗？

为什么说协程虽然轻量，但不是免费线程？过度使用有什么风险？

协程和 Flow 配合时，怎么处理 backpressure（背压）？

你是否实现过自己的 suspend 函数？怎么让一个普通回调函数变成挂起函数？