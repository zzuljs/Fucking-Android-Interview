# 1 Fragment的生命周期  

## 1.1 Fragment生命周期概述  

<img src="./../images/Fragment生命周期.png" alt="Fragment生命周期" style="display:block;margin:0 auto" />  
<br/>

<p>
Fragemnt的生命周期函数很多，但实际上内部只定义了6种有效状态  
</p>

```java
static final int INVALID_STATE = -1;   // Invalid state used as a null value.
static final int INITIALIZING = 0;     // Not yet created.
static final int CREATED = 1;          // Created.
static final int ACTIVITY_CREATED = 2; // The activity has finished its creation.
static final int STOPPED = 3;          // Fully created, not started.
static final int STARTED = 4;          // Created and started, not resumed.
static final int RESUMED = 5;          // Created started and resumed.
```

## 1.2 Fragment与Activity生命周期关联  
Fragment与Activity生命周期密切相关，Activity通过FragmentManager来管理所有Fragment，对应关系如下
| Activity生命周期阶段      | FragmentManager触发函数 | Fragment状态迁移路径                  | Fragment生命周期回调函数               |
|--------------------------|-------------------------|---------------------------------------|---------------------------------------|
| ​**onCreate**             | `dispatchCreate`        | INITIALIZING → CREATED               | `onAttach` → `onCreate`              |
| ​**onStart**              | `dispatchStart`         | CREATED → ACTIVITY_CREATED → STOPPED → STARTED | `onCreateView` → `onActivityCreated` → `onStart` |
| ​**onResume**<br>（实际是onPostResume） | `dispatchResume`       | STARTED → RESUMED                    | `onResume`                           |
| ​**onPause**              | `dispatchPause`         | RESUMED → STARTED                    | `onPause`                            |
| ​**onStop**               | `dispatchStop`          | STARTED → STOPPED                    | `onStop`                             |
| ​**onDestroy**            | `dispatchDestroy`       | STOPPED → ACTIVITY_CREATED → CREATED → INITIALIZING | `onDestroyView` → `onDestroy` → `onDetach` |

转换关系图如下：  

![Fragment与Activity生命周期关系](./../images/Fragment生命周期转换图.png)

# 2 Activity与Fragment的通信方式，Fragment之间如何进行通信  
1. 接口回调 interface+callback，适合处理Fragment向Activity传递事件，如按钮点击，需要onDetach中释放引用接口，防止内存泄漏  
2. ViewModel+LiveData，适用于UI共享状态管理  
3. 通过Fragment.setArguments，适用于Fragment初始化时，Activity向Fragment传递数据  
4. 通过Fragment.setFragmentResult，适用于Fragment向Activity、其他Fragment传递数据  
5. EventBus 事件总线，适用于跨组件通信  
6. Activity直接获取Fragment实例来调用具体方法，反之，Fragment也可以通过getActivity调用Activity，这样做并不规范，代码会变得特别耦合  

# 3 为什么使用Fragment.setArguments(Bundle)传递参数  
核心原因在于Activity重建时，通过setArguments方法设置的参数能够恢复  
```java
// Activity.java
@Override
protected void onSaveInstanceState(@NonNull Bundle outState) {
    super.onSaveInstanceState(outState);
    // 通过FragmentController保存Fragment状态
    mFragments.saveAllState();
}

// FragmentManager.java
void saveAllState() {
    // 保存Fragment状态到FragmentManagerState中
    FragmentManagerState fms = new FragmentManagerState();
    // 遍历所有Fragment，保存其状态到FragmentState数组
    fms.mActive = activeFragments.toArray(new FragmentState[0]);
}
```
FragmentManager.saveAllState会把每个Fragment状态封装成FragmentState对象，当Activity重建时恢复  
```java
// Activity.java
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // 恢复Fragment状态
    mFragments.restoreSaveState(savedInstanceState);
}

// FragmentManager.java
void restoreSaveState(@NonNull Parcelable state) {
    FragmentManagerState fms = (FragmentManagerState) state;
    // 遍历FragmentState数组，重新实例化Fragment
    for (FragmentState fs : fms.mActive) {
        Fragment f = fs.instantiate(mHost.getContext(), mFragmentFactory);
        f.setArguments(fs.mArguments);  // 关键：重新设置Arguments
        // 其他状态恢复...
    }
}
```

# 4 FragmentPageAdapter和FragmentStatePageAdapter区别以及使用场景  

