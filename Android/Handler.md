# 0 Handler的基本使用

1. 主线程更新UI  
```java
// 创建 Handler，处理消息
Handler handler = new Handler(Looper.getMainLooper()) {
    @Override
    public void handleMessage(Message msg) {
        // 处理消息（在主线程执行）
        if (msg.what == UPDATE_UI) {
            textView.setText("Message received");
        }
    }
};

// 子线程发送消息
Thread {
    val msg = handler.obtainMessage(1, "Hello from Kotlin!")
    handler.sendMessage(msg)
}.start()
```
这种用法常用于定制化处理Message，需要重写handleMessage方法  

2. post+Runnable
```java
// 主线程 Handler
Handler handler = new Handler(Looper.getMainLooper());

// 在子线程中 post Runnable
new Thread(() -> {
    handler.post(() -> {
        textView.setText("Updated by Runnable in Java");
    });
}).start();
```
常用于线程切换

3. 

# 1 Handler的实现原理  
Handler:负责消息的发送和处理  
Message:消息对象的封装，类似于链表节点  
MessageQueue:消息队列，用于存放消息对象的数据结构  
Looper:消息队列的处理者（用于轮询消息队列的消息对象）  

Handler发送消息时调用MessageQueue.enqueueMessage插入一条消息到MessageQueue,Looper不断轮询调用MessageQueue.next方法，如果消息队列存在消息就调用Handler.dispatchMessage处理消息

# 2 子线程中能不能直接new一个Handler，为什么主线程可以  

不能，Handler的构造方法会通过Looper.myLooper获取looper对象，如果为空，抛出异常，主线程则因为入口处ActivityThread.main方法中通过Looper.prepareMainLooper获取这个对象，并通过Looper.loop开启循环，子线程中如果想要构造Handler对象，需要手动调用Looper.prepare获取Looper对象，然后手动调用Looper.loop开启循环，如下：


```java
new Thread(() -> {
    Looper.prepare(); // 初始化 Looper
    Handler subHandler = new Handler(); // 正确创建 Handler
    Looper.loop(); // 启动消息循环
}).start();

// 或者用更高级的方式 HandlerThread+Handler
HandlerThread handlerThread = new HandlerThread("SubThread");
handlerThread.start();
Handler subHandler = new Handler(handlerThread.getLooper());

```

# 3 Handler导致的内存泄漏原因以及解决方案  

Activity中初始化一个Handler，Handler会持有Activity的引用，如果Handler中有尚未处理的消息，Activity提前销毁，此时Handler仍然持有Activity引用，导致泄漏 

解决方案：静态内部类+弱引用（WeakReference）
```java
public class MainActivity extends AppCompatActivity {
    private static class SafeHandler extends Handler {
        private WeakReference<MainActivity> mActivityRef;

        SafeHandler(MainActivity activity) {
            mActivityRef = new WeakReference<>(activity);
        }

        @Override
        public void handleMessage(Message msg) {
            MainActivity activity = mActivityRef.get();
            if (activity != null) {
                // 更新 UI
            }
        }
    }

    private SafeHandler mHandler = new SafeHandler(this);
}
```

也可以手动移除未执行的消息
```java
// Java 版本
public class MainActivity extends AppCompatActivity {
    private Handler mHandler = new Handler();

    @Override
    protected void onDestroy() {
        mHandler.removeCallbacksAndMessages(null); // 移除所有消息和回调
        super.onDestroy();
    }
}
```

或者用Lifecycle，让Handler与Activity生命周期绑定
```java
// 需 lifecycle 库
public class LifecycleHandler extends Handler implements LifecycleObserver {
    private WeakReference<LifecycleOwner> mOwnerRef;

    public LifecycleHandler(LifecycleOwner owner) {
        mOwnerRef = new WeakReference<>(owner);
        owner.getLifecycle().addObserver(this);
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    private void onDestroy() {
        removeCallbacksAndMessages(null);
        LifecycleOwner owner = mOwnerRef.get();
        if (owner != null) {
            owner.getLifecycle().removeObserver(this);
        }
    }
}
```

Kotlin中还可以结合扩展函数+委托的方式，更优雅的实现  
```kotlin
// 扩展函数：自动移除消息
fun Handler.autoRemove(owner: LifecycleOwner) {
    owner.lifecycle.addObserver(object : LifecycleObserver {
        @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
        fun onDestroy() {
            this@autoRemove.removeCallbacksAndMessages(null)
        }
    })
}

// 使用
private val handler = Handler(Looper.getMainLooper()).apply {
    autoRemove(this@MainActivity)
}
```


# 4 一个线程可以有几个Handler，几个Looper，几个MessageQueue对象  

一个线程可以有多个Handler，只有一个Looper, 一个MessageQueue  
Looper.prepare方法中创建Looper对象，并放入ThradLocal中，将Looper与线程绑定  
ThreadLocal内部维护了一个Map类，以Thread作为key，因此线程和Looper是一一对应关系  
Looper构造方法中创建了MessageQueue对象，所以MessageQueue也只有一个  

