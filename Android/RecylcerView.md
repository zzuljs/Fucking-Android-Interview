# 1 RecyclerView的多级缓存机制，每一级缓存具体作用是什么，分别在什么场景下会用到缓存  

## 1.1 RecyclerView缓存机制  

`RecyclerView`有四级缓存，分别是：  

- 一级缓存 `Scrap`  
- 二级缓存 `Cache`  
- 三级缓存 `ViewCacheExtension`  
- 四级缓存 `RecycledViewPool`  

`Scrap`缓存用于`RecyclerView`布局计算期间临时存储即将进入屏幕的`ViewHolder`，用于布局时期的快速复用，布局完成之后就会清空，常用于列表加载时，用户快速翻列表，此时会造成大量布局行为，`Scrap`专用于处理这种情况  

当列表滚动时，`LayoutManager`会先从`Scrap`缓存中取数据，如果有，直接复用，没有，继续向下一集查找，注意`Scrap`缓存仅存在于布局时期，布局结束，缓存清空，缓存数据以`ViewHolder`存储，不会触发`onBindViewHolder`行为    

`Cache`缓存用于处理刚离开屏幕的`ViewHolder`，如用户往下滑动列表此时顶部`item`离开屏幕，布局时期先进入`Scrap`，完成布局进入`Cache`，此时用户往上翻，直接从`Cache`缓存里面取，他们的`View`必须已经从`RecyclerView`中`detached`或`removed`，默认存储离开屏幕的2个`item`，可通过`setItemViewCacheSize`设置      

`ViewCacheExtension`缓存是留给开发者的自定义缓存，常用于缓存固定`item`及状态，如广告位，置顶位等，数据保留状态、匹配规则可由开发者自定义  

`RecycledViewPool`是全局共享缓存池，常用于多个列表页切换，默认每种`item`类型存5个`ViewHolder`，可通过`setMaxRecycledViews`调整，缓存池里的数据已经和数据解绑了，复用时会再次绑定数据（`onBindViewHolder`回调），因此，消耗内存较小，而复用加载需要更长的时间  

`RecyclerView`使用缓存的顺序是先从`Scrap`->`Cache`->`ViewCacheExtension`->`RecycledViewPool`, 如果都找不到，那么`createViewHolder`  

## 1.2 设计理念  

`RecyclerView`缓存机制本质上是以空间换时间，而多级缓存是以场景为驱动，应对更加细化的问题：  

`Scrap`用于解决布局计算期间快速响应需求，用户可能会快速滑动列表的场景，如999+的聊天记录快速上滑  

`Cache`用于解决短时滚动交互需求，如用户上下来回滚动  

`Pool`用于解决全局和跨列表复用需求  

`ViewCacheExtension`则是开放给开发者的缓存接口，优先级在`Scrap`、`Cache`之后  

## 1.3 Scrap和Cache的区别

`Scrap`和`Cache`作用接近，两者在`RecyclerView`中实现都是`ArrayList`，但分工不同  

`Scrap`仅作用于布局阶段，而布局阶段是非常短暂的，因此通常不会太大，查找起来会比较快，时间复杂度理论上是`O(n)`，但实际无限接近于`O(1)`  

`Cache`维持了一个`FIFO`队列结构，用于上下滚动场景，直接从队尾取数据，时间复杂度也是`O(1)`  

# 2 `RecyclerView`的滑动回收复用机制  

`RecyclerView`滑动回收本质上是`View`的动态复用，在滑动过程中，移除屏幕的`View`被回收到缓存，新进入屏幕的`item`优先从缓存池中获取，避免重复创建和布局计算  

从实现上来看，是`LayoutManager`、`Recycler`、`Adapter`和缓存机制的协作  
- `LayoutManager`:负责布局和滚动逻辑  
- `Recycler`:管理`Item`回收和缓存  
- `Adapter`:提供数据绑定机制，`onCreateViewHolder`和`onBindViewHolder`  

当用户滑动屏幕时，`LayoutManager`计算滑动距离，遍历所有`item`是否划出屏幕，如果划出，调用`Recycler.recycleView`方法，该方法用户可以自定义回收哪些资源，并将该`ViewHolder`回收进缓存，如果还没有`detached`，说明还在布局阶段，那么放入`Scrap`缓存，否则放入`Cache`缓存，`Cache`缓存队列满则进入`Pool`缓存  

