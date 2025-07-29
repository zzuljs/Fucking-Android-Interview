# 一、基本原理与使用

## Retrofit 的基本使用流程是什么？

假设请求接口：https://api.github.com/users/octocat 

```json
{
  "login": "octocat",
  "id": 583231,
  "name": "The Octocat",
  "company": "@github"
}
```

这个接口返回的字段非常多，我们仅访问以上4个字段

1. 添加依赖  
```groovy
implementation 'com.squareup.retrofit2:retrofit:2.9.0'
implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
```

2. 定义数据模型

```java
public class GitHubUser {
    public String login;
    public int id;
    public String name;
    public String company;
}
```

3. 定义API接口

```java
public interface GitHubApiService {
    @GET("users/{username}")
    Call<GitHubUser> getUser(@Path("username") String username);
}
```

4. 创建Retrofit实例  
```java
Retrofit retrofit = new Retrofit.Builder()
        .baseUrl("https://api.github.com/")
        .addConverterFactory(GsonConverterFactory.create())
        .build();

GitHubApiService service = retrofit.create(GitHubApiService.class);
```

5. 发起请求

```java
Call<GitHubUser> call = service.getUser("octocat");

call.enqueue(new Callback<GitHubUser>() {
    @Override
    public void onResponse(Call<GitHubUser> call, Response<GitHubUser> response) {
        if (response.isSuccessful()) {
            GitHubUser user = response.body();
            Log.d("Retrofit", "用户名: " + user.login + ", 公司: " + user.company);
        } else {
            Log.e("Retrofit", "请求失败，状态码: " + response.code());
        }
    }

    @Override
    public void onFailure(Call<GitHubUser> call, Throwable t) {
        Log.e("Retrofit", "网络请求失败", t);
    }
});

```
## Retrofit 和 OkHttp 之间是什么关系？

Retrofit 站在 OkHttp 之上，运用了动态代理、注解处理、反射、工厂模式等高阶编程技术，核心目的是抽象出业务接口，隐藏请求细节，实现接口声明即请求逻辑  

OkHttp专注于网络请求，是一个全能的网络库，Retrofit独立使用，底层的网络请求也是通过OkHttp使用的，这两个库都是Square公司的开源库，分工不同  

举例一：OkHttp单独使用  

这是一个最简单的网络请求，没有请求参数，没有解析响应
```java
OkHttpClient client = new OkHttpClient();

Request request = new Request.Builder()
        .url("https://api.example.com/users?id=123")
        .build();

try (Response response = client.newCall(request).execute()) {
    if (response.isSuccessful()) {
        String responseBody = response.body().string();
    }
} catch (IOException e) {
    e.printStackTrace();
}

```

举例二：OkHttp+Retrofit  

```java
interface ApiService {
    @GET("users")
    Call<UserResponse> getUsers(@Query("id") int id);
}


Retrofit retrofit = new Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .addConverterFactory(GsonConverterFactory.create())
            .build();

ApiService apiService = retrofit.create(ApiService.class);
Call<LoginResponse> call = apiService.getUsers(123);

try {
    LoginResponse response = call.execute().body();
} catch (IOException e) {
    e.printStackTrace();
}

```

Retrofit的基本调用链路是：
OkHttpClient --> Call.Factory --> Retrofit 动态代理 --> 接口API   

通常，接口API是调用者需要关注的业务核心  

## Retrofit 的注解（@GET、@POST、@Query、@Body、@Path、@Header 等）都有什么作用？

Retrofit的注解常用于方法注解和参数注解  

### 方法注解：定义请求类型和路径  

| 注解         | 含义         | 示例                      |
| ---------- | ---------- | ----------------------- |
| `@GET`     | GET 请求     | `@GET("users")`         |
| `@POST`    | POST 请求    | `@POST("login")`        |
| `@PUT`     | PUT 请求     | `@PUT("users/{id}")`    |
| `@DELETE`  | DELETE 请求  | `@DELETE("users/{id}")` |
| `@PATCH`   | PATCH 请求   | `@PATCH("users/{id}")`  |
| `@HEAD`    | HEAD 请求    | `@HEAD("users")`        |
| `@OPTIONS` | OPTIONS 请求 | `@OPTIONS("users")`     |


### 参数注解：绑定参数到请求各部分  

这里的参数是指URL请求参数  

| 注解      | 作用         | 示例                                                               |
| ------- | ---------- | ---------------------------------------------------------------- |
| `@Path` | 替换 URL 占位符 | `@GET("users/{id}")`<br>`Call<User> getUser(@Path("id") int id)` |
| `@Query`    | 单个 query 参数 | `@GET("search")`<br>`Call<List<User>> search(@Query("name") String name)` |
| `@QueryMap` | 批量 query 参数 | `@QueryMap Map<String, String> map`                                       |
| `@Header`    | 动态添加单个 Header  | `@Header("Authorization") String token`  |
| `@HeaderMap` | 动态添加多个 Headers | `@HeaderMap Map<String, String> headers` |
| `@Body` | 序列化整个对象为请求体 | `@POST("login")`<br>`Call<LoginResponse> login(@Body LoginRequest request)` |

