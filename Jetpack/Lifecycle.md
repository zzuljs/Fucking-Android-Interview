# 🧠 基础理解类

## 什么是 Lifecycle？它的作用是什么？

Lifecycle代表了Android组件（主要是Activity、Fragment）生命周期状态，以观察者模式分发生命周期，将依赖生命周期的业务解耦，Jetpack的很多组件都深度依赖Lifecycle  

```kotlin
class MyObserver : DefaultLifecycleObserver {

    // 这个方法会在 LifecycleOwner 的 onCreate 事件发生时被调用
    override fun onCreate(owner: LifecycleOwner) {
        // 执行初始化操作
        println("MyObserver: onCreate")
    }

    // 这个方法会在 LifecycleOwner 的 onStart 事件发生时被调用
    override fun onStart(owner: LifecycleOwner) {
        // 开始一些操作，例如注册广播接收器
        println("MyObserver: onStart")
    }

    // 这个方法会在 LifecycleOwner 的 onStop 事件发生时被调用
    override fun onStop(owner: LifecycleOwner) {
        // 停止一些操作，例如注销广播接收器
        println("MyObserver: onStop")
    }

    // 这个方法会在 LifecycleOwner 的 onDestroy 事件发生时被调用
    override fun onDestroy(owner: LifecycleOwner) {
        // 释放资源
        println("MyObserver: onDestroy")
    }
}

class MyActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 获取当前 Activity 的 Lifecycle，并添加观察者
        lifecycle.addObserver(MyObserver())
    }
}
```

## Lifecycle 和 Activity/Fragment 的生命周期有什么关系？

Lifecycle 和 Activity/Fragment 的生命周期是紧密相连且相互补充的关系。可以这样理解：

- Activity/Fragment 的生命周期：是 Android 框架定义的、组件自身所经历的一系列状态回调（如 onCreate(), onStart(), onResume(), onPause(), onStop(), onDestroy()）。它们是 Android 系统管理组件存在和行为的基础

- Lifecycle：是 Jetpack 库提供的一个抽象层，它将 Activity/Fragment 的这些底层生命周期回调抽象化并暴露为一个可观察的对象。它让任何其他组件（我们称之为“生命周期感知组件”）都能监听并响应 Activity/Fragment 的生命周期变化，而无需直接在 Activity/Fragment 的回调方法中编写大量耦合代码

## Lifecycle 是如何感知宿主（如 Activity）生命周期的？

1. 宿主ComponentActivity实现了LifecycleOwner接口

所有的宿主都实现了LifecycleOwner，即所有宿主都有一个LifecycleOwner实例，对于ComponentActivity、Fragment而言，内部会创建一个LifecycleRegistry作为Lifecycle对象  
```java
// appcompat-1.8 ComponentActivity.java

private final LifecycleRegistry mLifecycleRegistry = new LifecycleRegistry(this);

@NonNull
@Override
public Lifecycle getLifecycle() {
    return mLifecycleRegistry;
}
```

2. LifecycleRegistry的内部实现  

LifecycleRegistry是Lifecycle接口的默认实现，负责维护当前生命状态，并向所有注册的观察者分发事件  

```java
// 分发生命周期的核心方法
public void handleLifecycleEvent(@NonNull Event event) {
    // ... 内部逻辑，根据传入的事件更新当前状态，并通知观察者
}
```

这个LifecycleRegistry通过一个无UI的ReportFragment来触发，通过handleLifeEvent分发生命周期事件  

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

