# 🌱 一、基础使用（基础掌握）
## 什么是 Jetpack Compose，它相较于传统 View 有哪些优势？

## @Composable 注解的作用是什么？

## remember 和 mutableStateOf 的作用和使用场景？

## Compose 中如何实现列表（如 LazyColumn）？

## 如何在 Compose 中处理点击事件？

## 如何在 Compose 中实现文本输入框？和传统 EditText 有哪些区别？

## Compose 中的 Modifier 是什么？它解决了什么问题？

## SideEffect、LaunchedEffect、DisposableEffect 有什么区别？什么时候用？

## 如何让 Composable 响应某个状态变化？比如按钮点击后刷新 UI。

# 🌿 二、架构与状态管理（进阶）
## Compose 是如何进行状态管理的？什么是 Unidirectional Data Flow？

## remember 和 rememberSaveable 区别？使用场景？

## Compose 中如何与 ViewModel 配合？如何使用 collectAsState()？

## 如何在 Compose 中观察 LiveData 或 StateFlow？

## Compose 如何处理状态丢失，比如旋转屏幕？

## 如何实现 Compose 和 XML 的混合使用？有哪些限制？

# 🌳 三、导航与生命周期（进阶）
## Compose 中如何实现页面导航？NavController 是如何工作的？

## 如何处理返回键？如何设置某页面拦截返回？

## Compose 中如何感知生命周期？可以获取生命周期状态吗？

## DisposableEffect 如何释放资源？和 onDispose 有何区别？

# 🔥 四、自定义组件与重组机制（核心原理）
## 什么是重组（Recomposition）？Compose 如何判断一个 Composable 是否需要重组？

## Compose 是如何实现 UI diff 的？有没有 Virtual DOM？

## Compose 中的 key() 有什么作用？为什么需要它？

## Modifier 是如何实现链式组合的？背后实现原理？

## 如何写一个可复用、组合性强的自定义 Composable？

## Compose 如何和 Canvas 协作实现自定义绘制？比如饼图、雷达图等？

# 🚀 五、性能优化与调试（高阶）
## Compose 重组次数多会有什么问题？如何避免不必要的重组？

## 使用 derivedStateOf 是为了什么？它解决了什么问题？

## 如何检测 Compose 中的性能瓶颈？你用过哪些工具？

## 你如何理解 Compose 的 SlotTable、Group、Applier 等核心数据结构？

## Compose 的 Compiler 插件在编译期间做了哪些工作？为何说 Compose 是编译器驱动的 UI 框架？

# 🧬 六、与协程、Flow 结合（高阶）
## 如何在 Compose 中使用协程请求数据？为何推荐用 LaunchedEffect？

## 如何优雅地从 Flow 或 StateFlow 获取数据更新并刷新 UI？

## 如何避免协程泄漏或重复启动的问题？

# 🧩 七、Compose 与项目架构整合（高阶）
## Compose 如何整合到已有的 MVP、MVVM、MVI 架构中？

## 你是如何在 Compose 项目中组织模块的？有哪些最佳实践？

## Compose UI 的测试该如何写？支持哪些类型的测试？

## Compose 中有没有数据绑定的能力？如何实现？

# 🎯 八、开放式问题（考察理解深度与工程思维）
## 如果你负责一个大项目从 XML 迁移到 Compose，你会怎么规划和拆分？

## Compose 的组合性带来哪些好处和潜在风险？

## 你认为 Compose 是不是未来的方向？它有哪些不足？你如何看待它的生态现状？