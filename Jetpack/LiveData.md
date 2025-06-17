🧠 一、基础理解类
什么是 LiveData？它的基本用法有哪些？作用是什么？

LiveData 有哪些常用的子类？它们之间有什么区别？

如：MutableLiveData、MediatorLiveData、LiveData、Transformations.map、Transformations.switchMap。

LiveData 和普通的 Observable 有什么区别？

LiveData 为什么感知生命周期？它是如何做到的？

什么是黏性事件？LiveData 是黏性的吗？怎么避免黏性？

⚙️ 二、工作原理与源码机制
LiveData 如何感知 LifecycleOwner 的生命周期？源码里是怎么实现的？

期望你能提到 LifecycleObserver / LifecycleBoundObserver 等关键类。

LiveData 是如何管理观察者（Observer）的？支持多个 Observer 吗？如何保证线程安全？

LiveData 的 setValue() 和 postValue() 有什么区别？内部是如何实现的？

LiveData 是如何防止重复通知的？为什么你有时候会收不到数据？

LiveData 源码中有没有使用到设计模式？比如观察者模式？还有哪些？

🛠 三、实战应用与场景题
在 ViewModel 中使用 LiveData 有什么优势？

你是如何在项目中使用 LiveData 实现事件通知的？有没有遇到黏性事件的问题？如何解决？

LiveData 能否用在 Repository 或网络请求中？如何使用？

假如你需要向多个 UI 组件广播一次性事件，LiveData 能满足吗？如果不能你会如何做？

你了解 SingleLiveEvent 吗？它解决了什么问题？你如何实现一个非黏性的 LiveData？

🔍 四、对比与延伸问题
LiveData 和 Flow 有什么区别？你在什么场景下选择哪一个？

LiveData 和 RxJava 中的 Observable 有什么不同？各自的优缺点是什么？

为什么谷歌官方在新的架构组件中更推荐使用 Flow 而不是 LiveData？

🧩 五、扩展与思辨
LiveData 有没有可能内存泄漏？在什么情况下可能会发生？如何避免？

你能手写一个简化版的 LiveData 实现吗？