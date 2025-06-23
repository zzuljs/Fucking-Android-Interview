# 0 `Service`概述

`Service`是用于执行后台任务或者跨进程通信的核心组件，设计目标是用于处理不需要用户界面但必须长期运行的场景，如  
- 后台音乐播放
- `GPS`导航
- 跨进程处理结果（支付宝`SDK`用`Service`处理支付结果）
- 文件下载等后台耗时任务处理  

如果是短时间任务，如网络请求，可以考虑用线程、协程代替  
如果是延迟、定时任务，可以考虑用`WorkManager`代替

# 1 `Service`的生命周期、两种启动方式的区别

`startService`和`bindService`最大的区别就是前者独立运行，后者绑定组件（一般是`Activity`，但也可以是`Fragment`、`Application`）

- **`startService`**  
`onCreate`->`onStartCommand`->`onDestroy`  

- **`bindService`**  
`onCreate`->`onBind`->`onUnbind`->`onDestroy`  

## 1.1 两种方式区别

`startService`

1. 启动  

如果服务已经开启，多次执行`startService`不会重复执行`onCreate`，而是会调用`onStart`和`onStartCommand`  

如果服务已经开启，多次执行`bindService`，`onCreate`、`onBind`方法不会重复调用  

2. 销毁  

当执行`stopService`时，直接调用`onDestroy`  

调用者主动调用`unbindService`，后者调用者的`Content`不存在（如`Activity`被`finish`），`Service`就会调用`onUnbind`->`onDestroy`  
使用`startService`方法启用服务，调用者与服务之间没有关联，即使调用者被回收，服务仍然运行  

使用`bindService`方法启用服务，调用者和服务之间是绑定的，调用者退出，服务也会终止   

# 2 `Service`启动流程  

# 2.1 `startService`流程  

![Service启动流程图](./../images/Service启动流程图.png)

1. `Process` `A`进程采用`Binder IPC`向`system_server`进程（`AMS`）发起`startService`请求  
2. `system_server`进程接收到请求后，向`Zygote`进程发送创建进程请求  
3. `Zygote`进程`fork`子进程  
4. `App`进程，通过`Binder IPC`向`system_server`进程发起`attachApplication`请求  
5. `system_server`进程在收到请求后，进行一系列准备工作后，再通过`Binder IPC`向`App`进程发起`scheduleCreateService`请求  
6. `App`进程的`Binder`线程接收到请求后，通过`Handler`向主线程发送`CREATE_SERVICE`消息  
7. 主线程在收到`Message`后，通过发射机制创建目标`Service`，并回调`Service.onCreate`方法  

如果`App`进程已经存在，那么跳过2、3步骤

# 2.2 `bindService`流程

1. `Process` `A`进程发起`bindService`请求，`ContextImpl.bindService`接口被调用，通过`Binder IPC`向`system_server`进程发起请求  
2. `system_server`进程中的`AMS`收到请求，解析`Intent`数据，先判断是否有权限（没有权限直接抛出异常），然后通过`Process.start`向`Zygote`进程发起创建进程请求  
3. `Zygote`接收到请求后，`fork`子进程，执行`ActivityThread.main`作为`App`进程入口，以下是`App`进程初始化流程：  
- 创建`ActivityThread`:`App`主线程初始化，创建`ApplicationThread`，并注册到`AMS`  
- 绑定到`AMS`：通过`IActivityManager.attachApplication`通知`AMS`进程已启动  
- `AMS`回调：`AMS`收到`attachApplication`后，完成进程`App`进程初始化
4. `AMS`通过`IApplicationThread.scheduleBindService`，通知`Process` `A`绑定`Service`，`A`的处理逻辑：  
- 创建`Service`实例，通过反射调用目标`Service`构造函数，执行`Service.onCreate`  
- 调用`onBind`，并返回`Service.onBind`，返回`IBinder`对象，用于`IPC`  
- 通过`IActivityManager.publishService`发送回`AMS`  
5. `AMS`收到`App`的`IBinder`对象后，传递给调用方`Process` `A`，`A`回调`onServiceConnected`接口  
6. `Process` `A`获得`IBinder`对象后，与`App`进程的跨进程通信就建立了，可以将其转换成`AIDL`进行`IPC`  

# 3 `Service`与`Activity`如何通信  

## 3.1 通过`Binder`对象  

1. `Service`中添加一个继承`Binder`的内部类，并添加相应的逻辑方法；  
2. `Service`中重写`Service`的`onBind`方法，返回我们刚刚定义的内部类实例；  

```java
// MyService.java
public class MyService extends Service {
    private final IBinder binder = new MyBinder();

    // 自定义 Binder 类，用于暴露 Service 的方法
    public class MyBinder extends Binder {
        MyService getService() {
            return MyService.this; // 返回 Service 实例
        }
    }

    @Override
    public IBinder onBind(Intent intent) {
        return binder; // 返回 Binder 对象
    }
}
```

