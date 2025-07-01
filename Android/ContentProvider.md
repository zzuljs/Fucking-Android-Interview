# 1 什么是 `ContentProvider`，及其使用场景  

`ContentProvider` 的作用是跨应用数据分享提供统一接口，我们知道安卓系统是对外隔离的，要想让它应用能使用自己的数据，需要用 `ContentProvider`  

使用场景：跨应用分享数据，从系统 `App` 对外接口读取数据，如通讯录读取联系人  

# 2 `ContentProvider`、`ContentResolver`、`ContentObserver` 之间的关系

`ContentProvider`：管理数据、提供数据增删查改操作，数据源可以是数据库、文件、`XML`、网络等  
`ContentResolver`：外部进程可以通过 `ContentResolver` 与 `ContentProvider` 交互  
`ContentObserver`：监听 `ContentProvider` 中数据变化

# 3 `ContentProvider` 的实现原理

假设应用 `A` 提供共享数据，此时未启动，应用 `B` 查询 `A` 数据，整体过程如下：  

1. `query` 过程  

```java
// 应用B中调用：
Cursor cursor = getContentResolver().query(
    Uri.parse("content://com.appA.provider/table1"), 
    projection, 
    selection, 
    selectionArgs, 
    sortOrder
);
```
`ContentResolver` 将参数封装为 `QueryArgs`，通过 `acquireProvider` 方法获取 `IContentProvider` 的 `Binder` 代理对象，这是个本地缓存逻辑，如果缓存不存在，转向 `AMS` 发起跨进程请求  

2. `AMS` 处理 `Provider` 获取请求  

```java
// ActivityManagerService.java
public final ContentProviderHolder getContentProvider(
    IApplicationThread caller, String name, int userId, boolean stable) {
    ...
    // 根据 URI 的 authority 查找目标 Provider 信息
    ProviderInfo providerInfo = resolveContentProvider(name, ...);
    // 检查权限（readPermission）
    checkHoldingPermissions(...);
    // 若目标 Provider 所在进程未启动，触发启动
    if (providerProc == null || !providerProc.isAlive()) {
        ProcessRecord proc = startProcessLocked(...);
    }
    ...
}
```

通过 `PMS` 查找 `URI` 对应的 `ProviderInfo`（根据 `authority` 匹配）  
检查应用 `B` 是否有足够权限  
如果应用 `A` 没有在运行，调用 `startProcessLocked` 方法启动进程  

3. 启动应用 `A` 进程  

`AMS` 通过 `Zygote` 进程 `fork` 新进程，执行 `ActivityThread.main` 方法，进入主线程消息循环  

4. 应用 `A` 初始化 `ContentProvider`  

```java
// ActivityThread.java
public void handleBindApplication(AppBindData data) {
    ...
    // 创建Application实例
    app = data.info.makeApplication(...);
    // 初始化所有ContentProvider
    installContentProviders(app, data.providers);
    ...
}
```

`ActivityThread` 创建 `Application` 对象，并遍历 `AndroidManifest` 中所有的 `<provider>` 标签，调用 `installContentProvider` 方法初始化 `ContentProvider` 子类, 并同步 `AMS` Provider 已经就绪

```java
// ActivityThread.java
private void installContentProviders(Context context, List<ProviderInfo> providers) {
    for (ProviderInfo info : providers) {
        // 反射创建ContentProvider实例
        ContentProvider provider = (ContentProvider)Class.forName(info.name).newInstance();
        // 调用provider.attachInfo()
        provider.attachInfo(context, info);
        // 调用provider.onCreate()
        provider.onCreate();
        // 注册到ActivityThread的mProviders映射表
        mProviders.put(info.authority, provider);
    }
    // 通知AMS Provider已发布，ActivityManager持有AMS代理对象
    ActivityManager.getService().publishContentProviders(...);
}
```

4. `AMS` 返回 Provider 引用到应用 `B`  

```java
// ActivityManagerService.java
public void publishContentProviders(IApplicationThread caller, List<ContentProviderHolder> providers) {
    ...
    // 将ContentProviderHolder存入AMS的mProviderMap
    for (ContentProviderHolder holder : providers) {
        mProviderMap.put(holder.info.authority, holder);
    }
    ...
}
```
`AMS` 将 `ContentProviderHolder`（包含 `IContentProvider` Binder 接口）存储映射表，同时通过 Binder 传给 `B` 的 `ContentResolver`，`B` 将缓存 `IContentProvider` 代理，避免重复访问 `AMS`  

5. 执行跨进程查询操作  

```java
// ContentProviderNative.java (Binder接口)
public Cursor query(String callingPkg, Uri uri, String[] projection, 
    String selection, String[] selectionArgs, String sortOrder, 
    ICancellationSignal cancellationSignal) {
    ...
    // 实际调用到应用A的ContentProvider.query()
    return provider.query(uri, projection, selection, selectionArgs, sortOrder);
}
```
应用 `A` 的 Binder 线程池接收请求，调用 `query` 方法，并执行 `query` 数据交互逻辑  
查询结果 `Cursor`通过 `CursorWindow` 利用匿名共享内存（Ashmem，这是Android独有的机制）跨进程传输，`Cursor` 将数据缓冲区映射到共享内存，应用 `B` 可直接读取，无需拷贝  

6. 应用 `B` 接收并处理数据  

```java
// CursorWrapperInner.java
CursorWrapperInner cursor = new CursorWrapperInner(cursor, provider);
```

框架将 `Cursor` 封装为 `CursorWrapperInner`，确保跨进程访问的安全性  
`query` 在 Binder 线程池执行，会阻塞直到数据返回为止，注意 `query` 最终结果会返回到调用所在线程，而不是自动切回主线程  

# 4 `ContentProvider` 的优点

1. 封装  

提供对外访问数据接口，解耦了底层数据存储方式和细节，使访问更加简单、高效  
外界无需关心数据本身是如何实现、如何变更，只需要按照要求访问数据即可  

2. 提供一种跨进程数据共享方式  

相比于 `AIDL`，更加简单，能够做到跨进程数据增删查改，以及权限控制

# 5 URI 是什么

URI 即 Uniform Resource Identifier，即统一资源标识符  
作用是 `ContentProvider` 唯一标识，URI 分为系统预置 & 自定义，分别对应系统内置数据（如通讯录、日程表）和自定义数据库  

URI = `content://com.xxx.xxx/user/id`

- `content://` 前缀是固定的，表明这是一个 ContentProvider 的 URI  
- `com.xxx.xxx` 是授权名，必须与 `AndroidManifest` 中保持一致
- `user` 和 `id` 分别是表名和要查询的 id，按需自定义
