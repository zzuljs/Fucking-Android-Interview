🧠 一、基础概念类问题
什么是Hilt？为什么要使用它？

Hilt和Dagger是什么关系？

Hilt相对于Dagger有哪些简化和封装？

在Android中使用Hilt带来了哪些便利？

@Inject 和 @HiltAndroidApp 有什么作用？

🧱 二、注解与作用域相关问题
@HiltAndroidApp 的作用是什么？为什么必须加在 Application 上？

@AndroidEntryPoint 有什么用？可以加在哪些类上？为什么 Activity/Fragment 都要加这个注解？

@Inject 和 @Provides 的区别是什么？使用场景分别是什么？

@Singleton 的作用是什么？在Hilt中是如何实现单例的？

@ActivityRetainedScoped 是什么？和 @ViewModelScoped 区别是什么？

Hilt 中有哪些常见的作用域注解？能否举例说明它们适用于哪些组件？

Hilt 提供的作用域是线程安全的吗？

🧰 三、模块与绑定相关问题
Hilt 中的 Module 是什么？为什么需要它？

@InstallIn 注解的作用是什么？必须要指定安装在哪个 Component 吗？

@Binds 和 @Provides 有什么区别？什么时候用 @Binds？

如何将接口与实现类进行绑定？Hilt 是如何知道要注入哪个实现的？

你能手写一个用 Hilt Module 提供 Retrofit 实例的例子吗？

🔄 四、组件与生命周期集成
Hilt 支持哪些 Android 组件的依赖注入？

Fragment 和 ViewModel 中的依赖注入是如何实现的？有什么限制？

为什么 ViewModel 中不能直接使用 @Inject 构造函数注入？应该怎么做？

Hilt 中 ViewModel 的注入为什么必须配合 @HiltViewModel？这个注解做了什么？

⚙️ 五、底层原理与编译器机制
Hilt 是如何工作的？底层用到了哪些技术？（例如：注解处理器、代码生成）

你能描述一下编译期 Hilt 是如何生成代码并注入依赖的吗？

Component 是什么？Hilt 中有哪些 Component？这些 Component 有什么层级关系？

Hilt 是如何处理作用域之间的依赖传递的？

你了解 @GeneratedEntryPoint 和 Hilt 的 codegen 机制吗？

🔍 六、进阶用法与技巧
如何为自定义的类（比如工具类、协程调度器）注入依赖？

如何注入带参数的构造函数？Hilt 支持 Assisted Injection 吗？

如何使用 Qualifier 区分不同类型的依赖？举一个你用过的例子。

如果我需要注入多个相同类型的依赖，如何处理？（例如多个 Retrofit 实例）

Hilt 是否支持多模块项目？在多模块项目中应该如何配置依赖注入？

🧪 七、测试与调试相关问题
如何在单元测试中使用 Hilt？和正常开发有什么不同？

如何在 Instrumentation Test 中注入测试依赖？用到哪些注解？

@TestInstallIn 和 @UninstallModules 是什么？怎么使用？

你在使用 Hilt 进行测试时遇到过什么问题？是怎么解决的？

🆚 八、与其他依赖注入框架对比
Hilt 与 Dagger2 有哪些异同？为什么 Hilt 更适合 Android 项目？

Hilt 和 Koin 的对比你怎么看？为什么选择 Hilt？

你是否用过 Dagger 原生写 Component 的方式？和 Hilt 有哪些显著不同？

📦 九、实际项目与经验问题
你在项目中是如何引入和使用 Hilt 的？是否从 Dagger 迁移过来？

你在使用 Hilt 时遇到过哪些坑？是怎么解决的？

有没有遇到 Hilt 无法注入或编译报错的情况？怎么排查？

你会在 Hilt 中注入 View 吗？为什么不建议这么做？

你如何保证注入的对象在合适的生命周期范围内被销毁？

Hilt 对 App 启动速度有没有影响？有没有优化方法？

🧩 十、开放性问题（思维延伸）
如果你要在项目中实现一套简化的依赖注入框架，你会如何设计？借鉴 Hilt 哪些思想？

Hilt 是否可以用于非 Android 项目？如果不可以，为什么？

你觉得在小型项目中用 Hilt 是不是有点“杀鸡用牛刀”？你怎么看？

你怎么看待依赖注入的“滥用”问题？Hilt 是否会降低代码可读性？