3. `Activity`中`bindService`，重写`ServiceConnection`，`onServiceConnection`时返回`IBinder`调动逻辑方法； 

```java
// MainActivity.java
public class MainActivity extends AppCompatActivity {
    private MyService myService;
    private boolean isBound = false;

    // 定义 ServiceConnection
    private ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            MyService.MyBinder binder = (MyService.MyBinder) service;
            myService = binder.getService();
            isBound = true;
            
            // 设置回调（接收 Service 发来的数据）
            myService.setCallback(new MyService.ServiceCallback() {
                @Override
                public void onTaskCompleted(String result) {
                    runOnUiThread(() -> {
                        // 处理业务逻辑
                    });
                }
            });
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            isBound = false;
        }
    };

    @Override
    protected void onStart() {
        super.onStart();
        // 绑定 Service
        Intent intent = new Intent(this, MyService.class);
        bindService(intent, connection, Context.BIND_AUTO_CREATE);
    }

    @Override
    protected void onStop() {
        super.onStop();
        // 解绑 Service
        if (isBound) {
            unbindService(connection);
            isBound = false;
        }
    }

    // 调用 Service 方法（例如按钮点击事件）
    public void onButtonClick(View view) {
        if (isBound) {
            myService.performTask("Hello Service!");
        }
    }
}
```

## 3.2 通过`BroadCast`广播与`Activity`通信  

(不推荐) `Activity`注册动态广播`BroadcastReceiver`，`Service`直接调用`sendBroadcast`即可  

# 4 `IntentService`是什么，`IntentService`原理，应用场景及其与`Service`的区别  

`IntentService`是`Service`的一个子类，用于处理异步后台任务，通过工作线程（`HandlerThread`）逐个处理`Intent`请求，在任务完成后自动停止，开发者仅需要实现`onHandleIntent`方法即可  

```java
public class MyIntentService extends IntentService {

    public MyIntentService() {
        super("MyIntentService"); // 指定工作线程名称
    }

    @Override
    protected void onHandleIntent(@Nullable Intent intent) {
        // 在工作线程中处理任务
    }
}

// 启动服务（在 Activity 中）
Intent serviceIntent = new Intent(context, MyIntentService.class);
serviceIntent.setAction("ACTION_DOWNLOAD");
serviceIntent.putExtra("url", "https://example.com/file.zip");
context.startService(serviceIntent);
```
**适用场景** ：`IntentService`适用于轻量级、无需即时响应的后台任务，如异步下载文件、将用户行为定时记录到数据库等  

`IntentService`与`Service`的对比:  

| 特性           | `IntentService`                              | 普通 `Service`                              |
| -------------- | ------------------------------------------ | ---------------------------------------- |
| ​**线程模型**   | 自动创建工作线程，任务异步执行             | 默认在主线程运行，需手动创建子线程       |
| ​**任务管理**   | 串行处理任务队列                           | 需自行实现多线程和任务队列逻辑           |
| ​**生命周期**   | 任务完成后自动销毁                         | 需手动调用 `stopSelf()` 或 `stopService()` |
| ​**适用场景**   | 轻量级、短时间任务                         | 需长期运行或复杂生命周期管理的任务（如音乐播放） |

# 5 `Service`的`onStartCommand`方法有几种返回值？各代表什么意思？

1. `START_NOT_STICKY`  

在执行完`onStartCommand`后，服务被异常`kill`，系统不会重启该服务  

2. `START_STICKY`  

重传`Intent`，使用这个返回值时，如果在执行完`onStartCommand`后，服务被异常`kill`，系统会自动重启该服务，并且`onStartCommand`方法会执行，`onStartCommand`方法中的`intent`为`null`

3. `START_REDELIVER_INTEN`  

使用这个返回值时，服务被异常`kill`时，系统会自动开启服务，并将`Intent`的值传入  
适用于主动执行应该恢复的作业（如下载文件）

# 6 `bindService`与`startService`混合使用的生命周期以及如何关闭？

如果只是想启动一个后台服务长期进行某项服务，那么使用`startService`就可以了  

如果还要与`Activity`通信，有两种方案：1.使用`Broadcast`；2.使用`bindService`  

如果先`startService`，再`bindService`，那么生命周期顺序是：`onCreate`->`onStartCommand`->`onBind`  

如果先`bindService`，再`startService`，那么生命周期顺序是：`onCreate`->`onBind`->`onStartCommand`  

在 `startService`+`bindService`执行之后，  

如果仅调用`stopService`，`Service`不会立刻执行`onDestroy`，在`Activity`退出之后，再执行`onDestroy`  

如果仅调用`unbindService`，只有`onUnbind`方法回调，不会执行`onDestroy`，即便是`Activity`退出，也不会`onDestroy`  

如果要完全退出`Service`，那么应该先调用`unbindService`，再执行`stopService`，此时`Service`的生命周期：`onUnbind`->`onDestroy`