| 特性                        | FragmentPagerAdapter                     | FragmentStatePagerAdapter               |
|-----------------------------|-----------------------------------------|-----------------------------------------|
| Fragment销毁方式            | 仅销毁视图（detach），保留Fragment实例  | 完全销毁Fragment实例（remove+状态保存） |
| 内存占用                    | 较高（所有页面常驻内存）                | 较低（仅保留当前及相邻页面）            |
| 状态保存                    | 由Fragment自身维护（实例未销毁）        | 通过onSaveInstanceState保存Bundle       |
| 适用场景                    | 少量静态页面（如标签页）                | 大量动态页面（如图库、电子书）          |

# 5 Fragment懒加载  
Fragment懒加载是一种延迟加载策略，旨在Fragment真正对用户可见时才开始加载数据、执行业务逻辑，避免Fragment过多，同时加载造成的资源竞争、卡顿等问题  

在ViewPager+Fragment体系中，默认会预加载相邻Fragment（通过offscreenPageLimit可以控制，最小值为1），此时即便Fragment不可见，仍然会执行onResume回调。这种预加载机制会导致一定程度的资源浪费，随着Fragmeng嵌套层级加深，这个问题会变得非常严重，所以需要对Fragment的加载时机进行特殊处理    

1. 传统方式ViewPager  
```java
@Override
public void setUserVisibleHint(boolean isVisibleToUser) {
    super.setUserVisibleHint(isVisibleToUser);
    if (isVisibleToUser && isResumed()) {
        // 对用户可见时加载数据
        loadData();
    }
}
```
这种方式需要手动判断Fragment的生命周期，在ViewPager2出来之后已经被抛弃了  

2. 使用Lifecycle 
监听Fragment的生命周期，在onResume时加载数据，用一个变量标记防止重复加载  
```java
class LifecycleAwareFragment : Fragment() {
    private val viewModel: MyViewModel by viewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        lifecycle.addObserver(object : DefaultLifecycleObserver {
            override fun onResume(owner: LifecycleOwner) {
                if (!viewModel.isDataLoaded) {
                    viewModel.loadData()
                }
            }
        })
    }
}
```
3. 使用ViewPager2  
ViewPager2能够监听页面切换，使得当前Fragment被选中时才加载
```java
viewPager2.registerOnPageChangeCallback(object : ViewPager2.OnPageChangeCallback() {
    override fun onPageSelected(position: Int) {
        val fragment = adapter.getFragment(position)
        if (fragment is ModernLazyFragment) {
            fragment.loadDataIfNeeded()
        }
    }
})
```

ViewPager2保证onResume仅在可见时回调，因此也可以用onResume+控制变量的方法
```java
class ModernLazyFragment : Fragment() {
    private var isDataLoaded = false

    override fun onResume() {
        super.onResume()
        if (!isDataLoaded) {
            loadData()
            isDataLoaded = true
        }
    }

    private fun loadData() {
        // 加载数据
    }
}

```

# 6 ViewPager与ViewPager2区别  

1. FragmentStateAdapter 替代FragmentStatePagerAdapter，对生命周期有了更合理的管控， PagerAdapter被RecyclerView.Adapter替代  
2. 支持竖直滑动，禁止滑动  
3. PageTransformer用来设置页面动画，设置页面间距  
4. 预加载当setOffscreenPageLimit被设置为OFFSCREEN_PAGE_LIMIT_DEFAULT时候会使用RecyclerView的缓存机制  

# 7 Fragment嵌套问题  

1. 正确使用FragmentManager，在父Fragmeng中操作子Fragment时，需要调用`getChildFragmentManager`  
2. 布局优化，尽可能减少View嵌套，使用merge标签减少层级，延迟加载子Fragment内容  
3. 返回事件处理，子Fragment需拦截返回键，此时应该在Activity宿主中协调
```java
// 宿主Activity.java
@Override
public void onBackPressed() {
    if (!handleNestedFragmentsBack()) {
        super.onBackPressed();
    }
}
// 检查子Fragment是否消费返回事件
private boolean handleNestedFragmentsBack() {
    Fragment current = getChildFragmentManager().findFragmentById(R.id.container);
    return current instanceof ChildFragment && ((ChildFragment) current).onBackPressed();
}
```
4. Fragment间通信，避免直接引用子Fragmeng实例，使用更优雅的通信方式，如ViewModel，或者Interface+callback  
5. 避免Fragment嵌套过深  