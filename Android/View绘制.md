# 1 View绘制流程概述  

## 1.1 Activity触发addView的流程   
ActivityThread.handleResumeActivity->WindowManagerImpl.addView->WindowManagerGlobal->addView->ViewRootImpl.setView->ViewRootImpl.requestLayout->ViewRootImpl.scheduleTraversals->ViewRootImpl.doTraversal->ViewRootImpl.performTraversals->View绘制流程  

## 1.2 View绘制流程  

![View绘制流程](./../images/View绘制流程.png)

View绘制主要三个过程`onMeasure` `onLayout` `onDraw` ，分别对应测量、布局、绘制三个阶段，完成画多大、在哪画、画什么  

### 1.2.1 Measure过程  
Measure阶段的目的是确定每个View的尺寸（width、height），父View需要根据自身的约束条件和子View的LayoutParams，递归计算出所有子View的尺寸  

父View通过measureChildWidthMargins等方法，结合自身约束和子View的LayoutParams，生成子View的MeasureSpec，并调用子View的measure, 子View的measure方法被调用后，会先检测是否需要重新测量，然后调用onMeasure计算自身尺寸，最后保存测量结果（方便其他View调用，避免多次测量），父View在遍历子View的时候同时要考虑计算自身的padding和margin 

一个自定义View的Measure示例
```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    int totalWidth = 0;
    int maxHeight = 0;

    // 遍历所有子View
    for (int i = 0; i < getChildCount(); i++) {
        View child = getChildAt(i);
        if (child.getVisibility() == GONE) continue;

        // 测量子View（考虑margin）
        measureChildWithMargins(child, widthMeasureSpec, 0, heightMeasureSpec, 0);

        // 累加子View宽度（包括margin）
        totalWidth += child.getMeasuredWidth() 
            + ((MarginLayoutParams) child.getLayoutParams()).leftMargin
            + ((MarginLayoutParams) child.getLayoutParams()).rightMargin;

        // 计算最大高度（包括margin）
        maxHeight = Math.max(maxHeight, 
            child.getMeasuredHeight() 
            + ((MarginLayoutParams) child.getLayoutParams()).topMargin
            + ((MarginLayoutParams) child.getLayoutParams()).bottomMargin);
    }

    // 考虑父View的padding
    totalWidth += getPaddingLeft() + getPaddingRight();
    maxHeight += getPaddingTop() + getPaddingBottom();

    // 确定自身尺寸
    setMeasuredDimension(
        resolveSize(totalWidth, widthMeasureSpec),
        resolveSize(maxHeight, heightMeasureSpec)
    );
}
```

### 1.2.2 Layout过程  
Layout过程，即确定每个View在屏幕上的具体位置（left、top、right、bottom），由performTraversals触发，核心逻辑包括View.layout和View.onLayout  
```java
// View.java
public void layout(int l, int t, int r, int b) {
    // 1. 检查是否需要重新布局（新旧位置是否变化）
    if (l != mLeft || t != mTop || r != mRight || b != mBottom) {
        // 2. 设置新的坐标（left, top, right, bottom）
        setFrame(l, t, r, b);
        // 3. 调用onLayout()（ViewGroup需重写以布局子View）
        onLayout(changed, l, t, r, b);
        // 4. 触发布局完成的回调（如OnLayoutChangeListener）
        dispatchLayout();
    }
}

protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
    // 默认空实现，ViewGroup必须重写此方法
}
```

父View需要在自己的onLayout中遍历所有的子View，根据Measure阶段的结果和布局规则，考虑padding和margin，调用子View的layout确定其位置，以LinearLayout为例
```java
// LinearLayout.java
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    int currentLeft = getPaddingLeft(); // 起始位置考虑父View的padding
    for (int i = 0; i < getChildCount(); i++) {
        View child = getChildAt(i);
        if (child.getVisibility() == GONE) continue;

        // 1. 获取子View的测量尺寸
        int childWidth = child.getMeasuredWidth();
        int childHeight = child.getMeasuredHeight();

        // 2. 计算子View的位置（横向排列）
        int childLeft = currentLeft;
        int childTop = getPaddingTop(); // 顶部对齐
        int childRight = childLeft + childWidth;
        int childBottom = childTop + childHeight;

        // 3. 调用子View的layout()
        child.layout(childLeft, childTop, childRight, childBottom);

        // 4. 更新下一个子View的起始位置（考虑margin）
        currentLeft = childRight + ((MarginLayoutParams) child.getLayoutParams()).rightMargin;
    }
}
```

### 1.2.3 Draw过程  

Draw过程是把View呈现到屏幕上的最终步骤，包括六个核心步骤  
1.绘制背景；2.保存图层；3.绘制自身内容；4.绘制子View；5.绘制装饰器（前景、滚动条等）；6.恢复图层

