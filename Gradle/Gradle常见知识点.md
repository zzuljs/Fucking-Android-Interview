# 🧱 一、基础理解类  

## 1. Gradle 是什么？它在 Android 项目中起什么作用？

Gradle 就是 Android 项目的“大脑”和“流水线总控”，负责从代码管理到 APK编译 的整个流程 

| 组件                          | 作用              |
| --------------------------- | --------------- |
| `build.gradle`（项目级）         | 配置全局插件、仓库等      |
| `build.gradle`（模块级）         | 定义该模块的依赖、构建逻辑   |
| `settings.gradle`           | 声明项目包含的模块       |
| `gradle-wrapper.properties` | 固定使用的 Gradle 版本 |
| `buildSrc/`                 | 编写自定义插件或构建逻辑    |

Gradle 是一个基于 JVM 的构建自动化工具，被广泛用于 Android 项目的构建、管理和自动化流程中  
它最早诞生于 2007 年，设计初衷是结合 Ant 的灵活性 和 Maven 的依赖管理能力，并引入了 Groovy/Kotlin DSL 脚本语言，使构建脚本更加可编程和可扩展

## 2. 你了解 Gradle 构建生命周期（Build Lifecycle）吗？有哪些阶段？

1. 初始化阶段：执行`settings.gradle`文件，确定根项目和子项目，为每个项目创建Project实例  
2. 配置阶段：解析每个`build.gradle`文件，配置所有任务，建立任务图，不执行任务，建立依赖关系  
3. 执行阶段：执行用户定义的task，任务执行是懒加载的，未被依赖或未被指定的任务不会执行    

## 3. build.gradle 文件中常见的几个重要字段分别是什么意思？

1. 模块级build.gradle  

```groovy
plugins {
    id 'com.android.application' // 应用模块
    id 'org.jetbrains.kotlin.android' // Kotlin Android 支持
}

android {
    namespace 'com.example.myapp' // Android 12+ 推荐设置的包名
    compileSdk 34 // 编译时使用的 Android SDK 版本

    defaultConfig {
        applicationId "com.example.myapp" // 应用包名
        minSdk 24 // 最低支持 Android 版本
        targetSdk 34 // 目标 Android 版本
        versionCode 1 // 应用内部版本号
        versionName "1.0.0" // 展示给用户的版本名称

        // 向 Java/Kotlin 传递常量（在 BuildConfig 中生成）
        buildConfigField "String", "API_BASE_URL", '"https://api.example.com/"'

        // 配置 NDK 编译架构（如果用到 native 代码）
        ndk {
            abiFilters 'armeabi-v7a', 'arm64-v8a'
        }
    }

    buildTypes {
        debug {
            applicationIdSuffix ".debug" // debug 版本包名后缀
            versionNameSuffix "-debug" // 版本名称后缀
            debuggable true
        }

        release {
            minifyEnabled true // 启用代码压缩混淆
            shrinkResources true // 移除未使用的资源
            proguardFiles getDefaultProguardFile(
                'proguard-android-optimize.txt'),
                'proguard-rules.pro' // 混淆规则文件
            signingConfig signingConfigs.debug // 这里为了演示直接使用 debug 签名
        }
    }

    // 用于构建多渠道版本
    flavorDimensions "channel"
    productFlavors {
        google {
            dimension "channel"
            applicationIdSuffix ".google"
        }
        huawei {
            dimension "channel"
            applicationIdSuffix ".huawei"
        }
    }

    // 启用 Jetpack Compose（如用到的话）
    buildFeatures {
        compose true
    }

    composeOptions {
        kotlinCompilerExtensionVersion = "1.6.0"
    }

    kotlinOptions {
        jvmTarget = "1.8"
    }

    // 编译任务的 Java 版本设置
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

dependencies {
    // 核心库
    implementation 'androidx.core:core-ktx:1.12.0'
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'com.google.android.material:material:1.11.0'

    // 网络请求库
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'

    // 协程
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3'

    // Compose UI 组件
    implementation "androidx.compose.ui:ui:1.6.0"
    implementation "androidx.compose.material3:material3:1.2.0"

    // 单元测试库
    testImplementation 'junit:junit:4.13.2'

    // Android 测试库
    androidTestImplementation 'androidx.test.ext:junit:1.1.5'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.5.1'
}

```


