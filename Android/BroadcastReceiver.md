# 1 广播的分类和使用场景

Android 广播范围两个角色：广播发送者、广播接受者  

广播接收器的注册分为两种：静态注册、动态注册  

## 1.1 静态广播

静态广播接收者：通过`AndroidManifest.xml`的标签来申明的`BroadcastReceiver`  

1. 创建`Receiver`类  

```java
public class MyReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        // 在这里处理接收到的广播
        String action = intent.getAction();
        if (action != null) {
            if (action.equals("com.example.MY_CUSTOM_ACTION")) {
                // 处理自定义广播
            }
        }
    }
}

```
2. `xml`声明`<receiver>`标签

```xml
<receiver
    android:name=".MyReceiver"
    android:exported="true">
    <intent-filter>
        <action android:name="com.example.MY_CUSTOM_ACTION" />
    </intent-filter>
</receiver>
```

`name` 对应广播接收器的类  
`exported` 能否接收外部广播，`true`表示允许接收其他应用的广播，`false`表示只接收本应用广播    

**注意：**Android 8（API 26）以后，静态广播仅用于接收少数系统广播  (如开机启动、网络变化)

## 1.2 动态广播

通过`AMS.registerReceiver`方式注册的`BroadReceiver`, 动态注册更为灵活，可在不需要是通过`unregisterReceiver`取消注册  

与静态广播不同的是，动态广播一定与`Activity`或`Service`绑定，组件销毁时需要手动解绑，不然会内存泄漏  

1. 创建`Receiver`类（与静态广播相同） 

```java
public class MyReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        // 处理接收到的广播
        String action = intent.getAction();
        if (action != null) {
            if (action.equals("com.example.MY_CUSTOM_ACTION")) {
                // 处理业务逻辑
            }
        }
    }
}
```

2. 动态注册广播接收器

```java
public class MainActivity extends AppCompatActivity {
    private MyReceiver receiver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {

        // 创建广播接收器实例
        receiver = new MyReceiver();

        // 创建 IntentFilter 并指定要接收的广播类型
        IntentFilter filter = new IntentFilter("com.example.MY_CUSTOM_ACTION");

        // 注册广播接收器
        registerReceiver(receiver, filter);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        // 在 Activity 销毁时注销广播接收器
        unregisterReceiver(receiver);
    }
}
```

3. 发送广播  

```java
// 所有注册了 com.example.MY_CUSTOM_ACTION Receiver都能接收
Intent intent = new Intent("com.example.MY_CUSTOM_ACTION");
// 发送全局广播
sendBroadcast(intent);

```

4. 广播权限  

广播的发送方和接收方都可以指定广播权限  

发送广播指定权限：限制仅声明了正确权限的应用才可以接收  

接收广播指定权限：限制仅声明了正确权限的应用才可以接收  

```xml
<permission
    android:name="com.example.MY_CUSTOM_PERMISSION"
    android:protectionLevel="signature" />
```

​**android:protectionLevel**：权限的保护级别，常见的有：  
- `normal`：低风险权限，安装时自动授予。  
- `dangerous`：高风险权限，需要用户明确授权。  
- `signature`：只有使用相同签名的应用才能获得该权限。  
- `signatureOrSystem`：只有系统应用或使用相同签名的应用才能获得该权限。  

发送指定权限

```java
Intent intent = new Intent("com.example.MY_CUSTOM_ACTION");
sendBroadcast(intent, "com.example.MY_CUSTOM_PERMISSION");
```

接收指定权限  

```java
// 动态注册
IntentFilter filter = new IntentFilter("com.example.MY_CUSTOM_ACTION");
registerReceiver(receiver, filter, "com.example.MY_CUSTOM_PERMISSION", null);
```

# 2 广播的两种注册方式的区别

1. 静态注册  

常驻系统、不受组件生命周期影响，即便应用退出，广播还是可以接收，耗电、占内存  

2. 动态注册  

非常驻，跟随组件的生命变化，组件结束，广播结束，在组件结束前，需要先移除广播，否则容易造成内存泄漏  

# 3 广播发送和接收的原理  

## 3.1 静态注册  

静态注册是通过`AndroidManifest`文件中声明了`BroadcastReceiver`的自定义类和对应的`IntentFilter`，来告诉`PackageManagerService（PMS）`这个`App`注册了静态广播  

当AMS接收到广播后，会查找所有动态注册和静态注册的广播接收器，静态注册的广播接收器是通过PMS发现的，PMS找到对应的App  

如果 `App` 进程已经创建，那么直接调用 `App` 的 `ApplicationThread.scheduleReceiver`  

如果 `App` 进程尚未创建，先启动 `App` 进程，`App` 进程启动后回调 `AMS.attachApplication`，该方法继续派发刚才的广播，`App` 这边收到调用后，会先通过 `Handler` 转到主线程，然后根据 `AMS` 传过来的参数实例化 `App` 对应的 `Receiver`，然后回调 `onReceive` 方法  

## 3.2 动态注册  

1. 创建对象 `LoadedApk.ReceiverDispatcher.InnerReceiver` 实例，该对象继承于 `IIntentReceiver.Stub`（`InnerReceiver` 实际是一个 `Binder` 本地对象）  

2. 将 `IIntentReceiver` 对象和注册广播时所传递的 `IntentFilter` 对象发送给 `AMS`，`AMS` 记录 `IIntentReceiver`、`IntentFilter` 和注册的进程 `ProcessRecord`，并建立它们的对应关系  

3. 当广播发出时，`AMS` 根据广播 `intent` 所携带的 `IntentFilter` 找到对应的 `IIntentReceiver` 和 `ProcessRecord`，然后回调 `App` 的 `ApplicationThread` 对象的 `scheduleRegisteredReceiver`，将 `IIntentReceiver` 和广播 `intent` 一起传给 `App`，`App` 直接调用 `IIntentReceiver` 的 `performReceiver`  

4. 广播通过 `Binder` 线程回调到接收进程，接收进程通过 `ActivityThread` 里的 `Handler` 切换回主线程，然后回调 `BroadcastReceiver` 的 `onReceive` 方法  

# 4 本地广播和全局广播的区别

`BroadcastReceiver` 是针对应用间、应用与系统、应用内部进行通信的一种方式    

`LocalBroadcastReceiver` 仅在自己的应用内发送接收广播，也就是只有自己的应用能收到，数据更加安全，广播在这个程序里，更加高效  

`BroadcastReceiver` 基于 `Binder` 进行 `IPC` 通信  

`LocalBroadcastReceiver` 基于 `Handler` 通信机制  

`LocalBroadcastReceiver` 不能够静态注册、只能采用动态注册的方式

