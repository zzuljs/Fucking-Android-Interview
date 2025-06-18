# 🧠 基础理解类

## 什么是 ViewModel？它的作用是什么？

## ViewModel 的生命周期与 Activity、Fragment 有什么关系？

## ViewModel 中适合做哪些事情？不适合做哪些事情？

## ViewModel 与 onSaveInstanceState 有什么区别与联系？

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