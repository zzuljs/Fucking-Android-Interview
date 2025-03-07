# 1 Activity启动流程
分为两种情况:1.从桌面点击进入；2.App进程内进入
## 1.1 从桌面点击进入

桌面Launcher进程捕获点击事件，解析目标App的Intent（包含MAIN Action和LAUNCHER Category），通过Binder IPC向system_server进程中的ActivityManagerServices发起startActivity请求，
system_server的AMS收到请求后，先检查App进程是否存在，如果不存在，那么通过Socket向Zygote进程发起创建App进程的请求，
Zygote进程fork出新的进程，即App进程，
App进程在被创建后，随即通过Binder IPC向system_server进程的AMS发出attachApplication请求，该请求将ApplicationThread绑定到AMS，
AMS收到请求之后，创建Application对象，调用attachBaseContext，初始化ContentProvider以及执行Application.onCreate，
AMS在初始化完成之后，通过Binder IPC向App进程发送scheduleLaunchActivity请求，
App进程的Binder线程（ApplicationThread）收到之后，通过Handler向App主线程发送LAUNCH_ACTIVITY消息，
主线程接受到消息后，通过发射机制创建目标Activity，并回调Activity.onCreate、onStart、onResume等生命周期方法，
WindowManager添加窗口，SurfaceFlinger完成UI渲染

![Activity启动流程](./../images/Activity启动流程图.png)

## 1.2 从App内进入
App进程->(AMP，ActivityManagerProxy)startActivity(Hook插入点1)->system_server进程AMS->解析启动参数，scheduleLaunchActivity->App进程，ApplicationThread->ActivityThread(Hook插入点2)->Activity生命周期


## 1.3 补充内容
1. 什么是Launcher进程？
即桌面进程，用于桌面显示、近期任务切换等，由SystemServer启动（即system_server进程）

2. 什么是system_server进程？
system_server进程是Android核心系统进程之一，用于管理几乎所有关键系统服务，如AMS、WMS、PMS，
system_server由java类SystemServer.java运行，由Zygote进程fork出来

3. 什么是Zygote进程？
Zygote进程是Android最重要的核心进程之一，用于孵化所有App进程和system_server进程，
主要作用是创建Android运行时环境，预加载Android SDK核心类，fork其他进程，
由init进程fork

4. 什么是ActivityManagerServices
ActivityManagerService（AMS） 是 Android 系统中 负责管理应用进程和 Activity 生命周期的核心服务，它属于 Framework 层，运行在 system_server 进程 中。

它的主要职责包括：
✅ 进程管理：启动、停止、回收应用进程（与 Zygote 交互）
✅ Activity 管理：启动、暂停、销毁 Activity（维护 Activity 栈）
✅ 任务栈管理：控制 Task（任务栈）和 Back Stack（返回栈）
✅ 广播（Broadcast）管理：分发 BroadcastReceiver
✅ Service 管理：启动、绑定、停止 Service
✅ ANR 监控：检测 App Not Responding（ANR）


# 2 onSaveInstanceState(), onRestoreInstanceState调用时机
onSaveInstanceState和onRestoreInstanceState主要用于Activity和Fragment遭遇意外销毁（屏幕旋转、系统回收资源），保存和恢复临时UI状态
1. onSaveInstanceState
- 从最近应用切换其他程序
- 按下HOME键
- 屏幕方向切换
- 按电源键
- 从当前Activity启动一个新的Activity
生命周期：onPause->onSaveInstanceState->onStop
onSaveInstanceState常用于保存UI临时状态，比如输入框内容、滚动条位置

2. onRestoreInstanceState
只有在Activity确实被系统回收、重建Activity的情况下才会用，如
- 屏幕旋转, 屏幕旋转时，Activity生命周期如下：onPause->onSaveInstanceState->onStop->onDestroy->onCreate->onStart->onRestoreInstanceState->onResume
- 在后台被回收
- 按HOME键返回桌面，然后点击应用图标回到原来的页面，Activity生命周期：onStart->onRestoreInstanceState->onResume
注意：仅当onSaveInstanceState非空才调用onRestoreInstanceState，因此，onRestoreInstanceState被调用，onSaveInstanceState一定会被调用
3. 源码
系统会调用ActivityThread的preformStopActivity方法中调用onSaveInstanceState，将状态保存在mActivities中，mActivities维护了Activity信息表，当Activity重启时，先从mActivities查询到对应的ActivityClientRecord，
如果有信息，则调用Activity的onRestoreInstanceState方法，
在ActivityThread的performLaunchActivity方法中，会判断ActivityClientRecord对象state是否为空
不为空，通过Activity的onSaveInstanceState获取其UI状态信息，通过这些信息传递给onCreate方法

4. onCreate与onRestoreInstanceState两者都可以用于恢复状态：
- onCreate(Bundle savedInstanceState) 在 Activity 创建 时调用，可用于恢复状态。
- onRestoreInstanceState(Bundle savedInstanceState) 仅在 onSaveInstanceState() 发生后 才调用。

# 3 Activity的启动模式（Launch Mode）和使用场景
## 3.1 什么是任务栈
我们每次打开一个新的Activity，或者退出当前的Activity，都会在一个称为任务栈的结构中增减Activity，一个任务栈包含一组Activity集合，
Android通过ActivityRecord、TaskRecord、ActivityStack、ActivityStackSupervisor、ProcessRecord有序的管理每个Activity

