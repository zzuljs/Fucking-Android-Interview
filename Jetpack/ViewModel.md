# 🧠 基础理解类

## 什么是 ViewModel？它的基本用法有哪些？作用是什么？  

ViewModel是Jetpack架构组件之一，专为界面相关的数据存储和管理而设计，主要目的是在配置变化（如屏幕旋转）时保留和管理UI相关数据，避免Activity/Fragment被销毁重建时数据丢失  

```kotlin
// 创建自己的ViewModel类
class MyViewModel : ViewModel() {
    // 结合LiveData一起使用
    val count = MutableLiveData<Int>()

    fun increment() {
        val current = count.value ?: 0
        count.value = current + 1
    }
}

// Activity中使用
val viewModel = ViewModelProvider(this)[MyViewModel::class.java]

// Fragment中使用，需要宿主Activity
val viewModel = ViewModelProvider(requireActivity())[MyViewModel::class.java]

// 观察LiveData
viewModel.count.observe(this) { value ->
    textView.text = value.toString()
}

// 调用业务逻辑方法
button.setOnClickListener {
    viewModel.increment()
}
```  

ViewModel的核心价值在于：

1. 数据配置变更持久化  

ViewModel 的生命周期独立于 Activity/Fragment 的生命周期，它只在宿主 Activity 完全销毁（例如用户按返回键退出应用）时才会被清除。因此，将 UI 相关的数据（例如用户输入、网络请求结果、滚动位置等）存储在 ViewModel 中，可以确保这些数据在配置变更后依然存在，无需你手动在 onSaveInstanceState() 和 onRestoreInstanceState() 中进行复杂的保存和恢复  

2. 职责分离  

- ViewModel 将 UI 的状态和行为逻辑从 Activity/Fragment 中抽离出来
- Activity/Fragment (View 层) 变得更“薄”，只负责显示 UI、处理用户输入事件，并观察 ViewModel 中暴露的数据
- ViewModel (ViewModel 层) 则负责业务逻辑、数据获取（通过 Repository）、数据处理和准备 UI 所需的数据
- 这种分离使得代码结构更清晰、更易于维护，也大大提高了代码的可测试性。你可以独立地对 ViewModel 进行单元测试，而无需 Android 模拟器或设备

## ViewModel 的生命周期与 Activity、Fragment 有什么关系？  

ViewModel生命周期与Activity绑定是androidx库中ComponentActivity提供的能力  

本质上，ViewModel不与Activity、Fragment直接绑定，而是与ViewModelStore绑定，ComponentActivity管理ViewModelStore  

```kotlin
public class ComponentActivity extends ... implements ViewModelStoreOwner {
    private ViewModelStore mViewModelStore;
    
    @Override
    public ViewModelStore getViewModelStore() {
        if (mViewModelStore == null) {
            mViewModelStore = new ViewModelStore();
        }
        return mViewModelStore;
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        if (!isChangingConfigurations()) {
            // Activity 不是因配置更改而销毁，释放 ViewModel
            mViewModelStore.clear();
        }
    }
}
```

1. 对 Activity 而言   

使用 ViewModelProvider(this) 创建的 ViewModel，其生命周期和 Activity 绑定

Activity 被销毁（如调用 finish()），ViewModel 随之被清除

若仅是配置变更（如旋转屏幕），则 ViewModel 会自动保留

2. 对 Fragment 而言

使用 ViewModelProvider(this)，ViewModel 和 Fragment 绑定，Fragment 被销毁时一起销毁  

使用 ViewModelProvider(requireActivity())，则 ViewModel 和宿主 Activity 绑定，多个 Fragment 可共享  

## ViewModel 中适合做哪些事情？不适合做哪些事情？  

适合做的事情：

| 任务                                  | 说明                                              |
| ----------------------------------- | ----------------------------------------------- |
| 💾 **持有 UI 状态数据**                   | 比如列表数据、选中项、分页信息、加载状态等，ViewModel 在配置变化中会自动保留这些数据 |
| 🔄 **封装业务逻辑 & 请求数据**                | ViewModel 应该负责从仓库（Repository）中拉取数据、缓存、转换等       |
| 🎯 **状态管理 / 状态机实现**                 | 比如加载状态（loading / success / error）、UI 事件（导航、弹窗等） |
| 🧪 **做轻量逻辑处理**                      | 如字段拼接、列表过滤、类型转换、状态切换（不需要 UI 参与的逻辑）              |
| 🧵 **协程作用域（viewModelScope）中执行异步任务** | 可安全执行网络请求、数据库操作，不担心生命周期泄漏                       |


不适合做的事情：

| 不推荐行为                                   | 原因                                     |
| --------------------------------------- | -------------------------------------- |
| ❌ 持有 Context、Activity、Fragment 引用       | 会造成内存泄漏！ViewModel 生命周期比 UI 控制器长        |
| ❌ 处理 UI 操作，如 Toast、Dialog、Navigation    | ViewModel 是“无视图”的，不应操作 UI 层            |
| ❌ 做重量级长时间计算                             | ViewModel 是轻量容器，重计算应放到后台服务或 use case 层 |
| ❌ 写 View 相关逻辑（颜色、动画、尺寸、资源等）             | ViewModel 不应关心界面细节，这属于 View 层职责        |
| ❌ 管理生命周期资源（如 Camera、Sensor、MediaPlayer） | 这些需要和 UI 生命周期强绑定，不适合用 ViewModel 来托管    |


## ViewModel 与 onSaveInstanceState 有什么区别与联系？  

ViewModel 保存在内存中，跨越配置变更，生命周期比Activity长，不持久

onSaveInstanceState 保存到Bundle（磁盘或内存），用于重建后的数据恢复，可跨进程销毁  

两者应用场景不同，使用上是互补的  

# ⚙️ 使用细节类

## ViewModel 是如何避免内存泄漏的？

## 如何向 ViewModel 传参？

## 如果 ViewModel 需要持有 Context 对象，该怎么处理？

## 多个 Fragment 共用一个 ViewModel 怎么实现？其原理是什么？

## ViewModel 中启动协程，有哪些注意事项？

## ViewModel 如何结合 LiveData / StateFlow / Flow 使用？

# 🔍 源码机制类

## ViewModel 是如何被创建与存储的？背后用了什么数据结构？

## ViewModel 的默认工厂（ViewModelProvider.Factory）是怎么工作的？

## Android 是如何保证配置变化（如旋转屏幕）时 ViewModel 不会被销毁？

## ViewModelStore 和 ViewModelStoreOwner 是什么？它们的作用是什么？

## 清除 ViewModel 的时机是何时？onCleared() 是什么时候被调用的？

# 📚 进阶场景类

## 如果 ViewModel 中存在异步请求，当用户返回页面再进入，怎么保证数据正确性？

## 使用多个 ViewModel 时，如何管理它们的协同工作？

## ViewModel 能否作为 Repository 的替代？为什么？

## MVVM 架构中 ViewModel 的职责边界应该如何划分？

# ⚔️ 对比与思辨类

## ViewModel 与 Presenter（MVP）中的 Presenter 有何异同？

## ViewModel 与 Repository 的职责有何不同？

## ViewModel 与 SavedStateHandle 的结合场景和优势是什么？

# 🛠️ 实战问题类

## 请你设计一个支持旋转恢复的倒计时组件，说明 ViewModel 如何参与其中？

## 如果你的 ViewModel 中维护了多个 Job，你如何优雅地管理与取消它们？

## 你是否用过 Hilt 注入 ViewModel？和传统写法有何不同？

## 如何测试 ViewModel 中的业务逻辑？你用过哪些工具或框架？