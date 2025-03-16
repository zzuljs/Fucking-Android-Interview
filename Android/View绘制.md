1 View绘制流程  
# 2 MeasureSpec是什么  



3 子View创建MeasureSpec创建规则是什么  
4 自定义View,wrap_content不起作用的原因  
# 5 在Activity中获取某个View的宽高有几种方法  

1. 使用View.post

View.post发生于View被添加后，此时View宽高已经确定，直接获取即可
```java
View view = findViewById(R.id.my_view);
view.post(() -> {
    int width = view.getWidth();
    int height = view.getHeight();
    Log.d("ViewSize", "Width: " + width + ", Height: " + height);
});
```

2. Activity.onWindowFocusChanged

当View首次完成布局，Activity获得焦点，该方法回调，此时可以准确测量宽高  
```java
@Override
public void onWindowFocusChanged(boolean hasFocus) {
    super.onWindowFocusChanged(hasFocus);
    // hasFocus表示首次获得焦点
    if (hasFocus) {
        View view = findViewById(R.id.my_view);
        int width = view.getWidth();
        int height = view.getHeight();
        Log.d("ViewSize", "Width: " + width + ", Height: " + height);
    }
}
```
注意：onWindowFocusChanged方法可能会反复回调，仅当hasFocus==true时才是首次完成布局  

3. 监听ViewTreeObserver  

该方法能准确获取全局View.layout时机

```java
View view = findViewById(R.id.my_view);
ViewTreeObserver observer = view.getViewTreeObserver();
observer.addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
    @Override
    public void onGlobalLayout() {
        // 移除监听，避免重复调用
        view.getViewTreeObserver().removeOnGlobalLayoutListener(this);
        int width = view.getWidth();
        int height = view.getHeight();
        Log.d("ViewSize", "Width: " + width + ", Height: " + height);
    }
});
```

4. 手动measure，触发测量, 提前完成View绘制  

```java
View view = findViewById(R.id.my_view);
int widthSpec = View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED);
int heightSpec = View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED);
view.measure(widthSpec, heightSpec);
int measuredWidth = view.getMeasuredWidth(); // 测量宽度
int measuredHeight = view.getMeasuredHeight(); // 测量高度
```

这种做法会破坏View绘制流程，适合自定义View，不适合原生View  

5. 在addOnLayoutChangeListener回调中测量

```java
View view = findViewById(R.id.my_view);
view.addOnLayoutChangeListener(new View.OnLayoutChangeListener() {
    @Override
    public void onLayoutChange(View v, int left, int top, int right, int bottom, 
            int oldLeft, int oldTop, int oldRight, int oldBottom) {
        // 移除监听，避免重复调用
        view.removeOnLayoutChangeListener(this);
        int width = view.getWidth();
        int height = view.getHeight();
        Log.d("ViewSize", "Width: " + width + ", Height: " + height);
    }
});
```
适用于View变化比较频繁的场景，如动画导致的View大小变化  

# 6 为什么onCreate获取不到View的宽高  

Activity在执行完onResume之后才创建ViewRootImpl, 调用链如下：  
startActivity->ActivityThread.handleLaunchActivity->onCreate->完成DecorView和Activity的创建->handleResumeActivity->onResume->DecorView添加到WindowManager->ViewRootImpl.performTraversals->View.measure+layout+draw->DecorView自上而下遍历整个View树  

# 7 View.post和Handler.post的区别  

View.post一定在主线程中执行，调用时机在View被添加之后(`View.dispatchAttachedToWindow`), 在View被添加之后第一时间被执行  
Handler.post可以运行于任意线程，调用立即插入MessageQueue队列，等待Looper.loop触发调用

```java
// View.java
public boolean post(Runnable action) {
    final AttachInfo attachInfo = mAttachInfo;
    if (attachInfo != null) {
        // 情况1：View已附加到Window，直接使用主线程Handler
        return attachInfo.mHandler.post(action);
    }
    // 情况2：View未附加到Window，暂存任务到RunQueue
    getRunQueue().post(action);
    return true;
}

// 内部类HandlerActionQueue（即RunQueue）
private HandlerActionQueue mRunQueue;
public void post(Runnable action) {
    postDelayed(action, 0);
}
public void postDelayed(Runnable action, long delayMillis) {
    HandlerAction handlerAction = new HandlerAction(action, delayMillis);
    synchronized (mActions) {
        mActions.add(handlerAction);
    }
}

```
主要区别

| 特性             | **View.post(Runnable)**                                                 | **Handler.post(Runnable)**                                              |
|------------------|------------------------------------------------------------------------|-------------------------------------------------------------------------|
| **依赖的线程**    | 强制使用主线程的 Handler（通过 AttachInfo 获取）                        | 依赖关联的 Looper，可以是任意线程                                        |
| **执行时机**      | 若 View 未附加到 Window，任务暂存，附加后执行                           | 立即插入消息队列，不等待 View 生命周期状态                               |
| **生命周期安全性** | 自动关联 View 的附加状态，避免在未准备好时执行                           | 需手动管理生命周期（如防止 Activity 销毁后的泄漏）                       |
| **异常处理**      | 无需关心 Looper 是否存在（内部已处理）                                  | 若未绑定 Looper，直接抛出 RuntimeException                              |
| **典型使用场景**   | 在 View 布局完成后获取宽高或更新 UI                                     | 跨线程通信、定时任务等通用场景                                           |


