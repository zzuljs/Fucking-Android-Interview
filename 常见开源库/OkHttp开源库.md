# 📦 基础使用
OkHttp的核心使用流程是怎样的？基本用法？

OkHttp中的OkHttpClient为什么建议复用，而不是每次新建？

如何配置超时时间？分别有哪些超时时间？

如何添加请求头、请求体、参数等？

怎么取消一个正在进行的请求？

# ⚙️ 核心组件与设计
OkHttp的核心模块有哪些？（如：Dispatcher、ConnectionPool、Interceptor、RealCall）

说说OkHttp中的拦截器(Interceptor)的作用和分类？

应用拦截器（Application Interceptor）

网络拦截器（Network Interceptor）

两者的区别？

OkHttp的连接复用是如何实现的？ConnectionPool是干嘛的？

OkHttp如何处理DNS解析、代理、SSL证书校验？

🧵 线程模型
Dispatcher的作用是什么？最大请求数、最大主机请求数默认是多少？

Dispatcher的队列模型是怎样的？如何保证并发请求数量受控？

OkHttp内部用到了哪些线程池？为什么这么设计？

🔗 网络协议支持
OkHttp支持哪些HTTP协议版本？如：HTTP/1.1、HTTP/2、HTTP/3。

HTTP/2相比HTTP/1.1在性能上有哪些优势？OkHttp做了哪些适配？

如何开启或关闭HTTP/2？

🚀 高级应用与扩展
如何在OkHttp中实现请求重试与失败重连？

如何实现请求缓存？OkHttp缓存策略的核心逻辑是怎样的？

如何做持久连接、心跳机制？

OkHttp如何支持WebSocket？

🔎 源码与底层机制
OkHttp请求的整体流程是什么？（从newCall().execute()开始）

OkHttp为什么高效？连接复用、路由选择、拦截器链的设计亮点在哪里？

拦截器链 (Interceptor Chain) 责任链模式的具体实现逻辑？

OkHttp内部的连接复用为什么可以避免三次握手的频繁开销？

OkHttp如何实现了异步请求？异步和同步请求的区别？

ConnectionPool中的IdleConnection是如何回收的？

🔐 安全与稳定性
如何配置OkHttp支持HTTPS？如何处理自签名证书？

如何处理证书校验失败的问题？

如何避免SSL握手超时？

⚠️ 项目实战与优化
在你项目中用过OkHttp吗？遇到过哪些坑？

OkHttp的内存泄漏场景有哪些？

怎么监控OkHttp请求的耗时、失败率、请求日志？

如何和Retrofit结合使用？两者的职责分工？

🧠 开放性思考题
如果让你自己设计一个精简版OkHttp，会如何设计架构？

OkHttp和其他网络库（如：HttpURLConnection、Volley、Retrofit、Cronet）相比，优缺点是什么？

