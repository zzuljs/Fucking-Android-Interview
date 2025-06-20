# 🧠 一、基础概念类问题

## 什么是Hilt？基本使用方法有哪些？

1. 添加依赖

在gralde/libs.versions.toml中新增配置  

```groovy
# gradle/libs.versions.toml

[versions]
hilt = "2.48" # 定义Hilt的版本号，请使用最新的稳定版本

[libraries]
# Hilt 核心库
hilt-android = { module = "com.google.dagger:hilt-android", version.ref = "hilt" }
# Hilt 注解处理器
hilt-compiler = { module = "com.google.dagger:hilt-android-compiler", version.ref = "hilt" }
# Hilt 对 ViewModel/Navigation 的支持 (可选，如果使用)
androidx-hilt-navigation-fragment = { module = "androidx.hilt:hilt-navigation-fragment", version = "1.2.0" } # 请使用最新版本
androidx-hilt-compiler = { module = "androidx.hilt:hilt-compiler", version = "1.2.0" } # 对应上面的组件

[plugins]
# Hilt Gradle 插件
hilt-android = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
```
添加依赖，Hilt的引入与其他库不同，需要先在根目录build.gradle添加依赖  

```groovy
// Top-level build file where you can add configuration options common to all sub-projects/modules.
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.compose) apply false
    alias(libs.plugins.android.library) apply false
    // 添加这一行来声明Hilt Gradle 插件
    alias(libs.plugins.hilt.android) apply false // 这里引用了 libs.versions.toml 中 plugins 下的 hilt-android
}
```

然后在应用模块build.gradle增加依赖：

```groovy
// :base 模块的 build.gradle 文件

plugins {
    // 确保这里包含了 'kotlin-kapt'
    id 'kotlin-kapt' // 如果是纯Kotlin模块
    // 或者 id 'org.jetbrains.kotlin.android' 和 id 'kotlin-kapt' 如果是Android library模块
    // 比如：
    id 'com.android.library' // 如果是 Android 库模块
    id 'org.jetbrains.kotlin.android'
    id 'kotlin-kapt' // 这一行至关重要！

    // 如果Hilt也要在这个模块使用，也要加上
    id 'com.google.dagger.hilt.android'
}

android {
    // ... android 配置 ...
}

dependencies {
    // ... 其他依赖 ...

    // Hilt 注解处理器
    kapt(libs.hilt.compiler) // 您的错误就是出在这里，因为它找不到kapt方法
    kapt(libs.androidx.hilt.compiler) // 如果使用Hilt的androidx扩展

    // ... 其他 kapt 依赖（如果有的话）
    // kapt(libs.room.compiler) // 例如 Room 的注解处理器
}
```

2. 标记Application类  

标记Application类是Hilt框架的初始化，如果没有正确标记Application，那么Hilt将无法正确使用  
```kotlin
@HiltAndroidApp
class MainApplication:Application(){

}
```

3. 标记Android组件

Hilt可以为Android生命周期类注入依赖，使用`@AndroidEntryPoint`注解标记需要进行依赖注入的`Activity`、`Fragment`、`View`、`Service`或`BroadcastReceiver`

```kotlin
// MainActivity.kt

@AndroidEntryPoint // 标记此Activity将通过Hilt进行依赖注入
class MainActivity : AppCompatActivity() {

    // 假设我们要注入一个 MyRepository
    @Inject // Hilt 会找到如何提供 MyRepository 的实例
    lateinit var myRepository: MyRepository

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 现在可以使用注入的 myRepository
        myRepository.doSomething()
    }
}
```

4. 提供依赖  

Hilt需要知道如何创建注入类的实例，有两种方式：  

- 构造函数注入（Constructor Injection）

如果一个类没有依赖，或者其所有依赖都可以通过Hilt的其他方式提供，那么只需要用`@Inject`注解标记其构造函数即可  