```java
public void draw(Canvas canvas) {
    // 1. 绘制背景（若有）
    drawBackground(canvas);

    // 2. 保存图层（若需要），避免View绘制干扰其他View
    int saveCount = canvas.saveLayer(...);

    // 3. 绘制自身内容（onDraw()）
    onDraw(canvas);

    // 4. 绘制子View（dispatchDraw()）
    dispatchDraw(canvas);

    // 5. 绘制装饰（滚动条、前景等）
    onDrawForeground(canvas);

    // 6. 恢复图层（若保存过）
    if (saveCount >= 0) {
        canvas.restoreToCount(saveCount);
    }
}
```


参考：https://juejin.cn/post/6904192359253147661

# 2 MeasureSpec是什么  

MeasureSpec是一个32位整型值，它的高2位表示测量模式SpecMode，低30位表示某种测量模式下的规格大小SpecSize, MeasureSpec是View类的一个静态内部类， 用来说明如何测量这个View，有三种模式：
1. EXACTLY:精确测量模式，View宽高指定为match_parent或具体数值时生效，表示父View决定子View的大小，这种模式下View的测量值就是SpecSize的值  
2. AT_MOST:最大测量值模式，View宽高指定为wrap_parent生效，此时子View的尺寸不超过父View的最大尺寸  
3. UNSPECIFIED:不指定测量模式，父View没有限制子View的大小，子View可以是想要的任何尺寸，一般用于系统内部  

MeasureSpec通过将SpecMode和SpecSize打包成一个int值避免够多的内存消耗，makeMeasureSpec是打包方法，getMode和getSize是解包方法  


# 3 子View创建MeasureSpec创建规则是什么  

| 父View的MeasureSpec | 子View的LayoutParams       | 子View的MeasureSpec                    | 说明                                      |
|---------------------|---------------------------|----------------------------------------|------------------------------------------|
| ​**EXACTLY**         | 固定值（如100dp）         | `EXACTLY` + 固定值                     | 子View尺寸固定                           |
|                     | `match_parent`            | `EXACTLY` + 父可用空间                 | 子View填满父View剩余空间                 |
|                     | `wrap_content`            | `AT_MOST` + 父可用空间                 | 子View尺寸不超过父View剩余空间           |
| ​**AT_MOST**         | 固定值（如100dp）         | `EXACTLY` + 固定值                     | 子View尺寸固定                           |
|                     | `match_parent`            | `AT_MOST` + 父可用空间                 | 子View尺寸不超过父View剩余空间           |
|                     | `wrap_content`            | `AT_MOST` + 父可用空间                 | 子View尺寸不超过父View剩余空间           |
| ​**UNSPECIFIED**     | 固定值（如100dp）         | `EXACTLY` + 固定值                     | 子View尺寸固定                           |
|                     | `match_parent`            | `UNSPECIFIED` + 父可用空间             | 子View尺寸无限制（如滚动布局测量子View） |
|                     | `wrap_content`            | `UNSPECIFIED` + 父可用空间             | 子View尺寸无限制（如滚动布局测量子View） |

# 4 自定义View,wrap_content不起作用的原因  

1. 未能正确处理onMeasure中的AT_MOST模式  
当子View的LayoutParams为wrap_content时，父View的MeasureSpec=AT_MOST时，此时，如果不做处理，子View会直接占满父View

```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    // 1. 计算View自身内容的尺寸（例如文本、图形的宽高）
    int contentWidth = calculateContentWidth();  // 自定义方法
    int contentHeight = calculateContentHeight();

    // 2. 根据MeasureSpec模式调整尺寸
    int width = resolveSize(contentWidth, widthMeasureSpec);
    int height = resolveSize(contentHeight, heightMeasureSpec);

    // 3. 保存测量结果
    setMeasuredDimension(width, height);
}

// 工具方法：根据MeasureSpec模式修正尺寸
private int resolveSize(int contentSize, int measureSpec) {
    int mode = MeasureSpec.getMode(measureSpec);
    int specSize = MeasureSpec.getSize(measureSpec);
    switch (mode) {
        case MeasureSpec.EXACTLY:
            return specSize; // 必须使用父View指定的尺寸
        case MeasureSpec.AT_MOST:
            return Math.min(contentSize, specSize); // 不能超过父View限制
        case MeasureSpec.UNSPECIFIED:
            return contentSize; // 无限制，直接使用内容尺寸
        default:
            return contentSize;
    }
}
```

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


# 8 Android绘制和屏幕刷新机制的原理 


# 9 Choreography原理  
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

