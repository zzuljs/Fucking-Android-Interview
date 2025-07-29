🚦一、基础理解（判断候选人是否真的用过 Navigation）
Jetpack Navigation 是如何帮助管理 Fragment 的？相较传统的 FragmentTransaction 有哪些优势？

NavController 和 NavHostFragment 分别是什么？它们的职责划分是怎样的？

你能讲一下 Navigation Graph 的作用吗？它如何支持可视化编辑？

在使用 Navigation 时，如何传递参数？safeArgs 是什么？有什么优势？

🔧二、进阶使用与项目实战（挖掘候选人是否能落地复杂导航场景）
你在项目中使用 Navigation 遇到过什么坑？比如返回栈、动画、状态恢复等？怎么解决的？

如何实现多个模块间的解耦导航？Navigation 如何支持组件化项目？你是怎么落地的？

Navigation 支持 DeepLink 吗？你是如何处理 DeepLink 进入 App 后导航流程的？

你是如何处理返回键（BackStack）的？比如多模块嵌套导航、BottomNavigation+NavHost 的组合下？

🧠三、原理与源码视角（适合高级开发或技术专家）
NavBackStackEntry 的生命周期是如何管理的？它与 Fragment 生命周期有何区别？

你是否了解 Navigation 的导航动作是如何从 XML graph 转换为 FragmentTransaction 的？能否简要描述源码调用链？

Navigation Graph 是怎么进行 ID 映射、节点查找、动作解析的？你是否读过相关源码？

🧩四、架构设计与可扩展性（看是否具备架构能力）
如果你的 App 采用的是多Activity架构，你会怎么管理导航？Navigation 还适用吗？

Navigation 如何和底层路由框架（如ARouter）协同使用？你会如何做兼容设计？

你如何在 MVVM 或 MVI 架构中使用 Navigation？怎么避免 ViewModel 中耦合导航逻辑？

📦五、开放题 / 实战题（考察候选人系统思考和实际落地能力）
设想一个复杂场景：App 有多个 Tab，每个 Tab 下有独立的导航栈，要求切换 Tab 时保存状态，再次进入恢复状态，你会如何实现？Navigation 能胜任吗？你如何设计？

如果公司希望 App 支持“脚本式配置导航”，即导航规则来自服务器，你认为 Navigation 是否合适？如果不合适，你会如何做改造或设计？

✅ Bonus（小细节决定候选人对细节的掌握程度）
NavOptions 中有哪些参数？如何控制动画、返回栈行为？

使用 Navigation 遇到 IllegalStateException 的可能原因有哪些？如何规避？

如何优雅地在非 Activity/Fragment 的类中触发导航行为？