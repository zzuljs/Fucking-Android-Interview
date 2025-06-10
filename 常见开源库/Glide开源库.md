# 🧩 一、基础使用（用于筛查是否熟悉常用API）

## Glide 和 Picasso 有哪些区别？你为什么选 Glide？

Glide和Picasso的区别：

1. 对GIF的支持：Glide支持Gif图片，而Picasso天然不支持Gif图片
2. 生命周期集成：具有生命周期集成功能，与Android开发理念高度契合，如在Activity中加载、销毁图片，Glide能够很好的管理图片的生命周期，避免不必要的内存占用和泄露  
3. 内存管理和性能：Glide的双层缓存策略，能够提高资源复用和IO性能，适用于列表快速滚动、资源频繁创建和销毁等Android常见场景  
4. 丰富的图片变化功能：Glide提供了更丰富的图片变换功能(如圆角、灰度、模糊等)    

## Glide基本用法

### 用 Glide 加载一张图片到 ImageView 的基本写法？

引入Glide库
```java
dependencies {
    implementation 'com.github.bumptech.glide:glide:4.16.0' // Glide API
    annotationProcessor 'com.github.bumptech.glide:compiler:4.16.0' // Glide编译器，提供功能支撑
}
```

```java
Glide.with(context)    // 自带生命周期管理功能，但需要注意context不能为null
     .load(imageUrl)   // 网络图片，也可以是本地图片
     .placeholder(R.drawable.ic_placeholder) // 替换为你的占位符图片资源
     .error(R.drawable.ic_error)       // 替换为你的错误图片资源
     .into(myImageView);
```
### 如何加载 GIF 动图？如何禁用动画？

gif基本加载
```java
    String gifUrl = "https://media.giphy.com/media/l0HlCLiCq0Q/giphy.gif"; // GIF URL

    // 方式一：最简单的方式，Glide 会自动检测并播放 GIF
    Glide.with(this)
         .load(gifUrl)
         .into(gifImageView);

    // 方式二：明确告诉 Glide 你期望加载一个 GIF (GifDrawable)，更具类型安全性
    // 这种方式在你想对 GifDrawable 进行额外操作时很有用，例如控制播放次数等
    Glide.with(this)
         .asGif() // 明确指定加载为 GifDrawable
         .load(gifUrl)
         .into(gifImageView);
```

禁用动画
```java
String gifUrl = "https://media.giphy.com/media/l0HlCLiCq0Q/giphy.gif";

// 方法一：dontAnimate禁用动画
Glide.with(this)
     .load(gifUrl)
     .dontAnimate() // 禁用所有动画，包括 GIF 动画
     .into(gifImageView);

// 方法二：作为Bitmap展示，此时仅展示第一帧
Glide.with(this)
     .asBitmap() // 强制将图片解码为 Bitmap，即使它是 GIF
     .load(gifUrl)
     .into(gifImageView);
```

### 如何设置占位图、错误图、加载失败重试？
```java
private int retryCount = 0;
private final int MAX_RETRIES = 3; // 最大重试次数

private void loadImageWithRetry(final String imageUrl, final ImageView imageView) {
    Glide.with(this)
         .load(imageUrl)
         .placeholder(R.drawable.ic_placeholder)
         .error(R.drawable.ic_error)
         .listener(new RequestListener<Drawable>() { // 监听请求结果
             @Override
             public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Drawable> target, boolean isFirstResource) {
                 Log.e("Glide", "Image load failed: " + e.getMessage());
                 if (retryCount < MAX_RETRIES) {
                     retryCount++;
                     Log.d("Glide", "Retrying image load... (Attempt " + retryCount + ")");
                     // 重新发起加载请求
                     loadImageWithRetry(imageUrl, imageView);
                     return true; // 返回 true 表示我们已经处理了异常，不再执行默认的错误处理（即显示错误图）
                 }
                 // 如果重试次数用完，或者不是可重试的错误，则显示错误图
                 Toast.makeText(MainActivity.this, "图片加载失败，请检查网络", Toast.LENGTH_SHORT).show();
                 return false; // 返回 false，让 Glide 显示错误图
             }

             @Override
             public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, DataSource dataSource, boolean isFirstResource) {
                 retryCount = 0; // 加载成功，重置重试计数器
                 return false; // 返回 false，让 Glide 执行默认的资源显示逻辑
             }
         })
         .into(imageView);
}

// 在 onCreate 或其他地方调用
// loadImageWithRetry(imageUrl, myImageView);
```

