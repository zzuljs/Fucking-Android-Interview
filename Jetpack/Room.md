✅ 基础使用相关
什么是 Room？它与 SQLite 有什么关系？

Room 中的三大核心组件分别是什么？它们的职责是什么？

如何在 Room 中定义一张表？有哪些注解？

@PrimaryKey(autoGenerate = true) 有什么作用？底层如何实现自增？

@Embedded 和 @Relation 分别是什么？解决了什么问题？

Room 支持哪些数据类型？如何存储复杂类型如 List、Date 等？

如何自定义字段类型的转换逻辑？@TypeConverter 的用法？

Room 中 DAO 接口支持哪些操作？@Insert、@Update、@Delete 的限制？

🔁 查询与事务
Room 的 @Query 支持哪些 SQL 功能？是否支持 JOIN？模糊查询怎么写？

Room 是如何保证线程安全的？为何不能在主线程查询？

Room 如何处理事务？如何保证事务原子性？

@Transaction 注解有什么作用？什么场景必须用它？

Room 中如何做批量插入或更新？性能如何？

🔄 观察与响应式支持
Room 是如何与 LiveData/Flow 协作实现数据变更通知的？

Room + Flow 的组合中，背压是如何处理的？Flow 是冷流如何适配 Room？

Room 如何检测数据变更并推送通知？它是监听文件、表，还是别的？

🔒 并发与性能优化
Room 是否支持多线程访问？并发场景下如何保证数据一致性？

Room 中使用了哪些 SQLite 优化策略？例如 WAL、预编译语句等？

Room 中的查询语句可以缓存吗？有哪些优化技巧？

Room 数据库版本升级（version 升级）是如何处理的？你写过 migration 吗？

🧩 设计与架构思想
你如何看待 Room 中 DAO 的设计？它是否符合面向接口编程思想？

Room 的架构中，为什么不直接暴露 SQLiteOpenHelper？

Room 为什么不推荐直接暴露 Entity，而是封装在 Repository 中？

🧬 源码与原理向
Room 的注解处理器（Annotation Processor）做了哪些代码生成？生成了哪些类？

Room 的 @Database 最终生成的类是什么样子？构造逻辑在哪里？

Room 是如何实现数据库变更监听并通知 LiveData/Flow 的？中间有哪些类？

Room 的编译时校验是如何实现的？为什么它能在编译期发现 SQL 错误？

Room 是怎么在后台线程执行操作并和 UI 线程通信的？

💡 开放型与场景题
请设计一个包含多表关联（如用户、订单、商品）的 Room 数据结构，并实现一个查询最近7天内下单用户及其订单详情的函数。

如果 Room 查询过慢，你会如何排查性能瓶颈？

在实际项目中你是否使用过 Room + Paging3？是如何集成的？遇到过什么问题？