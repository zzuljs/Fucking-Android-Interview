1. Flow 的基本概念
什么是 Kotlin Flow？它解决了什么问题？

Flow 和 RxJava 的 Observable 有什么异同？

Flow 和 LiveData 有什么区别和联系？

2. Flow 的构建与收集
如何创建一个 Flow？举例说明。

Flow 是冷流（Cold Stream）还是热流（Hot Stream）？请解释原因。

Flow 的 collect 操作是什么？有什么作用？

Flow 什么时候开始执行代码？什么时候停止执行？

3. Flow 的操作符
请说说你常用的 Flow 操作符有哪些？比如 map、filter、flatMapConcat、flatMapMerge、flatMapLatest 的区别是什么？

Flow 中的 transform 操作符和 map 有什么区别？

Flow 的 debounce、distinctUntilChanged 作用是什么？适合什么场景？

4. Flow 与协程的关系
Flow 是如何与协程结合使用的？

Flow 的 collect 是挂起函数吗？为什么？

如何在 ViewModel 中安全地使用 Flow？

5. 线程与调度
Flow 默认在哪个线程执行？如何切换线程？

withContext 和 flowOn 的区别和联系？

flowOn 的执行位置和作用是什么？

6. 异常处理
Flow 中如何处理异常？

Flow 的 catch 和 try-catch 有什么区别？

如何实现 Flow 的重试机制？

7. Flow 的取消与生命周期
Flow 是如何支持取消的？

协程取消时，Flow 会怎样表现？

如何在 Activity 或 Fragment 中绑定 Flow 的生命周期？

8. SharedFlow 和 StateFlow
什么是 SharedFlow 和 StateFlow？它们和普通 Flow 有什么区别？

StateFlow 是如何实现状态管理的？

SharedFlow 有哪些应用场景？

如何使用 StateFlow 实现 MVVM 中的数据绑定？

9. 进阶与性能
Flow 的背压（Backpressure）是如何处理的？

Flow 的缓冲（buffer）机制如何工作？什么时候用它？

Flow 的 collectLatest 和 collect 有什么区别？

10. 实际应用
请说说你项目中用 Flow 解决了哪些问题？

在 Flow 结合 Room 数据库时，有哪些注意点？

如何使用 Flow 结合 Retrofit 实现网络请求？