# 14 SurfaceView、TextureView、SurfaceTexture、GLSurfaceView区别  
# 15 getWidth和getMeasureWidth的区别  

getMeasuredWidth获取的是setMeasuredDimension方法的值，由父View的MeasureSpec和子View的LayoutParams共同确定，是一个理论值，有可能与最终可见View大小不同，在measure方法调用之后确定，通常在onMeasure方法中使用  

getWidth获取的是mRight-mLeft的值，这个值在layout之后确定，代表View最终可见宽度真实值，在布局结束后，通常调用这个方法

# 16 invalidate和postInvalidate的区别  

1. invalidate是直接触发的View重绘，需要在主线程中调用.View的invalidate递归调用父View的invalidateChildInParent，然后触发peformTraversals，会导致当前View重绘，由于mLayoutRequested设置为false，onMeasrue和onLayout不会回调，onDraw回调（仅在View大小发生变化时才会测量、布局，除非手动调用了requestLayout）  
2. postInvalidate可以在任意线程调用，把重绘调用转发给内部持有的主线程Handler执行，切回主线程，然后走View.invalidate流程  

# 17 requestlayout、onLayout、onDraw的联系  

requestLayout方法会导致调用measure和layout，不一定会触发onDraw，requestLayout会递归调用父窗口的requestLayout，直到ViewRootImpl.perfomTraversals, 由于requestLayout会把mLayoutRequested设置为true，这个布尔值会控制是否回调onLayout、onMeasure，所以requestLayout一定回调onLayout、onMeasure，这是职责分离设计思路的体现，先要触发onDraw，必须调用invalidate方法  

# 18 LinearLayout、FrameLayout、RelativeLayout哪个效率高  
FrameLayout>LinearLayout(无权重)>LinearLayout(有权重)>RelativeLayout  

1. FrameLayout  
仅需要将子View堆叠排列，所有子View测量和布局一次完成，适用于子View需要重叠显示（如Fragment容器）场景，无法处理复杂布局  

2. LinearLayout  
LinearLayout会按照排布方向进行测量，没有权重设置时，子View不存在依赖关系，只需要一次测量，如果有权重会有两次测量，适用于线性排列布局  

3. RelativeLayout  
子View之间存在依赖关系，可能会需要多次测量

# 19 LinearLayout的绘制流程  

LinearLayout是View的子类，绘制流程严格遵守View绘制流程，即Measure->Layout->Draw  

## 19.1 Measure

核心任务：计算自身和子View的尺寸  
步骤：1.确定排列方向（垂直 VERTICAL, 水平 HORIZONTAL）；2.遍历所有子View；3.处理权重（weight），如果设置了权重，那么需要测量2次，第一次先遍历所有子View，计算总尺寸，第二次根据权重分配尺寸  
```java
// LinearLayout.java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    if (mOrientation == VERTICAL) {
        measureVertical(widthMeasureSpec, heightMeasureSpec);
    } else {
        measureHorizontal(widthMeasureSpec, heightMeasureSpec);
    }
}

private void measureVertical(int widthMeasureSpec, int heightMeasureSpec) {
    // 遍历子 View，计算总高度和最大宽度
    for (int i = 0; i < count; i++) {
        final View child = getVirtualChildAt(i);
        measureChildBeforeLayout(child, i, widthMeasureSpec, 0, heightMeasureSpec, totalWeight == 0 ? mTotalLength : 0);
        // 处理 weight 逻辑...
    }
    // 确定自身尺寸
    setMeasuredDimension(resolveSize(mTotalWidth, widthMeasureSpec), resolveSize(heightSize, heightMeasureSpec));
}
```


## 19.2 Layout  

核心任务：确定子View的位置  
步骤：1.确定起始位置，考虑自身的padding和margin；2.按方向排列；3.处理权重；4.处理子View的gravity  

```java
// LinearLayout.java
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    if (mOrientation == VERTICAL) {
        layoutVertical(l, t, r, b);
    } else {
        layoutHorizontal(l, t, r, b);
    }
}

private void layoutVertical(int left, int top, int right, int bottom) {
    int childTop = mPaddingTop; // 起始位置
    for (int i = 0; i < count; i++) {
        final View child = getVirtualChildAt(i);
        // 确定子 View 的 left/top/right/bottom
        setChildFrame(child, childLeft, childTop, childWidth, childHeight);
        childTop += childHeight + lp.bottomMargin; // 更新下一个子 View 的起始位置
    }
}
```

## 19.3 Draw阶段  
核心任务：将View绘制到屏幕上  
步骤：1.绘制自身背景；2.调用每个子View的draw方法；3.绘制分割线  