2. 项目级build.gradle  

```groovy
// 项目构建脚本的构建依赖配置区块
buildscript {
    // 配置构建脚本本身使用的仓库，比如 Android Gradle 插件
    repositories {
        google() // Android 官方仓库
        mavenCentral() // Maven 中央仓库
    }

    // 构建脚本所需依赖（不是 app 的依赖！）
    dependencies {
        classpath 'com.android.tools.build:gradle:8.2.0' // Android 构建插件
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:1.9.23" // Kotlin 支持插件
    }
}

// Gradle 7.0 后推荐用 plugins {} 和 settings.gradle.kts 配合管理插件
// 但很多遗留项目仍然使用 buildscript + classpath 的方式

// 所有模块共用的仓库配置（也可以改为在 settings.gradle.kts 中配置）
allprojects {
    repositories {
        google()
        mavenCentral()
    }
}

// 可选：定义全局常量，供模块中使用
ext {
    kotlin_version = "1.9.23"
    compose_version = "1.6.0"
}
```


# 🧩 二、配置与脚本能力  

## 4. build.gradle 中 implementation 和 api 有什么区别？如何选择？  

implementation不会传递依赖、编译速度更快，api会传递依赖，编译速度相对慢  

如果引入三方库只需要在当前模块用，那么选择implementation  

如果还需要提供给上层模块使用，那么选择api，比如ARouter的注解

## 5. Android 项目中 buildTypes 和 productFlavors 是如何工作的？你实际用过哪些场景？  

buildTypes定义构建模式，如debug、release  
productFlavors定义渠道、版本维度，如HUAWEI、Xiaomi  

两者通常组合使用，生成build variants（构建变体）  

一个完整示例  
```groovy
android {
    compileSdk 34
    namespace "com.example.app"

    defaultConfig {
        applicationId "com.example.app"
        minSdk 24
        targetSdk 34
        versionCode 100
        versionName "1.0.0"
    }

    // 构建类型：debug / release
    buildTypes {
        debug {
            applicationIdSuffix ".debug"
            versionNameSuffix "-debug"
            debuggable true
            buildConfigField "boolean", "LOG_ENABLE", "true"
        }
        release {
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile(
                'proguard-android-optimize.txt'), 'proguard-rules.pro'
            buildConfigField "boolean", "LOG_ENABLE", "false"
            signingConfig signingConfigs.release
        }
    }

    // 渠道维度
    flavorDimensions "channel"

    // 产品渠道配置
    productFlavors {
        internal {
            dimension "channel"
            applicationIdSuffix ".internal"
            versionNameSuffix "-internal"
            buildConfigField "String", "BASE_URL", '"https://test-api.internal.local/"'
            resValue "string", "channel_name", "Internal Test"
        }
        google {
            dimension "channel"
            buildConfigField "String", "BASE_URL", '"https://api.google.com/"'
            resValue "string", "channel_name", "Google Play"
        }
        huawei {
            dimension "channel"
            buildConfigField "String", "BASE_URL", '"https://api.huawei.com/"'
            resValue "string", "channel_name", "Huawei Market"
        }
    }

    // 可选：开启 ViewBinding、Compose、其他构建功能
    buildFeatures {
        viewBinding true
    }

    // Java/Kotlin 版本设置
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = "1.8"
    }
}

```

根据flavor建立不同的资源目录  

```
src/
├── main/
│   ├── java/
│   └── res/
├── internal/
│   └── res/          ← 内部测试专属资源（如 app_name、logo）
├── google/
│   └── res/          ← Google 商店特有图标
├── huawei/
│   └── res/          ← 华为专属资源（如华为支付图标）
```

注意，flavor名称和资源目录名称，必须一致！否则不会报错，但无法生效  

## 6. 什么是 Gradle DSL？你写过哪些自定义逻辑？  

Gradle DSL是用来描述构建逻辑的专用语言  

除了声明依赖、配置SDK等常规用途，Gradle DSL还能：