```kotlin
// MyRepository.kt
@Singleton // 标记为单例，整个应用只有一个实例
class MyRepository @Inject constructor(
    private val apiService: ApiService // ApiService 也会由 Hilt 提供
) {
    fun doSomething() {
        println("MyRepository is doing something with ${apiService.getApiEndpoint()}")
    }
}

// ApiService.kt
class ApiService @Inject constructor() {
    fun getApiEndpoint(): String {
        return "https://api.example.com"
    }
}
```

- 模块绑定  

有些时候无法通过构造函数注入，比如：接口与实现类的绑定、不拥有源代码第三方类库、需要特殊构建过程的类（如Retrofit实例）  

此时使用Hilt模块来提供这些依赖，模块是使用`@Module`注解的类，包含了`@Provides`或`@Binds`注解的方法  

```kotlin
// NetworkModule.kt

@Module // 标记这是一个Hilt模块
@InstallIn(SingletonComponent::class) // 告诉Hilt这个模块的依赖在Application生命周期内可用 (单例)
object NetworkModule { // 使用object或class都可以

    @Singleton // 确保OkHttpClient是单例
    @Provides // 提供OkHttpClient实例
    fun provideOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder()
            // .addInterceptor(...)
            .build()
    }

    @Singleton // 确保Retrofit是单例
    @Provides // 提供Retrofit实例
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .client(okHttpClient)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }

    @Singleton
    @Provides // 提供ApiService的实例 (如果ApiService没有@Inject构造函数)
    fun provideApiService(retrofit: Retrofit): ApiService {
        return retrofit.create(ApiService::class.java)
    }
}

// 假设ApiService是一个接口
interface ApiService {
    // ... API 定义 ...
}

// 假设MyApiServiceImpl是ApiService的实现
class MyApiServiceImpl @Inject constructor() : ApiService {
    override fun getApiEndpoint(): String {
        return "Real API Endpoint"
    }
}

@Module
@InstallIn(SingletonComponent::class)
abstract class ServiceBindingModule { // 使用abstract class for @Binds
    @Binds // 将ApiService接口绑定到MyApiServiceImpl实现
    @Singleton
    abstract fun bindApiService(impl: MyApiServiceImpl): ApiService
}
```
`@Provides`:用于提供类的实例  
`@Binds`：用于将接口绑定到现实类（更高效、因为Hilt不会生成额外的工厂类）
`@InstallIn`：指定模块生命周期，如`SingletonComponent::class`表示与Application绑定

5. 注入ViewModel  

Hilt提供了对Jetpack ViewModel的支持

```kotlin
// MyViewModel.kt
@HiltViewModel // 标记这是一个Hilt ViewModel
class MyViewModel @Inject constructor(
    private val myRepository: MyRepository // MyRepository 由 Hilt 注入
) : ViewModel() {

    fun loadData() {
        myRepository.doSomething()
    }
}

// MainActivity.kt 或 MyFragment.kt
class MainActivity : AppCompatActivity() {
    private val viewModel: MyViewModel by viewModels() // Hilt 会自动提供 ViewModelFactory

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        viewModel.loadData()
    }
}
```

## Hilt和Dagger是什么关系？

Hilt 是 Dagger 在 Android 平台上的一个“包装”或“扩展”。 它利用了 Dagger 的强大功能，但通过一系列预设的组件、作用域和自动化生成，极大地减少了在 Android 中使用 Dagger 所需的模板代码和复杂配置

## Hilt相对于Dagger有哪些简化和封装？

1. 自动化Android核心组件注入 @AndroidEntryPoint  
2. 预定义Android生命周期绑定的作用域
3. 简化模块安装机制  
4. 对ViewModel的原生支持  

# 🧱 二、注解与作用域相关问题

## @Inject 有什么作用？  

@Inject 是让依赖的构建过程“自动化”，从而解耦类之间的“构建”和“使用”逻辑  

常用于构造函数和属性  

```kotlin
class UserRepository @Inject constructor(
    private val apiService: ApiService
)

@AndroidEntryPoint
class MainActivity : AppCompatActivity() {

    @Inject
    lateinit var userRepository: UserRepository

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // 此时 userRepository 已经自动注入
    }
}
```

## @HiltAndroidApp 的作用是什么？为什么必须加在 Application 上？