```java
// LinearLayout.java
@Override
public void onDraw(Canvas canvas) {
    if (mDivider == null) return;

    if (mOrientation == VERTICAL) {
        drawDividersVertical(canvas);
    } else {
        drawDividersHorizontal(canvas);
    }
}

private void drawDividersVertical(Canvas canvas) {
    for (int i = 0; i < count; i++) {
        final View child = getVirtualChildAt(i);
        if (child != null && child.getVisibility() != GONE) {
            // 在子 View 下方绘制分割线
            drawVerticalDivider(canvas, child.getBottom() + params.bottomMargin);
        }
    }
}
```

# 20 自定义View的流程和注意事项  

1. 自定义View  
onMeasure：可以不重写，但是如果不重写就需要在外部指定宽高，一般会重写  
onDraw：如果有要画的内容就重写  
onTouchEvent：需要交互就重写  

2. 自定义ViewGroup  
onMeasure：必须重写，这里测量每一个子View，还有处理自己的尺寸  
onLayout：必须重写，对子View进行布局  
onTouchEvent：如果有触摸事件，那么重写  
onDraw：如果想在ViewGroup中画点东西，但是又不设置background，那么会画不出来，需要setWillNotDraw=false  

注意：如果自定义布局属性，需要在构造方法中获取属性值后及时调用recycle回收资源，onDraw和onTouchEvent都尽量避免创建对象，避免卡顿  

# 21 自定义View如何考虑机型适配  
1. 合理使用warp_content/match_parent  
2. 尽可能使用ConstraintLayout  
3. 针对不同的机型、屏幕dp，使用不同的布局文件  
4. 尽量使用9图  
5. 使用dp而非sp

# 22 自定义控件优化方案  

1. onDraw不要有耗时操作  
2. 避免过度绘制，移除不必要的背景，减少背景绘制（对于ViewPager+多个Fragment组成的首页界面，若每个Fragment都设有背景色，ViewPager没有设置的必要，可以移除，不然会多次绘制）  
3. 减少不必要的View嵌套，UI层级过高会导致卡顿  
4. 使用clipRect/quickReject给Canvas设置裁剪区，局部绘制


# 23 invalidate如何局部刷新  
View.invalidate的局部刷新可以通过指定脏区（Dirty Rect）来实现
```java
// 指定矩形区域（主推方案）
public void invalidate(Rect dirty)
public void invalidate(int left, int top, int right, int bottom)

// 指定单个坐标点（适用于点击反馈等场景）
public void invalidate(int l, int t, int r, int b)
```
原理：系统会将多次invalidate调用的区域合并成为一个最小包围矩形，在Android 4+开启硬件加速后，系统会自动进行区域裁剪，仅重绘脏区  

使用举例：
```java
public class CustomView extends View {
    public CustomView(Context context) {
        super(context);
        // 关键设置：允许View保存绘制内容（默认为true）
        setWillNotDraw(false);
    }

    // 示例：在触摸事件中局部刷新
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                // 计算需要刷新的区域（示例为以触摸点为中心的50x50区域）
                int size = 50;
                invalidate(
                    (int)event.getX() - size/2,
                    (int)event.getY() - size/2,
                    (int)event.getX() + size/2,
                    (int)event.getY() + size/2
                );
                return true;
        }
        return super.onTouchEvent(event);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        // 获取当前脏区域
        Rect clipBounds = canvas.getClipBounds();
        if (clipBounds.isEmpty()) {
            return; // 无需要绘制的区域
        }

        // 仅绘制脏区域内的内容（关键优化）
        if (clipBounds.intersect(mDrawArea)) {
            // 执行局部绘制操作
            canvas.drawRect(clipBounds, mPaint);
        }
    }
}
```

# 24 setContentView加载流程  

setContentView是Activity构建UI的核心入口，调用链：Activity.setContentView->PhoneWindow.setContentView->LayoutInflater.inflate->View树构建->DecorView添加到Window  

## 24.1 PhoneWindow初始化  
PhoneWindow是Activity窗口实现类，继承自抽象类Window，通过WindowManager添加、删除、更新，主要用来管理窗口装饰（如状态栏），内容容器创建，与Activity生命周期相同
```java
// Activity.java
public void setContentView(@LayoutRes int layoutResID) {
    getWindow().setContentView(layoutResID); // getWindow() 返回 PhoneWindow 实例
    initWindowDecorActionBar(); // 初始化 ActionBar（若有）
}
```

## 24.2 DecorView初始化  
```java
// PhoneWindow.java
public void setContentView(int layoutResID) {
    if (mContentParent == null) {
        installDecor(); // 创建 DecorView 和 mContentParent
    }
    // 将用户布局添加到 mContentParent
    mLayoutInflater.inflate(layoutResID, mContentParent);
}
```



# 25 View、DecorView、Window之间的关系