| 类型        | 示例                                              |
| --------- | ----------------------------------------------- |
| 定义构建逻辑和规则 | `buildTypes`, `productFlavors`                  |
| 动态注入变量    | `buildConfigField`、`resValue`                   |
| 操作任务      | `tasks.register`, `doLast` 等                    |
| 重命名输出 APK | 自定义 `.apk` 文件名规则                                |
| 多模块编译控制   | 动态开关模块、替换依赖等                                    |
| 添加钩子逻辑    | `afterEvaluate {}`、`applicationVariants.all {}` |

一些用法举例：  

1. 自定义APK文件  
```groovy
android.applicationVariants.all { variant ->
    variant.outputs.all { output ->
        def buildType = variant.buildType.name
        def flavor = variant.flavorName
        def version = variant.versionName

        outputFileName = "myapp-${flavor}-${buildType}-v${version}.apk"
    }
}
```

2. 自定义任务  

```groovy
tasks.register("clearCache") {
    group = "custom"
    description = "清理 app 缓存目录"
    doLast {
        delete fileTree(dir: "$buildDir/intermediates")
        println "缓存已清除"
    }
}
```

3. 定义全局变量  
```groovy
ext {
    kotlin_version = "1.9.23"
    okhttp_version = "4.12.0"
}

implementation "com.squareup.okhttp3:okhttp:$okhttp_version"
```  

4. 打印构建时间  
```groovy
gradle.buildFinished { result ->
    println "🎉 Build completed with status: ${result.failure ? 'FAILED' : 'SUCCESS'}"
}
```

5. 控制模块是否参与编译  

```groovy
// 设置一个开关（你也可以从系统环境变量读取）
def enableDebugTools = false

include ':app', ':feature_login', ':feature_home'

// 动态 include debug_tools 模块
if (enableDebugTools) {
    include ':debug_tools'
    println "✅ debug_tools 模块已启用"
} else {
    println "🚫 debug_tools 模块未启用"
}
```

## 7. 你了解 gradle.properties、local.properties 的区别与用途吗？  

| 文件名                 | 用途                             | 作用范围             |
| ------------------- | ------------------------------ | ---------------- |
| `gradle.properties` | **项目级配置参数**，可用于全局变量、性能调优、开关控制、JVM或Gradle参数等 | 项目内所有模块都可访问      |
| `local.properties`  | **本地机器私有配置**，如 SDK 路径、签名文件路径等  | 只在当前开发机器使用，不建议提交 |


gradle.properties常见设置：  
```groovy
# Kotlin JVM 版本
kotlin.code.style=official
kotlin.jvmTarget=1.8

# 自定义字段
ENABLE_DEBUG_TOOLS=true
API_KEY=xxx

# 提升构建性能
org.gradle.daemon=true
org.gradle.parallel=true
org.gradle.jvmargs=-Xmx4g
```


## 8. ext 扩展字段怎么用？有没有管理依赖版本的最佳实践？  

ext是Gradle提供的扩展机制，用于build.gradle中定义自定义属性，供整个项目使用  

Project级build.gradle:  
```groovy
// Project-level build.gradle
ext {
    // SDK
    compileSdkVersion = 34
    minSdkVersion = 24
    targetSdkVersion = 34

    // 版本号统一管理
    versions = [
        kotlin      : '1.9.23',
        hilt        : '2.51',
        retrofit    : '2.9.0',
        okhttp      : '4.12.0',
        glide       : '4.16.0',
        arouter     : '1.5.2'
    ]

    // 依赖统一管理（可选）
    deps = [
        kotlinStdlib : "org.jetbrains.kotlin:kotlin-stdlib:${versions.kotlin}",
        hiltAndroid  : "com.google.dagger:hilt-android:${versions.hilt}",
        hiltCompiler : "com.google.dagger:hilt-compiler:${versions.hilt}",
        retrofit     : "com.squareup.retrofit2:retrofit:${versions.retrofit}",
        okhttp       : "com.squareup.okhttp3:okhttp:${versions.okhttp}",
        glide        : "com.github.bumptech.glide:glide:${versions.glide}",
        arouterApi   : "com.alibaba:arouter-api:${versions.arouter}",
        arouterApt   : "com.alibaba:arouter-compiler:${versions.arouter}",
    ]
}
```