### 如何实现图片圆角、圆形、模糊、灰度等效果？

Glide 提供了多种图片变换（Transformation）功能，可以轻松实现圆角、圆形、模糊、灰度等效果

1. 圆角  

```java
// 设置圆角半径，单位是像素
RequestOptions requestOptions = new RequestOptions()
        .transform(new RoundedCorners(16)); // 16dp 的圆角

Glide.with(this)
     .load(imageUrl)
     .apply(requestOptions) // 应用 RequestOptions
     .into(roundedImageView);

```

2. 圆形  

```java
// 应用圆形裁剪
RequestOptions requestOptions = new RequestOptions().transform(new CircleCrop());

Glide.with(this)
     .load(imageUrl)
     .apply(requestOptions)
     .into(circleImageView);
```

3. 模糊  

模糊需要再引入一个三方库 `glide-transformations`：
```groovy
dependencies {
    implementation 'jp.wasabeef:glide-transformations:4.3.0'
}
```
使用 `BlurTransformation`
```java
// 模糊半径 (radius) 和采样率 (sampling)
// radius 越大越模糊，sampling 越大性能越高但模糊效果可能稍差
RequestOptions requestOptions = new RequestOptions()
        .transform(new BlurTransformation(25, 3)); // 模糊半径 25，采样率 3

Glide.with(this)
     .load(imageUrl)
     .apply(requestOptions)
     .into(blurImageView);
```

4. 灰度  

灰度同样依赖 `glide-transformations`
```java
// 应用灰度变换
RequestOptions requestOptions = new RequestOptions()
                .transform(new GrayscaleTransformation());

Glide.with(this)
     .load(imageUrl)
     .apply(requestOptions)
     .into(grayscaleImageView);
```
## 你如何控制 Glide 的缓存策略？

### 什么是 `DiskCacheStrategy`？

### 如何只从内存加载，不使用磁盘缓存？

### 如何监听图片加载结果（成功/失败）？

如何给 RecyclerView 加载大量图片时防止卡顿或错位？    

🔍 二、进阶机制（用于判断是否理解 Glide 的内部设计）
Glide 的图片加载流程是怎样的？（大致从调用到显示的过程）

Glide 的内存缓存和磁盘缓存是如何工作的？使用了哪些数据结构？

Glide 是如何实现缓存 key 的？能手动指定吗？

Glide 是如何避免重复下载同一张图片的？（比如两个 ImageView 显示相同 URL）

Glide 如何处理大图加载、采样缩放（Downsampling）？

Glide 中的 RequestManager 和 RequestBuilder 是什么关系？

Glide 如何感知 Activity/Fragment 生命周期？怎么做到自动取消请求的？

Glide 中的 BitmapPool 有什么用？和 LruCache 有什么区别？

🔬 三、源码与优化（适合高级面试者）
你了解 Glide 的模块化设计吗？比如 AppGlideModule 有什么作用？

如果要给 Glide 增加一个自定义的解码器（例如 WebP 动图），如何实现？

Glide 是如何调度线程和任务的？用了哪些线程池？

Glide 的内存缓存使用了哪种数据结构？如何计算 Bitmap 的大小？

Glide 是如何避免 OOM 的？你在实际项目中遇到过 OOM 吗，如何处理的？

如何调试或监控 Glide 的性能？比如加载速度、缓存命中率？

你有没有手动清理 Glide 的缓存？什么时候需要这么做？
