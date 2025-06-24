🧱 一、基础理解类  

1. Gradle 是什么？它在 Android 项目中起什么作用？
follow-up: Gradle 和 Maven 有哪些本质上的区别？

2. 你了解 Gradle 构建生命周期（Build Lifecycle）吗？有哪些阶段？
3. build.gradle 文件中常见的几个重要字段分别是什么意思？
比如 plugins、android、dependencies、buildTypes、flavorDimensions 等。

🧩 二、配置与脚本能力  

4. build.gradle 中 implementation 和 api 有什么区别？如何选择？  
5. Android 项目中 buildTypes 和 productFlavors 是如何工作的？你实际用过哪些场景？  
6. 什么是 Gradle DSL？你写过哪些自定义逻辑？举个例子。  
7. 你了解 gradle.properties、local.properties 的区别与用途吗？  
8. ext 扩展字段怎么用？有没有管理依赖版本的最佳实践？  

🧰 三、自定义能力与优化实践  

9. 你写过 Gradle 插件吗？请谈谈实现思路和使用方式。
follow-up：Gradle 插件和 Task 是什么关系？

follow-up：如何为公司定制公共构建逻辑？

10. Gradle 是如何处理任务依赖关系的？你是否手动定义过 Task 的执行顺序？
11. 有遇到过构建速度很慢的问题吗？你是如何定位和优化构建性能的？
follow-up：了解 configuration phase vs execution phase 吗？

follow-up：--profile 和 build scan 用过吗？

12. 你是否使用过 buildSrc 模块？它的作用是什么？
🌐 四、多人协作与CI集成
13. 你如何管理多模块 Gradle 项目？是否做过组件化、插件化相关实践？
14. 你是如何在 CI/CD 环境中使用 Gradle 构建 Android 项目的？
follow-up：有使用过 Gradle Wrapper 吗？为什么推荐使用？

🧪 五、进阶与细节探索
15. Gradle 是如何与 Kotlin DSL 配合的？你更倾向使用 Groovy 还是 Kotlin 脚本？为什么？
16. Gradle 中的 Configuration Avoidance API 是什么？为什么推荐用 register() 而不是 create()？
17. 了解 Gradle 的增量构建原理吗？什么情况会导致构建缓存失效？
18. Gradle 依赖冲突怎么解决？有没有踩过坑？用过 resolutionStrategy 吗？
💬 六、开放性问题（用于评估候选人经验）
19. 请谈谈你在实际工作中遇到过的最复杂的 Gradle 使用场景，并说说你是怎么解决的？
20. 如果你接手一个构建时间超过 3 分钟的 Android 项目，你会怎么排查、优化？
