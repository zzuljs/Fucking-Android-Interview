
# 1 什么是ContentProvider，及其使用场景
ContentProvider 的作用是跨应用数据分享提供统一接口，我们知道安卓系统是对外隔离的，要想让它应用能使用自己的数据，需要用ContentProvider  

使用场景：跨应用分享数据，从系统App对外接口读取数据，如通讯录读取联系人 


# 2 ContentProvider、ContentResolver、ContentObserver之间的关系
`ContentProvider`:管理数据、提供数据增删查改操作，数据源可以是数据库、文件、XML、网络等  
`ContentResolver`:外部进程可以通过ContentResolver与ContentProvider交互  
`ContentObserver`:监听ContentProvider中数据变化

# 3 ContentProvider的实现原理

假设应用A提供共享数据，此时未启动，应用B查询A数据，整体过程如下：  
1. query过程  
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
ContentResolver将参数封装为QueryArgs，通过acquireProvider方法获取IContentProvider的Binder代理对象，这是个本地缓存逻辑，如果缓存不存在，转向AMS发起跨进程请求  

2. AMS处理Provider获取请求  
```java
// ActivityManagerService.java
public final ContentProviderHolder getContentProvider(
    IApplicationThread caller, String name, int userId, boolean stable) {
    ...
    // 根据URI的authority查找目标Provider信息
    ProviderInfo providerInfo = resolveContentProvider(name, ...);
    // 检查权限（readPermission）
    checkHoldingPermissions(...);
    // 若目标Provider所在进程未启动，触发启动
    if (providerProc == null || !providerProc.isAlive()) {
        ProcessRecord proc = startProcessLocked(...);
    }
    ...
}
```

通过PMS查找URI对应的ProviderInfo（根据authority匹配）  
检查应用B是否有足够权限  
如果应用A没有在运行，调用startProcessLocked方法启动进程  

3. 启动应用A进程  
AMS通过Zygote进程fork新进程，执行ActivityThread.main方法，进入主线程消息循环  

4. 应用A初始化ContentProvider  
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

ActivityThread创建Application对象，并遍历AndroidManifest中所有的<provider>标签，调用installContentProvider方法初始化ContentProvider子类, 并同步AMS Provider已经就绪

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
4. AMS返回Provider引用到应用B  
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
AMS将ContentProviderHolder（包含IContentProvider Binder接口）存储映射表，同时通过Binder传给B的ContentResolver，B将缓存IContentProvider代理，避免重复访问AMS  

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
应用A的Binder线程池接收请求，调用query方法，并执行query数据交互逻辑  
查询结果Cursor通过CursorWindow利用匿名共享内存（Ashmem，这是Android独有的机制）跨进程传输，Cursor将数据缓冲区映射到共享内存，应用B可直接读取，无需拷贝  

6. 应用B接收并处理数据  
```java
// CursorWrapperInner.java
CursorWrapperInner cursor = new CursorWrapperInner(cursor, provider);
```
框架将Cursor封装为CursorWrapperInner，确保跨进程访问的安全性  
query在Binder线程池执行，会阻塞直到数据返回为止，注意query最终结果会返回到调用所在线程，而不是自动切回主线程  

# 4 ContentProvider的优点
1. 封装  
提供对外访问数据接口，解耦了底层数据存储方式和细节，使访问更加简单、高效  
外界无需关心数据本身是如何实现、如何变更，只需要按照要求访问数据即可  

2. 提供一种跨进程数据共享方式  
相比于AIDL，更加简单，能够做到跨进程数据增删查改，以及权限控制

# 5 URI是什么
URI即 Uniform Resource Identifier，即统一资源标识符  
作用是ContentProvider唯一标识，URI分为系统预置&自定义，分别对应系统内置数据（如通讯录、日程表）和自定义数据库  

URI = content://com.xxx.xxx/user/id

- content:// 前缀是固定的，表明这是一个ContentProvider的URI  
- com.xxx.xxx 是授权名，必须与AndroidManifest中保持一致
- user和id分别是表名和要查询的id，按需自定义