# 5 Message对象创建方式有哪些，分别有哪些区别  
1. 直接new  
2. 通过Message.obtain方法，如果消息池存在对象，那么直接返回，复用这个消息，避免对象重复创建，这是个加锁的静态方法，线程安全，如果消息池没有，那么直接new一个    
3. 通过Handler.obtainMessage方法，内部也是Message.obtain  

Message内部持有一个静态Message和私有成员next，    
当有新的Message对象通过MessageQueue进入队列时，静态变量指向自身，next指向上一个Message对象，组成一个单链表，链表最大size限制50  

# 6 Handler有哪些发送消息的方法  

sendMessage(Message msg)
sendMessageDelayed(Message msg, long uptimeMillis)
sendMessageAtTime(Message msg,long when)
post(Runnable r)
postDelayed(Runnable r, long uptimeMillis)

# 7 Handler的post与sendMessage区别和应用场景  

post传递的是Runnable对象，虽然内部还是封装成一个Message，但是无需关心Message传递和处理过程，适合单一线程回调场景  
sendMessage传递的是Message，需要更多的参数，以及重写handleMessage，手动处理消息，更适合多判断条件场景


# 8 Handler postDelay后消息队列有什么变化，假设先postDelay 10s，再postDelay 1s，怎么处理这2条消息  


# 9 MessageQueue是什么数据结构  


# 10 Handler怎么做到一个线程对应一个Looper，如何保证自由一个MessageQueue ThreadLocal在Handler机制中的作用是什么  


# 11 HandlerThread是什么？原理？使用场景有哪些？  
HandlerThread是继承自Thread的线程类，内部有自己的Looper对象，本质上是对Looper和Thread的封装，目的是更方便在子线程中使用Handler机制，适用于子线程处理耗时操作

# 12 IdleHandler使用场景  


# 13 消息屏障，同步屏障机制是什么  


# 14 子线程能不能更新UI  

这是Android规定的机制，所有UI操作最终都会触发`ViewRootImpl.checkThread`方法，如果不是主线程，会直接抛出`CalledFromWrongTheadException`异常，报错信息 `Only the original thread that created a view hierarchy can touch its views`

# 15 为什么Android系统不建议子线程访问UI

在Android中子线程可以有好多个，如果每个子线程都对UI进行访问，会存在多线程对同一资源操作产生线程安全问题，如果用锁机制，会大大降低运行效率，所以出于性能考虑，Android不建议子线程访问UI，并规定UI线程是主线程  

# 16 Android中为什么主线程不会因为Looper.loop死循环卡死  

主线程的Looper.loop是通过**休眠-唤醒机制**和**事件驱动模型**，在保证及时响应UI事件的同时，避免了无意义的CPU消耗  
- 无消息时休眠：当消息队列为空时，Looper通过底层Native层`epoll`机制让线程进入休眠状态  
- 唤醒机制：当有新消息进入，系统会主动唤醒休眠的线程  


# 17 Handler消息机制中，一个Looper是如何区分多个Handler的？当Activity有多个Handler的时候，怎么区分当前消息由哪个Handler处理？处理Message的时候如何区分哪个callback？  

Looper与线程是一一对应关系，Handler能够根据所创建的线程获取对应的Looper  
Message中有个字段是target，用于存储处理它的Handler，处理Message时，会通过target.dispatchMessage来分发消息，包括callback

# 18 Looper.quit/quitSafely的区别  




# 19 通过Handler如何实现线程切换 

当A线程创建Handler时，同时也创建了MessageQueue和Looper，Looper在线程A中会调用loop进入一个无线for循环从MessageQueue中取消息  
当B线程调用A线程创建的handler对象并sendMessage时，会调用A线程的MessageQueue.enqueueMessage，这是个加锁的方法，保证多线程调用下不会存在线程安全问题，这个方法里面有个Native方法`nativeWake`，触发epoll机制唤醒线程A，完成线程B到A的切换  

关键机制：  
1. 线程绑定的MessageQueue，每个Handler绑定一个Looper，而Looper是线程私有的（通过ThreadLocal存储），线程A的Handler只能操作线程A的MessageQueue；  
2. MessageQueue.enqueueMessage内部加锁（synchronized）保证线程安全，即使多个线程同时发消息也不会冲突；  
3. Native提供的线程唤醒机制  

# 20 Handler如何与Looper关联  

Handler内部持有一个Looper对象，Handler在构建时，会通过`Looper.myLooper()`赋值，该静态方法是从Looper内部的ThreadLocal中取Looper，如果为null，直接抛异常。主线程会主动调用Looper.prepareMainLooper, 子线程需要手动调用Looper.prepare，否则直接异常


# 21 Looper如何与Thread关联  

