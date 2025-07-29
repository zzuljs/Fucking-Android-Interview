🧠 一、基础理解
你如何理解 RxJava 的响应式编程模型？

follow-up：和传统回调或协程比有什么优缺点？

Observable、Flowable、Single、Maybe、Completable 有什么区别？使用场景分别是什么？

你如何理解背压（Backpressure）？RxJava 是如何解决背压问题的？

subscribeOn 和 observeOn 的区别？分别影响哪部分线程？

CompositeDisposable 是什么？在 Activity/Fragment 中如何正确管理？

🛠️ 二、实际开发场景
你在项目中是如何使用 RxJava 做网络请求的？举一个典型的使用链条例子，并解释各个操作符的作用。

RxJava 中如何处理异常？你是怎么做到链路上的容错的？

多个网络请求需要串行/并行组合（比如 A 接口完成后请求 B 接口），你会怎么写？

RxJava 与生命周期的绑定问题是如何处理的？有没有遇到内存泄露问题？

RxJava 如何与 Room、Retrofit 结合使用？你在实际开发中有没有踩过坑？

🔍 三、进阶操作符与调度器
flatMap 与 concatMap 的区别是什么？它们的线程顺序和结果顺序有何差异？

switchMap 的使用场景是什么？它有什么潜在的坑？

Schedulers.io、Schedulers.computation、Schedulers.newThread、Schedulers.single 的区别？如何选择？

如何创建一个自定义的操作符？有没有实现过？

📦 四、源码层面（适合对 RxJava 有深入了解的候选人）
RxJava 的事件是如何从上游传递到下游的？简述它的事件传导机制。

你能简单分析一下 RxJava 的 subscribeOn 和 observeOn 是如何在源码中切换线程的？

你如何理解 Observable.create 与 just、fromCallable 等的区别？它们底层是如何封装的？

背压策略（BUFFER、DROP、LATEST、ERROR）在 Flowable 中是如何实现的？哪种策略默认使用？

✅ 五、开放性问题
你认为 RxJava 在今天的 Android 开发中是否还有优势？面对 Kotlin 协程，它的角色变化了吗？

如果你负责维护一个 RxJava 老项目，你会在什么前提下考虑替换为 Flow 或协程？为什么？