### 一个使用Retrofit注解的例子

一个电商系统，接口包括：

- 获取商品详情  
- 搜索商品  
- 用户登录  
- 创建订单  


1. 定义数据类

```java
// 登录请求
class LoginRequest {
    String username;
    String password;

    public LoginRequest(String username, String password) {
        this.username = username;
        this.password = password;
    }
}

// 登录响应
class LoginResponse {
    String token;
    String userId;
}

// 商品详情
class Product {
    String id;
    String name;
    double price;
    String description;
}

// 搜索结果
class SearchResponse {
    List<Product> products;
}

// 下单请求
class OrderRequest {
    String userId;
    List<String> productIds;
}

// 下单响应
class OrderResponse {
    String orderId;
    String status;
}

```

2. 定义Retrofit接口  
```java
public interface ECommerceApi {

    // 登录：POST + Body
    @POST("user/login")
    Call<LoginResponse> login(@Body LoginRequest request);

    // 获取商品详情：GET + Path
    @GET("product/{id}")
    Call<Product> getProductDetail(@Path("id") String productId);

    // 搜索商品：GET + Query
    @GET("product/search")
    Call<SearchResponse> searchProducts(
        @Query("keyword") String keyword,
        @Query("page") int page,
        @Query("size") int size
    );

    // 创建订单：POST + Header + Body
    @POST("order/create")
    Call<OrderResponse> createOrder(
        @Header("Authorization") String token,
        @Body OrderRequest request
    );
}

```

3. 创建Retrofit实例  
```java

public class RetrofitClient {

    private static final String BASE_URL = "https://api.example.com/";

    public static ECommerceApi create() {
        // 配置 OkHttp 拦截器（用于日志、调试）
        HttpLoggingInterceptor loggingInterceptor = new HttpLoggingInterceptor();
        loggingInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY);

        OkHttpClient okHttpClient = new OkHttpClient.Builder()
            .addInterceptor(loggingInterceptor)
            .build();

        // Retrofit 配置
        Retrofit retrofit = new Retrofit.Builder()
            .baseUrl(BASE_URL)
            .client(okHttpClient)
            .addConverterFactory(GsonConverterFactory.create())  // 序列化/反序列化
            .build();

        return retrofit.create(ECommerceApi.class);
    }
}

```

4. 调用  
```java
public class Main {

    public static void main(String[] args) {
        ECommerceApi api = RetrofitClient.create();

        // 登录
        LoginRequest loginRequest = new LoginRequest("jason", "123456");
        try {
            LoginResponse loginResponse = api.login(loginRequest).execute().body();
            System.out.println("登录成功，Token: " + loginResponse.token);

            // 获取商品详情
            Product product = api.getProductDetail("123").execute().body();
            System.out.println("商品名称：" + product.name);

            // 搜索商品
            SearchResponse searchResponse = api.searchProducts("手机", 1, 10).execute().body();
            searchResponse.products.forEach(p -> System.out.println("搜索结果：" + p.name));

            // 下单
            OrderRequest orderRequest = new OrderRequest();
            orderRequest.userId = loginResponse.userId;
            orderRequest.productIds = List.of("123", "456");

            OrderResponse orderResponse = api.createOrder("Bearer " + loginResponse.token, orderRequest)
                    .execute().body();
            System.out.println("订单创建成功，订单ID：" + orderResponse.orderId);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```
## @FormUrlEncoded 和 @Multipart 有什么区别？  

Retrofit 通过注解标记请求体编码类型，@FormUrlEncoded 适用于纯表单参数，内部以 URL 编码序列化，而 @Multipart 适用于文件上传，内部基于 OkHttp 的 MultipartBody 动态组织每个 part，支持复杂多媒体数据传输

### @FormUrlEncoded 👉 纯文本表单提交（URL编码格式）
```java
@FormUrlEncoded
@POST("user/login")
Call<LoginResponse> login(
    @Field("username") String username,
    @Field("password") String password
);

```
实际发送内容：
```
Content-Type: application/x-www-form-urlencoded

username=jason&password=123456
```

### @Multipart 👉 多部分表单（支持文件）  
```java
@Multipart
@POST("user/upload")
Call<UploadResponse> uploadFile(
    @Part MultipartBody.Part file,
    @Part("description") RequestBody description
);

```
实际发送内容：
```
Content-Type: multipart/form-data; boundary=xxx

--xxx
Content-Disposition: form-data; name="file"; filename="avatar.jpg"
Content-Type: image/jpeg

<二进制文件数据>

--xxx
Content-Disposition: form-data; name="description"

头像上传

--xxx--
```


# 二、设计思想

## Retrofit 的核心设计思想是什么？为什么说它用了“面向接口编程”？

Retrofit 核心设计思想：用注解+接口描述 HTTP API，通过动态代理生成请求逻辑，屏蔽底层网络细节，提升可维护性与可扩展性