对于新划入屏幕的`item`，从`Scrap`、`Cache`、`ViewCacheExtension`、`Pool`四级缓存中查找，如果不存在，那么直接调用`Adapter.onCreateViewHolder`创建`item`对应的`ViewHolder`，如果是从`Scrap`、`Cache`取到的缓存，直接用，`VCE`是扩展接口，是否创建`ViewHolder`取决于具体实现，`Pool`会触发`Adapter.onBindViewHolder`，重新绑定数据  

以上就是一次完整的滑动回收  

# 3 `RecyclerView`的刷新回收复用机制  

`notify`后，`RecyclerView`会进行两次布局，一次预布局，一次实际布局，然后执行动画操作  

1.查找改变的`Holder`，保存在`mChangedScrap`中，其他未改变的保存在`mAttachedScrap`中  
2.创建新的`Holder`并绑定数据，充当改变位置的`Holder`，其他`Holder`从`mAttachedScrap`中获取  

# 4 `RecyclerView`为什么要预布局  

预布局是支持动画和高效布局更新的核心机制，目的是在数据变化前记录视图的旧状态、为动画过渡提供基准  
当`RecyclerView`发生数据变动时，若直接应用新布局，会导致以下问题：
- 动画基准缺失：无法获取旧布局的起始位置，无法生成平滑过渡动画  
- 布局闪烁：直接刷新会导致视图短暂错乱，页面闪烁  
- 性能浪费：新布局需要重新计算，无法复用旧布局已有的信息  

实现过程主要对应源码中的 `dispatchLayoutStep1/2/3`  

1. 预布局阶段：当数据发生变动，且需要执行动画时，记录当前屏幕中所有`ViewHolder`的旧位置，然后标记即将改动的`ViewHolder`  
```java
// RecyclerView.java
void dispatchLayoutStep1() {
    // 标记预布局阶段
    mState.mInPreLayout = true;
    // 调用 LayoutManager 的预布局逻辑
    mLayout.onLayoutChildren(mRecycler, mState);
    // 保存旧位置信息
    mState.mOldItemPositions = ...;
}
```

2. 实际布局阶段：计算新数据所有`Item`位置，复用可复用的旧数据的`ViewHolder`
```java
// RecyclerView.java
void dispatchLayoutStep2() {
    mState.mInPreLayout = false;
    mLayout.onLayoutChildren(mRecycler, mState); // 重新布局
}
```

3. 动画布局阶段：对比布局和实际布局的差异，生成动画，然后执行动画  
```java
// RecyclerView.java
void dispatchLayoutStep3() {
    mState.mInPreLayout = false;
    // 触发动画
    mItemAnimator.runPendingAnimations();
}
```

# 5 `ListView`与`RecyclerView`区别  

1. 布局更加灵活  

`RecyclerView`支持更多布局（线性、网格、瀑布流），通过`LayoutManager`动态配置  
`ListView`仅支持垂直滚动列表，灵活性差  

2. `ViewHolder`模式  

`RecyclerView`强制`ViewHolder`模式，通过继承`RecyclerView.ViewHolder`，减少`findViewById`调用，提升性能  
`ListView`模式为可选优化，开发者需要手动实现  

3. 动画支持  

`RecyclerView`内置`ItemAnimator`，支持自定义动效，扩展性强  
`ListView`不支持内置动画，动画需要手动实现，复杂度高  

4. 数据更新  

`RecyclerView`支持局部更新（`notifyItemInserted`、`notifyItemRemoved`），避免全局重绘  
`ListView`仅支持`notifyDataSetChanged`全局刷新，性能较差  

5. 性能优化  

`RecyclerView`提供四级缓存机制，复用机制更高效，支持预加载和嵌套滚动  
`ListView`采用两级缓存机制，复杂列表容易卡顿，嵌套滚动性能差  

# 6 `RecyclerView`性能优化  

1. 数据处理和视图加载分离  

`onBindViewHolder`中仅做UI处理，不做任何耗时操作，业务逻辑尽可能在`onBindViewHolder`完成  

2. 布局优化  

减少过度绘制，减少布局层级  

3. 避免频繁全局更新，用布局更新`notifyItemChanged`、`notifyItemInserted`替代全局更新`notifyDataSetChanged`  

4. 预加载机制  

通过`LinearLayoutManager.setInitialPrefetchItemCount(n)`提前加载`n`个 `item`，避免滑动卡顿

5. 调整缓存大小  

对于内存不敏感的场景，调整`setRecycledViewPool()`和`setMaxRecycledViews(viewType, count)`以优化缓存命中率