8 Android绘制和屏幕刷新机制的原理  
9 Choreography原理  
# 10 什么是双缓冲  

通俗来讲就是有两个缓冲区，一个后台缓冲区和一个前台缓冲区，每次后台缓冲区接受数据，当填充完整后交换给前台缓冲，这样就保证了前台缓冲里的数据都是完整的。   
Surface 对应了一块屏幕缓冲区，是要显示到屏幕的内容的载体  
每一个 Window 都对应了一个自己的Surface，这里说的 window 包括 Dialog, Activity, Status Bar等，SurfaceFlinger 最终会把这些 Surface 在 z 轴方向上以正确的方式绘制出来（比如 Dialog 在 Activity 之上）  
SurfaceView的每个 Surface 都包含两个缓冲区，而其他普通 Window 的对应的 Surface则不是 

```java
// 子线程中的绘制循环
public void run() {
    while (isRendering) {
        Canvas canvas = null;
        try {
            // 1. 锁定后台缓冲区
            canvas = surfaceHolder.lockCanvas();
            // 2. 绘制内容（覆盖整个缓冲区以避免残留）
            canvas.drawColor(Color.BLACK); // 清空背景
            drawGameFrame(canvas);         // 绘制游戏内容
        } finally {
            // 3. 提交并交换缓冲区
            if (canvas != null) {
                surfaceHolder.unlockCanvasAndPost(canvas);
            }
        }
    }
}
```


# 11 什么是SurfaceView，为什么使用SurfaceView  

SurfaceView是View的子类，实现了Parcelable接口，内嵌了一个专门用于绘制的Surface，SurfaceView可以控制这个Surface的大小、位置，使用双缓冲机制，独立线程绘制View，SurfaceView虽然独立绘制视图，但是也可以作为宿主视图ViewTree的一个节点，参与到宿主ViewTree绘制流程  
从SurfaceView类的成员函数draw和dispatchDraw的实现可以看出，SurfaceView在其宿主窗口绘图表面上所做的操作就是将自己所占区域绘制成黑色，用户需要重写onDraw方法填充自己的内容  

View是通过刷新来重绘视图，系统通过发出VSSYNC信号来进行屏幕的重绘，刷新的时间间隔是16ms，如果界面更新频繁，绘制逻辑复杂，有可能会造成界面卡顿，采用SurfaceView就不存在这个问题

# 12 View和SurfaceView的区别  

View适用于主动更新的情况，SurfaceView适用于被动更新的情况，比如频繁刷新页面，View在主线程中对页面进行刷新，而SurfaceView则需要开启一个子线程进行刷新，View绘图时没有实现双缓冲机制，SurfaceView在底层机制中包含双缓冲机制  

# 13 SufraceView为什么可以直接子线程绘制  

View之所以必须在主线程中更新，是因为View更新时调用一系列ViewRootImpl.performXXX方法，这些方法都会checkThread，这是Android单线程模型的约束  
SurfaceView绕过了传统的View模型，不参与View的绘制，使用独立的Surface，可以视为一个独立的图层，底层使用双缓冲技术，使View更新由子线程驱动，释放了主线程资源，适合一些无需计较资源消耗的高性能页面（如游戏、相机等）更新，代价是更高的内存占用、更高的系统资源消耗（如耗电）

14 SurfaceView、TextureView、SurfaceTexture、GLSurfaceView区别  
# 15 getWidth和getMeasureWidth的区别  

getMeasuredWidth获取的是setMeasuredDimension方法的值，由父View的MeasureSpec和子View的LayoutParams共同确定，是一个理论值，有可能与最终可见View大小不同，在measure方法调用之后确定，通常在onMeasure方法中使用  

getWidth获取的是mRight-mLeft的值，这个值在layout之后确定，代表View最终可见宽度真实值，在布局结束后，通常调用这个方法

# 16 invalidate和postInvalidate的区别  

1.invalidate是直接触发的View重绘，需要在主线程中调用.View的invalidate递归调用父View的invalidateChildInParent，然后触发peformTraversals，会导致当前View重绘，由于mLayoutRequested设置为false，onMeasrue和onLayout不会回调，onDraw回调（仅在View大小发生变化时才会测量、布局，除非手动调用了requestLayout）  
2. postInvalidate可以在任意线程调用，把重绘调用转发给内部持有的主线程Handler执行，切回主线程，然后走View.invalidate流程  

# 17 requestlayout、onLayout、onDraw的联系  

requestLayout方法会导致调用measure和layout，不一定会触发onDraw，requestLayout会递归调用父窗口的requestLayout，直到ViewRootImpl.perfomTraversals, 由于requestLayout会把mLayoutRequested设置为true，这个布尔值会控制是否回调onLayout、onMeasure，所以requestLayout一定回调onLayout、onMeasure，这是职责分离设计思路的体现，先要触发onDraw，必须调用invalidate方法  

18 LinearLayout、FrameLayout、RelativeLayout哪个效率高  
19 LinearLayout的绘制流程  
20 自定义View的流程和注意事项  
21 自定义View如何考虑机型适配  
22 自定义控件优化方案  
23 invalidate如何局部刷新  
24 View加载流程  