开发者只需要定义接口，不需要关心请求的底层实现。

接口方法通过注解（如 @GET、@POST、@Body、@Query 等）声明请求类型、参数、路径、头信息等

## Retrofit 是如何将接口方法动态转换为网络请求的？

Retrofit 本质上是：
用动态代理捕获方法调用 ——> 反射分析注解 ——> 构造请求 ——> 交由 OkHttp 执行

```
Retrofit
  └── create() -> 动态代理 Proxy
           └── InvocationHandler
                   └── ServiceMethod
                           ├── 解析注解
                           ├── 拼接URL
                           ├── 组装Request
                   └── OkHttpCall
                           └── 交给OkHttp执行请求

```

## Retrofit 的动态代理是如何实现的？用到了哪些Java基础知识？

Retrofit 利用 Java JDK 原生的动态代理（java.lang.reflect.Proxy）和反射（java.lang.reflect.Method、注解处理）机制，实现了接口方法到网络请求的动态转换  

相关源码：  

```java
public <T> T create(final Class<T> service) {
    // 校验接口是否合法 (是否为接口，是否包含注解)
    validateServiceInterface(service);

    // 核心：通过 JDK 动态代理创建接口实现
    return (T) Proxy.newProxyInstance(
            service.getClassLoader(),
            new Class<?>[]{service},
            new InvocationHandler() {
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) {
                    // 方法调用时会进入这里
                    ...
                }
            });
}


// 接口调用本质上触发的是InvocationHandler.invoke()方法 ，这里通过解析注解+反射来创建具体的OkHttpCall对象，从而生成执行网络请求的对象
public Object invoke(Object proxy, Method method, Object[] args) {
    // 处理默认方法 (JDK 8 的 default interface method)
    if (method.isDefault()) { ... }

    // 核心逻辑：解析注解，生成 ServiceMethod
    ServiceMethod serviceMethod = loadServiceMethod(method);

    // 创建实际的 Call 对象
    OkHttpCall call = new OkHttpCall(serviceMethod, args);
    return call;
}

```

# 三、底层实现细节
Retrofit 的整体架构是怎样的？涉及哪些核心类？（如：Retrofit、ServiceMethod、OkHttpCall、CallAdapter 等）

Retrofit 的请求流程可以简要描述一下吗？从接口调用到最终发送请求。

Retrofit 如何实现参数注解到请求的转换？@Query、@Path 是如何拼接 URL 的？

Retrofit 中的 Converter 是如何工作的？常见的有哪些 Converter？

Retrofit 的 CallAdapter 有什么作用？它在异步和同步调用中扮演了什么角色？

# 四、与协程和 RxJava 的集成

## Retrofit 如何支持协程（suspend function）？背后的实现机制是什么？

Retrofit 在接口解析时检测 suspend 函数，利用 CallAdapter 将 Call<T> 自动转换为支持协程挂起的实现，内部使用 suspendCancellableCoroutine 做桥接，自身并不依赖协程库，也不依赖任何协程能力  

Retrofit 本身只根据API定义来构造对象，不使用对象具体能力，所以API可以被声明为协程，最终使用过程在调用者手中，调用者自身保证协程调度的安全性就可以了  

## Retrofit 如何支持 RxJava？需要注意哪些坑？


### Retrofit使用RxJava

Retrofit通过RxJava2CallAdapterFactory实现对RxJava的支持

```kotlin
// 需要显示适配RxJava
Retrofit.Builder()
    .baseUrl(BASE_URL)
    .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
    .build()

// RxJava线程调度
Observable.create(emitter -> {
    call.enqueue(new Callback<T>() {
        onResponse: emitter.onNext(response.body())
        onFailure: emitter.onError(throwable)
    })
})

```

把 OkHttp 的 Call 包装成 RxJava 流式类型

内部通过 Call.enqueue() 异步执行请求，把 onResponse、onFailure 回调转换成 RxJava 的 onNext()、onError()、onComplete()

### RxJava注意事项  

- Retrofit默认执行Call是同步的，即OkHttp的线程池，RxJava只负责将其包装成流，不会自动切线程，所以需要显示指定线程
- 生命周期管理，RxJava+Retrofit最好结合Lifecycle组件管理生命周期，否则可能内存泄漏

## 如果 Retrofit 协程请求失败，你一般如何做异常捕获与重试？

可以与协程搭配使用, 考虑封装一个重试函数：  

```kotlin
suspend fun <T> retryWithDelay(
    times: Int = 3,
    initialDelay: Long = 1000L,
    maxDelay: Long = 3000L,
    factor: Double = 2.0,
    block: suspend () -> T
): T {
    var currentDelay = initialDelay

    repeat(times - 1) {
        try {
            return block()
        } catch (e: IOException) {
            // 只在网络异常时重试
        }
        delay(currentDelay)
        currentDelay = (currentDelay * factor).toLong().coerceAtMost(maxDelay)
    }
    return block() // 最后一次尝试
}
```

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