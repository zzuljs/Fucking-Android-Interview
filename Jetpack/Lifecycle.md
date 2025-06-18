# 🧠 基础理解类

## 什么是 Lifecycle？它的作用是什么？

考察点：是否理解 Android 生命周期的抽象封装。

## Lifecycle 和 Activity/Fragment 的生命周期有什么关系？

考察点：是否理解二者的绑定关系。

## Lifecycle 是如何感知宿主（如 Activity）生命周期的？

考察点：是否了解 ReportFragment 或 LifecycleDispatcher 的原理。

## 常见的 Lifecycle.State 有哪些？它们的含义是什么？

INITIALIZED、CREATED、STARTED、RESUMED、DESTROYED

## LifecycleOwner 和 LifecycleObserver 是什么？它们的作用是什么？

# 🧩 源码与机制类

## Lifecycle 是如何通知观察者状态变化的？源码流程是怎样的？

考察点：是否了解 LifecycleRegistry 和 ObserverWithState 的分发机制。

## LifecycleObserver 接口与 DefaultLifecycleObserver 有什么区别？为什么引入后者？

回答应提到默认方法支持（Java 8+）的问题、可读性、类型安全等。

## Lifecycle 事件的分发顺序是怎样的？多个 Observer 注册时会有哪些注意事项？

# ⚒️ 实践与应用类

## 你在项目中是如何使用 Lifecycle 的？有哪些典型场景？

比如：监听前后台切换、Camera/MediaPlayer/Map 等组件释放与重建等。

## LiveData 是如何结合 Lifecycle 自动取消观察的？

考察点：LiveData 如何在 onInactive() 和 onActive() 中配合 Lifecycle。

## Lifecycle 在 Jetpack 中的其它组件（如 ViewModel、LiveData、Navigation）中扮演什么角色？

# 🧱 架构与设计类

## Lifecycle 的引入解决了 Android 开发中哪些痛点？

比如：生命周期错位回调、内存泄漏、资源无法释放等。

## 你了解 Lifecycle 的替代方案或更底层的实现方式吗？比如自己封装生命周期感知？

## 你如何使用 Lifecycle 实现一个组件自动感知生命周期的能力（比如实现一个 Lifecycle 感知型 Logger）？

## 如何在自定义组件或 SDK 中优雅地使用 Lifecycle？有何注意事项？

# 🧪 延伸思考类
## Lifecycle 是否线程安全？在多线程中如何使用需要注意什么？

## Lifecycle 如何支持跨模块开发？在组件化架构中如何解耦对 Lifecycle 的使用？

## Lifecycle 和协程（Coroutine）是如何配合使用的？比如在 repeatOnLifecycle 中如何自动取消挂起任务？