## @AndroidEntryPoint 有什么用？可以加在哪些类上？为什么 Activity/Fragment 都要加这个注解？

## @Inject 和 @Provides 的区别是什么？使用场景分别是什么？

## @Singleton 的作用是什么？在Hilt中是如何实现单例的？

## @ActivityRetainedScoped 是什么？和 @ViewModelScoped 区别是什么？

## Hilt 中有哪些常见的作用域注解？能否举例说明它们适用于哪些组件？

## Hilt 提供的作用域是线程安全的吗？

# 🧰 三、模块与绑定相关问题
## Hilt 中的 Module 是什么？为什么需要它？

## @InstallIn 注解的作用是什么？必须要指定安装在哪个 Component 吗？

## @Binds 和 @Provides 有什么区别？什么时候用 @Binds？

## 如何将接口与实现类进行绑定？Hilt 是如何知道要注入哪个实现的？

## 你能手写一个用 Hilt Module 提供 Retrofit 实例的例子吗？

# 🔄 四、组件与生命周期集成
## Hilt 支持哪些 Android 组件的依赖注入？

## Fragment 和 ViewModel 中的依赖注入是如何实现的？有什么限制？

## 为什么 ViewModel 中不能直接使用 @Inject 构造函数注入？应该怎么做？

## Hilt 中 ViewModel 的注入为什么必须配合 @HiltViewModel？这个注解做了什么？

# ⚙️ 五、底层原理与编译器机制
## Hilt 是如何工作的？底层用到了哪些技术？（例如：注解处理器、代码生成）

## 你能描述一下编译期 Hilt 是如何生成代码并注入依赖的吗？

## Component 是什么？Hilt 中有哪些 Component？这些 Component 有什么层级关系？

## Hilt 是如何处理作用域之间的依赖传递的？

## 你了解 @GeneratedEntryPoint 和 Hilt 的 codegen 机制吗？

# 🔍 六、进阶用法与技巧
## 如何为自定义的类（比如工具类、协程调度器）注入依赖？

## 如何注入带参数的构造函数？Hilt 支持 Assisted Injection 吗？

## 如何使用 Qualifier 区分不同类型的依赖？举一个你用过的例子。

## 如果我需要注入多个相同类型的依赖，如何处理？（例如多个 Retrofit 实例）

## Hilt 是否支持多模块项目？在多模块项目中应该如何配置依赖注入？

# 🧪 七、测试与调试相关问题
## 如何在单元测试中使用 Hilt？和正常开发有什么不同？

## 如何在 Instrumentation Test 中注入测试依赖？用到哪些注解？

## @TestInstallIn 和 @UninstallModules 是什么？怎么使用？

## 你在使用 Hilt 进行测试时遇到过什么问题？是怎么解决的？

# 🆚 八、与其他依赖注入框架对比
## Hilt 与 Dagger2 有哪些异同？为什么 Hilt 更适合 Android 项目？

## Hilt 和 Koin 的对比你怎么看？为什么选择 Hilt？

## 你是否用过 Dagger 原生写 Component 的方式？和 Hilt 有哪些显著不同？

# 📦 九、实际项目与经验问题
## 你在项目中是如何引入和使用 Hilt 的？是否从 Dagger 迁移过来？

## 你在使用 Hilt 时遇到过哪些坑？是怎么解决的？

## 有没有遇到 Hilt 无法注入或编译报错的情况？怎么排查？

## 你会在 Hilt 中注入 View 吗？为什么不建议这么做？

## 你如何保证注入的对象在合适的生命周期范围内被销毁？

## Hilt 对 App 启动速度有没有影响？有没有优化方法？

## 🧩 十、开放性问题（思维延伸）
## 如果你要在项目中实现一套简化的依赖注入框架，你会如何设计？借鉴 Hilt 哪些思想？

## Hilt 是否可以用于非 Android 项目？如果不可以，为什么？

## 你觉得在小型项目中用 Hilt 是不是有点“杀鸡用牛刀”？你怎么看？

## 你怎么看待依赖注入的“滥用”问题？Hilt 是否会降低代码可读性？