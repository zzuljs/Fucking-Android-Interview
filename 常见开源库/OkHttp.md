# 📦 基础使用

## OkHttp的核心使用流程是怎样的？基本用法？　　

1. 最基本用法：同步请求 

```java
OkHttpClient client = new OkHttpClient();

Request request = new Request.Builder()
        .url("https://www.example.com")
        .build();

try {
    Response response = client.newCall(request).execute();
    if (response.isSuccessful()) {
        String responseBody = response.body().string();
        System.out.println(responseBody);
    }
} catch (IOException e) {
    e.printStackTrace();
}

```

2. 异步请求

```java
OkHttpClient client = new OkHttpClient();

Request request = new Request.Builder()
        .url("https://www.example.com")
        .build();

client.newCall(request).enqueue(new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {
        e.printStackTrace();
    }

    @Override
    public void onResponse(Call call, Response response) throws IOException {
        if (response.isSuccessful()) {
            String responseBody = response.body().string();
            // 一般要JSON转数据对象
        }
    }
});

```

3. 发送POST请求  

```java
OkHttpClient client = new OkHttpClient();

RequestBody formBody = new FormBody.Builder()
        .add("username", "test")
        .add("password", "123456")
        .build();

Request request = new Request.Builder()
        .url("https://www.example.com/login")
        .post(formBody)
        .build();

client.newCall(request).enqueue(new Callback() {
    // 同上
});

// 也可以使用JSON传参
RequestBody body = RequestBody.create(
    "{\"key\":\"value\"}",
    MediaType.parse("application/json; charset=utf-8")
);

```

4. 定制化请求

```java
// 设置超时
OkHttpClient client = new OkHttpClient.Builder()
        .connectTimeout(10, TimeUnit.SECONDS)
        .readTimeout(30, TimeUnit.SECONDS)
        .writeTimeout(15, TimeUnit.SECONDS)
        .build();

// 拦截器
OkHttpClient client = new OkHttpClient.Builder()
        .addInterceptor(chain -> {
            Request newRequest = chain.request().newBuilder()
                    .addHeader("Authorization", "Bearer token")
                    .build();
            return chain.proceed(newRequest);
        })
        .build();

// 配置缓存
Cache cache = new Cache(new File("cache_directory"), 10 * 1024 * 1024);
OkHttpClient client = new OkHttpClient.Builder()
        .cache(cache)
        .build();
```

5. 请求取消

```java
OkHttpClient client = new OkHttpClient();

Request request = new Request.Builder()
        .url("https://www.example.com")
        .build();

Call call = client.newCall(request);

// 异步发起请求
call.enqueue(new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {
        if (call.isCanceled()) {
            System.out.println("请求被取消");
        } else {
            e.printStackTrace();
        }
    }

    @Override
    public void onResponse(Call call, Response response) throws IOException {
        if (response.isSuccessful()) {
            System.out.println(response.body().string());
        }
    }
});

// 某些时刻你可以取消请求
call.cancel();
```
## OkHttp中的OkHttpClient为什么建议复用，而不是每次新建？

因为OkHttpClient维护了大量复用资源（连接池、线程池、DNS、拦截器链、SSL缓存等），反复新建会浪费资源  

核心资源：  

- 连接池（ConnectionPool）：复用TCP连接，节省握手开销（尤其是HTTPS握手）  
- 线程池（Dispatcher）：每次新建Client会浪费线程资源，上下文混乱  
- SSLSession缓存、DNS缓存、拦截器、证书校验、代理等：如果每次新建，开销比较大  

# ⚙️ 核心组件与设计

## OkHttp的核心模块有哪些？（如：Dispatcher、ConnectionPool、Interceptor、RealCall）

```
OkHttpClient (配置中心)
    ↓
Dispatcher (调度中心)
    ↓
Interceptor Chain (责任链)
    ↓
  ↳ CacheInterceptor (缓存)
  ↳ RetryAndFollowUpInterceptor (重试重定向)
  ↳ ConnectInterceptor (连接复用)
    ↓
CallServerInterceptor (真正发起请求)
    ↓
ConnectionPool (连接池)
    ↓
HttpCodec / Exchange (数据IO)
```

## 说说OkHttp中的拦截器(Interceptor)的作用和分类？应用拦截器（Application Interceptor）和网络拦截器（Network Interceptor）两者的区别？

OkHttp 用了**责任链模式** 实现请求与响应全过程的处理，每个拦截器就是责任链中的一个环节，可以在任意阶段加工、修改、短路请求。

OkHttp有两类拦截器，分别是应用拦截器和网络拦截器：

- 应用拦截器：作用于应用逻辑层，关注请求整体逻辑，通常不感知重试、缓存、重定向等网络行为，执行一次

- 网络拦截器：更靠近底层网络层，能感知每次网络请求和响应，支持精确抓包、处理缓存、控制重定向  

```java
// 应用拦截器
OkHttpClient client = new OkHttpClient.Builder()
    .addInterceptor(chain -> {
        Request original = chain.request();
        Request request = original.newBuilder()
                .header("Authorization", "Bearer token")
                .build();
        return chain.proceed(request);
    })
    .build();

// 网络拦截器
OkHttpClient client = new OkHttpClient.Builder()
    .addNetworkInterceptor(chain -> {
        Response response = chain.proceed(chain.request());
        // 可以在这里做响应数据的抓取、监控
        return response;
    })
    .build();

```


