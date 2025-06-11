# 一、基本原理与使用
Retrofit 的基本使用流程是什么？

Retrofit 和 OkHttp 之间是什么关系？

Retrofit 的注解（@GET、@POST、@Query、@Body、@Path、@Header 等）都有什么作用？

@FormUrlEncoded 和 @Multipart 有什么区别？

# 二、设计思想
Retrofit 的核心设计思想是什么？为什么说它用了“面向接口编程”？

Retrofit 是如何将接口方法动态转换为网络请求的？

Retrofit 的动态代理是如何实现的？用到了哪些Java基础知识？

# 三、底层实现细节
Retrofit 的整体架构是怎样的？涉及哪些核心类？（如：Retrofit、ServiceMethod、OkHttpCall、CallAdapter 等）

Retrofit 的请求流程可以简要描述一下吗？从接口调用到最终发送请求。

Retrofit 如何实现参数注解到请求的转换？@Query、@Path 是如何拼接 URL 的？

Retrofit 中的 Converter 是如何工作的？常见的有哪些 Converter？

Retrofit 的 CallAdapter 有什么作用？它在异步和同步调用中扮演了什么角色？

# 四、与协程和 RxJava 的集成
Retrofit 如何支持协程（suspend function）？背后的实现机制是什么？

Retrofit 如何支持 RxJava？需要注意哪些坑？

如果 Retrofit 协程请求失败，你一般如何做异常捕获与重试？

# 五、扩展与自定义
如何自定义 Converter？举个自定义 GsonConverter 的例子。

如何自定义 CallAdapter？实际工作中有没有用过？

Retrofit 是否支持拦截器？和 OkHttp 拦截器的关系如何？

# 六、性能与优化
Retrofit 中接口复用、全局 Retrofit 单例管理怎么做更合理？

Retrofit 在高并发场景下有没有性能瓶颈？如何优化？

Retrofit 如何处理线程切换？在哪个线程执行请求、在哪个线程回调？

# 七、容错与最佳实践
你在实际项目中如何处理 Retrofit 的超时、断网、重试等问题？

Retrofit 的全局异常捕获怎么设计？

如何与统一的 API 返回规范（如统一的 Result 包装类）结合使用？

# 八、开放性思考题
Retrofit 能否用在 WebSocket、长连接等场景？为什么？

相比于 Retrofit，Volley、Ktor、GraphQL 各自有什么优缺点？

如果让你设计一套类似 Retrofit 的库，你会怎么设计？