## 3.2 Standard
默认模式，每次启动Activity都会创建一个新的Activity实例  
适用场景：可以重复打开，无需复用的页面，如详情页
## 3.3 SingleTop
栈顶复用模式，若目标Activity存在栈顶，直接复用，调用该Activity.onNewIntent方法，如果目标Activity不存在栈顶，重建实例  
适用场景：通知栏跳转详情页、支付结果页，防止栈顶重复创建
## 3.4 SingleTask
栈内单例，若目标Activity存在，则清除上方页面，并复用，如果不存在，那么新建Activity  
适用场景：主界面
## 3.5 SingleInstance
全局单例，单开一个Task，仅允许存在目标Activity  
适用场景：系统级单个页面，如来电页面

## 3.6 启动模式与Intent Flags关系
开发者可以通过 Intent.addFlags() ​动态覆盖 Manifest 中静态定义的启动模式：  
|常用Flag|说明|
|---|-------|
|FLAG_ACTIVITY_NEW_TASK	|在新任务栈中启动 Activity（需与 singleTask 或 taskAffinity 配合）|
|FLAG_ACTIVITY_CLEAR_TOP|若目标 Activity 已存在，清除其上的所有 Activity。|
|FLAG_ACTIVITY_SINGLE_TOP|效果等同 singleTop 模式|

# 4 Activity A跳转Activity B，再按返回键，生命周期执行顺序
A跳转B：A.onPause->B.onCreate->B.onStart->B.onResume->A.onStop  
B在按下back键：B.onPause->A.onRestart->A.onStart->A.onResume->B.onStop->B.onDestroy  

如果B的launchMode是singleInstance、singleTask且B有可复用实例，生命周期回调如下：  
A.onPause->B.onNewIntent->B.onRestart->B.onStart->B.onResume->A.onStop->(A被移除的话)A.onDestory  

当B的launchMode是singleTop且已经在栈顶，此时:  
B.onPause->B.onNewIntent->B.onResume  

# 5 横竖屏切换、HOME键、返回键、锁屏与解锁，跳转透明Activity，启动一个Theme为Dialog的Activity，弹出Dialog的Activity生命周期
1. 横竖屏切换
```xml
<activity
    android:name=".MainActivity"
    android:configChanges="orientation|screenSize|screenLayout|keyboardHidden" />
```
仅当设置上述属性，横竖屏切换才不会触发Activity重建，仅触发onConfigurationChanged方法回调  

启动：onCreate->onStart->onResume  
切换横竖屏：onPause->onSaveInstanceState->onStop->onDestory->onCreate->onStart->onSaveInstanceState->onResume  
2. HOME键  
onPause->onStop->onRestart->onStart->onResume  
3. BACK键
onPause->onStop->onDestory
4. 锁屏  
锁屏只会调用onPause，不会调用onStop，开屏后onResume  
5. 弹出Dialog  
Dialog是通过WindowManager.addView显示的，没有经过AMS，所以不会对生命周期有任何影响  
6. A跳转Theme为Dialog的Activity、透明Activity B  
A.onPause->B.onCreate->B.onStart->B.onResume  
注意：如果Activity A跳转了一个theme为Dialog的Activity B，但是B设置了全屏，那么此时A的onStop会被回调：  
A.onPause->B.onCreate->B.onStart->B.onResume->A.onStop  
A.onStop是否回调判断依据是A是否完全不可见
# 6 onStart和onResume, onPause和onStop的区别  
onStart表示页面已经可见  
onResume表示页面可交互，能够获取焦点  
onStop表示页面不可见  
onPause表示页面失去焦点，不可交互  
# 7 Activity之间传递数据的方式Intent是否有大小限制，如果传递数量偏大，有哪些方案
Intent携带数据要从App进程IPC到system_server进程的AMS处理，使用的是Binder通信，Binder通信有大小限制，通常是1MB，Binder本身不是为了拷贝大量数据，而是为了进程间频繁、灵活通信设计，是一种轻量级、低开销的IPC机制  
如果是非IPC，那么使用单例、SQlite、Sharedpreference、file都可以  
如果是IPC，考虑使用共享内存和Socket

Binder 通信过程：  
1. Binder驱动在内核空间创建一个数据接收缓存区；  
2. 在内核空间开辟一块内核缓存区，建立内核缓存区和内核空间数据接收区之间的映射关系，以及内核中数据接收缓存区和接收进程用户空间地址的映射关系；  
3. 发送方进程通过系统调用copyfromuser将数据copy到内核空间的内核缓存区，由于内核缓存区和接收进程的用户空间存在内存映射，因此也就相当于把数据发送到了接收进程的用户空间，这样便完成了一次进程间通信，相比Socket，少了一次内存拷贝.

# 8 Activity的onNewIntent方法什么时候执行

# 9 显式启动和隐式启动

# 10 scheme使用场景、协议格式、如何使用

# 11 ANR的四种场景

# 12 onCreate和onRestoreInstance恢复数据的区别

# 13 Activity间传递数据的方式
1. 通过Intent传递（putExtra内部是一个Bundle）  
2. 通过全局变量传递
3. SharedPreference
4. 数据库
5. 文件

# 14 跨App启动Activity方式

# 15 Activity的任务栈是什么

# 16 有哪些Activity常用的标记位Flags

# 17 Activity的数据是怎么保存的，进程被kill后，保存的数据怎么恢复的