Looper内部有个静态变量`sThradLocal`，用于关联Looper与Thread，所有的Looper都在prepare方法中创建

```java
// Looper.java
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
        }
    sThreadLocal.set(new Looper(quitAllowed));
}

// ThreadLocal.java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        map.set(this, value);
    } else {
        createMap(t, value);
    }
}
```

# 22 Looper.loop源码  
loop核心是一个无限循环的for，阻塞于消息队列的next方法，取出消息后通过msg.target.dispatchMessage来处理消息(target即Handler)
```java
public static void loop() {
    // 1. 获取当前线程的 Looper 实例
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }

    // 2. 获取消息队列
    final MessageQueue queue = me.mQueue;

    // 3. 确保当前线程的 Identity 正确（IPC 相关）
    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();

    // 4. 进入无限循环
    for (;;) {
        // 5. 获取下一条消息（可能阻塞）
        Message msg = queue.next(); // 关键点！
        if (msg == null) {
            // 6. 队列已退出，结束循环
            return;
        }

        // 7. 记录消息分发日志（调试用）
        final long start = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
        try {
            // 8. 分发消息到 Handler
            msg.target.dispatchMessage(msg);
        } finally {
            // 9. 确保消息回收
            msg.recycleUnchecked();
        }

        // 10. 检测慢消息（ANR 监控相关）
        final long end = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
        if (slowDispatchThresholdMs > 0 && (end - start) > slowDispatchThresholdMs) {
            Slog.w(TAG, "Dispatch took " + (end - start) + "ms on " + Thread.currentThread().getName());
        }
    }
}
```

# 23 MessageQueue的enqueueMessage方法如何进行线程同步  

通过`synchronized` 和`nativeWake`方法共同实现线程同步，前者是锁机制，后者是Native方法，epoll机制的一部分

# 24 MessageQueue的next方法内部原理  
`MessageQueue.next`内部也是个死循环，调用了Native方法`nativePollOnce`，这是epoll机制，看似死循环，实际能够释放CPU资源  
`nativePollOnce`能够精准时间的阻塞，在Native层，将进入`pullInner`方法，使用`epoll_wait`阻塞等待以去读管道通知，如果没有得到Native层消息，这个方法不会返回，此时主线程会释放CPU资源进入休眠状态  


# 25 子线程中是否可以用MainLooper去创建Handler，Looper和Handler是否一定处于一个线程？  
可以，直接new即可
```java
Handler handler = new Handler(Looper.getMainLooper());
```
由于Looper与子线程捆绑，此时Looper属于子线程，而Handler内部的Looper属于主线程，那么Handler执行时，会切换到主线程，Handler和Looper不在一个线程中  

# 26 ANR与Handler的联系  

ANR产生的本质原因是主线程执行了耗时操作，导致线程阻塞，通常对于Activity，阻塞5秒会造成ANR，所以我们需要把耗时操作挪到子线程中去，这就需要用Handler切换线程

# 27 handleMessage和dispatchMessage的区别

两者都可以用于处理Message，区别在于dispatchMessage是handleMessage的上游方法，用于全局分发Message和Callback，而handleMessage仅用于处理分发后的Message，是留给开发者的专用接口  
一般而言，如果只是处理Message，那么重写handleMessage方法是比较规范的做法，如果需要接管Message全局处理规则，那么可以重写dispatchMessage，关键源码如下
```java
// Handler.java
public void dispatchMessage(Message msg) {
    if (msg.callback != null) { 
        // 情况1：消息是 Runnable（通过 post() 发送）
        handleCallback(msg); // 直接执行 Runnable
    } else {
        if (mCallback != null) { 
            // 情况2：Handler 设置了全局 Callback
            if (mCallback.handleMessage(msg)) {
                return; // Callback 处理了消息，不再向下传递
            }
        }
        // 情况3：默认调用开发者重写的 handleMessage()
        handleMessage(msg);
    }
}

// 处理 Runnable 消息
private static void handleCallback(Message message) {
    message.callback.run(); // 直接执行 Runnable
}

// 空实现，需开发者重写
public void handleMessage(Message msg) {}
```
补充：这里还有个**`mCallback.handleMessage`**，优先级高于`handleMessage`，主要作用是全局层面，拦截、控制`handlerMessage`的处理逻辑，比如说：

```java
Handler.Callback callback = new Handler.Callback() {
    @Override
    public boolean handleMessage(Message msg) {
        if (msg.what == BLOCKED_MSG) {
            Log.d("Callback", "拦截消息: " + msg.what);
            return true; // 拦截消息，不再传递
        }
        return false; // 允许其他消息继续传递到 handleMessage()
    }
};
Handler handler = new Handler(Looper.getMainLooper(), callback);
```

从Handler设计哲学上来看，这是一种责任链模式的体现，优先顺序为：
1. Message自带的Runnable最先执行；  
2. 全局Callback拦截；  
3. 子类重写的handleMessage  

