# 第三方库源码分析总结

## Okhttp


## Glide


## MMKV


## LeakCanary2.0
### 初始化
- 利用ContentProvider在Applicant之前执行实现自动初始化
### 检测对象
- Activity:  
    >通过Application#registerActivityLifecycleCallbacks(…)接口监听Activity#onDestroy事件
- Fragment:  
    >通过Application#registerActivityLifecycleCallbacks(…)接口监听onActivityCreated
  > 获取到activity#FragmentManager, 通过FragmentManager#registerFragmentLifecycleCallbacks
  > 监听Fragment#onDestroy事件
- ViewModel:  
    >在Activity#onCreate和Fragment#onCreate事件中实例化一个自定义ViewModel,在进入ViewModel#onClear()方法时,通过hook获取ViewModelMap,把Map中所有的ViewModel对象交给ObjectWatcher监控
- RootView:  
    >hook WMS RootView列表,检查view对应window类型(通过类型判断view是否存在于Activity或Fragment,不属于才监听),注册View#addOnAttachStateChangeListener()监听
- Service:  
    >hook mHandle#mCallback回调,监听STOP_SERVICE暂存Service,使用动态代理Hook AMS 中IActivityManager#Binder对象，代理serviceDoneExecuting()方法(等同于Service#onDestroy),拿到暂存的Service对象交给ObjectWatcher监控
### 检测原理
- 在对象销毁的时机把对象添加到弱引用WeakReference并且关联上引用队列ReferenceQueue,如果弱引用持有的对象被回收会把弱应用加入到关联的队列Queue,通过此机制判断对象是否泄露。
### 检测流程
-  在对象销毁的时候通过检测对象创建一个弱引用,并且关联一个引用队列,把弱应用储存到map,通过mainHandle#postDelay()延时5秒对象被GC后(系统gc会在对象未被强引用之后5秒内执行)检测弱应用是否出现在引用队列,移除Map中出现在队列中的对象(即未泄露对象),剩下的就是泄露的对象.
## ARoute