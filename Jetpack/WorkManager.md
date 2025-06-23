⚙️ 一、基础理解
什么是 WorkManager？它适用于哪些使用场景？

WorkManager 与 JobScheduler、AlarmManager、FirebaseJobDispatcher 有何区别？为什么推荐使用 WorkManager？

WorkManager 支持哪些类型的任务？分别适用于什么场景？

OneTimeWorkRequest

PeriodicWorkRequest

ExpeditedWorkRequest（加急任务）

🧩 二、使用方式与配置
如何定义一个自定义的 Worker？有哪些方式？

Worker、CoroutineWorker 和 RxWorker 有什么区别？各自的优缺点是什么？

如何设置任务的约束条件（Constraints）？都支持哪些约束？

如何设置任务的延迟执行？重试策略？重复间隔？

如何取消一个正在运行或等待执行的任务？

🔗 三、任务链与依赖
如何串行或并行地执行一系列任务？用 WorkManager 如何实现任务链？

如果任务链中某个任务失败了，后续任务会怎么处理？能否做错误恢复？

任务合并器（WorkContinuation.combine()）在实际项目中如何应用？

🧠 四、底层机制与原理
WorkManager 在不同 Android 版本下的底层调度机制是怎样的？（例如 API 23+ 使用 JobScheduler，低版本用 AlarmManager + BroadcastReceiver）

WorkManager 是如何保证任务“可靠执行”的？是否有任务丢失的风险？如何规避？

WorkManager 的任务是在主线程执行吗？如何进行线程切换？

如何处理应用重启或系统重启后的任务恢复问题？WorkManager 是如何持久化任务的？

📦 五、进阶实战题
你有用 WorkManager 做过文件上传/数据库备份/日志上报等后台任务吗？遇到哪些坑？怎么解决的？

如何让任务在应用被杀死或设备重启后依然执行？需要配置什么？

你如何调试 WorkManager？能否查看任务的运行状态？

WorkManager 和前台服务（ForegroundService）如何配合？加急任务是否也需要前台通知？

🧬 六、源码与架构设计
WorkManager 的调度器（Schedulers）是怎么设计的？如何选择合适的调度器？

你了解 WorkManager 的任务状态流转（ENQUEUED、RUNNING、SUCCEEDED、FAILED 等）吗？源码中是如何维护状态的？

数据库表结构你了解吗？任务是如何存储和恢复的？Room 有什么作用？

在高并发下，有没有分析过 WorkManager 的并发模型和线程池配置？是否存在线程安全隐患？

🧪 七、面试实战场景题（情景设计）
如果你要实现“每天早上8点检查一次App是否需要更新并弹出提示”，你会用 WorkManager 吗？为什么？如何实现？

你在项目中遇到 WorkManager 不执行、延迟很久才执行、任务状态卡住等问题，是怎么排查和解决的？

项目中多个模块都想使用 WorkManager 提交任务，如何进行统一管理？有没有做过封装？