模块级build.gradle：

```groovy
android {
    compileSdk rootProject.ext.compileSdkVersion
    defaultConfig {
        minSdk rootProject.ext.minSdkVersion
        targetSdk rootProject.ext.targetSdkVersion
    }
}

dependencies {
    implementation rootProject.ext.deps.kotlinStdlib
    implementation rootProject.ext.deps.hiltAndroid
    kapt         rootProject.ext.deps.hiltCompiler

    implementation rootProject.ext.deps.okhttp
    implementation rootProject.ext.deps.retrofit
    implementation rootProject.ext.deps.glide

    implementation rootProject.ext.deps.arouterApi
    kapt         rootProject.ext.deps.arouterApt
}
```

Gradle 7.0+版本提供了更高级的方式：`libs.versions.toml`

```groovy
# gradle/libs.versions.toml
[versions]
kotlin = "1.9.23"
retrofit = "2.9.0"

[libraries]
retrofit = { module = "com.squareup.retrofit2:retrofit", version.ref = "retrofit" }
```

# 🧰 三、自定义能力与优化实践  

## 9. 你写过 Gradle 插件吗？请谈谈实现思路和使用方式。
follow-up：Gradle 插件和 Task 是什么关系？

follow-up：如何为公司定制公共构建逻辑？

## 10. Gradle 是如何处理任务依赖关系的？你是否手动定义过 Task 的执行顺序？

Gradle的任务（Task）构建最小执行单元，每个Task可以声明依赖的其他Task，任务依赖构成了一个有向无环图（DAG），Gradle根据DAG自动计算任务执行顺序，保证所有依赖任务先执行，再执行当前任务  

```groovy
// 任务定义
task taskA {
    doLast {
        println "执行任务 A"
    }
}

task taskB {
    doLast {
        println "执行任务 B"
    }
}

// 方式1：用 dependsOn 显式声明依赖
taskB.dependsOn(taskA)

// 方式2：在定义时声明
task taskC(dependsOn: 'taskB') {
    doLast {
        println "执行任务 C"
    }
}
```

Gradle 执行时，会保证 taskA -> taskB -> taskC 顺序执行  

一个更全面的例子  

```groovy
task clean(type: Delete) {
    delete rootProject.buildDir
}

task generateSources {
    doLast { println "生成代码" }
}

task compileJava {
    doLast { println "编译Java代码" }
    dependsOn generateSources          // 编译前必须先生成代码
    mustRunAfter clean                 // 编译任务必须在clean之后执行（但不依赖clean）
}

compileJava.finalizedBy {
    println "编译后执行的任务，比如代码检查"
}
```


## 11. 有遇到过构建速度很慢的问题吗？你是如何定位和优化构建性能的？
follow-up：了解 configuration phase vs execution phase 吗？

follow-up：--profile 和 build scan 用过吗？

## 12. 你是否使用过 buildSrc 模块？它的作用是什么？

# 🌐 四、多人协作与CI集成

## 13. 你如何管理多模块 Gradle 项目？是否做过组件化、插件化相关实践？


## 14. 你是如何在 CI/CD 环境中使用 Gradle 构建 Android 项目的？
follow-up：有使用过 Gradle Wrapper 吗？为什么推荐使用？

# 🧪 五、进阶与细节探索

## 15. Gradle 是如何与 Kotlin DSL 配合的？你更倾向使用 Groovy 还是 Kotlin 脚本？为什么？


## 16. Gradle 中的 Configuration Avoidance API 是什么？为什么推荐用 register() 而不是 create()？


## 17. 了解 Gradle 的增量构建原理吗？什么情况会导致构建缓存失效？


## 18. Gradle 依赖冲突怎么解决？有没有踩过坑？用过 resolutionStrategy 吗？

# 💬 六、开放性问题（用于评估候选人经验）

## 19. 请谈谈你在实际工作中遇到过的最复杂的 Gradle 使用场景，并说说你是怎么解决的？


## 20. 如果你接手一个构建时间超过 3 分钟的 Android 项目，你会怎么排查、优化？