| 对比点          | 应用拦截器          | 网络拦截器                                     |
| ------------ | -------------- | ----------------------------------------- |
| 触发时机         | **整个请求流程前后**   | **只在真正网络IO前后**                            |
| 是否受重试/重定向影响  | **只执行一次**      | **每次重试/重定向都执行**                           |
| 是否可见缓存       | **不可见**        | **可见**（可以拿到CacheResponse、NetworkResponse） |
| 是否拿到请求后的网络状态 | 否              | 是                                         |
| 典型应用         | 公共参数、鉴权、签名、加解密 | 文件上传下载进度、抓包、缓存控制、日志监控                     |


## OkHttp的连接复用是如何实现的？`ConnectionPool`是干嘛的？  

OkHttp通过`ConnectionPool`管理`TCP`连接复用，减少频繁建连，提高请求速度，降低CPU与网络延迟开销  

### OkHttp复用连接

HTTP请求正常流程：DNS解析->TCP建连->TLS握手->请求发送  

其中，TCP建连（三次握手）和TLS握手（加密+验证证书）最为消耗资源  

**OkHttp复用链接的核心逻辑：** 

- 同一个Host（IP+端口相同）：直接复用连接  
- HTTP/1.1:通过Keep-Alive复用  
- HTTP/2：直接支持多路复用  
- Host不同：新建连接  

### `ConnectionPool`连接池  

ConnectionPool是一个连接池调度中心，内部维护了一个连接队列，存放可复用的RealConnection，统一管理连接生命周期、超时释放、空闲清理（可配置）  


核心代码片段

```java
public class ConnectionPool {
    private final Deque<RealConnection> connections = new ArrayDeque<>();
    
    RealConnection get(...){
        for (RealConnection connection : connections) {
            if (connection.isEligible(...)) { 
            //isEligible() 负责判断：host 是否一致、协议是否兼容、是否支持多路复用等。
                return connection;
            }
        }
    }

    void put(RealConnection connection) {
        connections.add(connection);
    }
}

```

## OkHttp如何处理DNS解析、代理、SSL证书校验？

通常情况下，Android客户端无需主动改动这些，一般都是配合服务端、网络安全这些角色进行修改  

1. DNS解析  

OkHttp默认的是`Dns.SYSTEM`，通过系统默认DNS解析域名得到IP地址  

也可以自定义DNS解析：

```java
OkHttpClient client = new OkHttpClient.Builder()
    .dns(new Dns() {
        @Override
        public List<InetAddress> lookup(String hostname) throws UnknownHostException {
            // 自定义 DNS 解析逻辑（如接入 HTTPDNS）
            return myCustomDnsService.resolve(hostname);
        }
    })
    .build();
```

常用于：
- 提高域名解析速度（HTTPDNS）  
- 解决运营商劫持  
- 解决DNS缓存失效  

2. Proxy  

OkHttp支持HTTP_PROXY和SOCKS两种代理

```java
OkHttpClient client = new OkHttpClient.Builder()
    .proxy(new Proxy(Proxy.Type.HTTP, new InetSocketAddress("proxy.example.com", 8080)))
    .build();

// 代理认证：OkHttp 也支持代理服务器需要认证时，提供代理认证器：
client.newBuilder()
    .proxyAuthenticator(new Authenticator() {
        @Override
        public Request authenticate(Route route, Response response) {
            String credential = Credentials.basic("username", "password");
            return response.request().newBuilder()
                .header("Proxy-Authorization", credential)
                .build();
        }
    })
    .build();
```

3. SSL校验  

```java
// 这里的trustManager是自定义的，用于SSL证书校验
OkHttpClient client = new OkHttpClient.Builder()
    .sslSocketFactory(sslContext.getSocketFactory(), trustManager)
    .build();

client.newBuilder()
    .hostnameVerifier(...) // 自定义HostnameVerifier·
    .build();
```

TrustManager用于校验证书合法性  
HostnameVerifier校验证书里的CN或SAN是否与目标域名一致  
即便是信任证书，如果Hostname不一致，仍然会报错  

# 🧵 线程模型  

## Dispatcher的作用是什么？最大请求数、最大主机请求数默认是多少？

## Dispatcher的队列模型是怎样的？如何保证并发请求数量受控？

## OkHttp内部用到了哪些线程池？为什么这么设计？

# 🔗 网络协议支持

OkHttp支持哪些HTTP协议版本？如：HTTP/1.1、HTTP/2、HTTP/3。

HTTP/2相比HTTP/1.1在性能上有哪些优势？OkHttp做了哪些适配？

如何开启或关闭HTTP/2？

# 🚀 高级应用与扩展

如何在OkHttp中实现请求重试与失败重连？

如何实现请求缓存？OkHttp缓存策略的核心逻辑是怎样的？

如何做持久连接、心跳机制？

OkHttp如何支持WebSocket？

# 🔎 源码与底层机制  

OkHttp请求的整体流程是什么？（从newCall().execute()开始）

OkHttp为什么高效？连接复用、路由选择、拦截器链的设计亮点在哪里？

拦截器链 (Interceptor Chain) 责任链模式的具体实现逻辑？

OkHttp内部的连接复用为什么可以避免三次握手的频繁开销？

OkHttp如何实现了异步请求？异步和同步请求的区别？

ConnectionPool中的IdleConnection是如何回收的？

# 🔐 安全与稳定性 

如何配置OkHttp支持HTTPS？如何处理自签名证书？

如何处理证书校验失败的问题？

如何避免SSL握手超时？

# ⚠️ 项目实战与优化
在你项目中用过OkHttp吗？遇到过哪些坑？

OkHttp的内存泄漏场景有哪些？

怎么监控OkHttp请求的耗时、失败率、请求日志？

如何和Retrofit结合使用？两者的职责分工？

# 🧠 开放性思考题
如果让你自己设计一个精简版OkHttp，会如何设计架构？

OkHttp和其他网络库（如：HttpURLConnection、Volley、Retrofit、Cronet）相比，优缺点是什么？

