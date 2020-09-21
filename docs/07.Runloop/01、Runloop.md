### Runloop
#### Runloop是什么？
runloop本质是一个while循环，当有任务需要处理时就开始处理任务，没有任务处理时就进入休眠状态。

#### Runloop和线程的关系
1. runloop和线程是一一对应的关系，主线程的runloop自动创建，子线程的runloop需要开发者手动创建
2. runloop在第一次获取时创建，在线程结束时销毁

#### runloop的mode是什么
mode指的是runloop运行模式。在runloop中含有多个模式，运行时只能选择一种模式运行。
系统默认注册5个Mode：
1. kCFRunLoopDefaultMode：App默认mode，通常主线程在这个mode下运行
2. UITrackingRunLoopMode：界面跟踪mode，用于ScrollView追踪触摸滑动，保证滑动时不受其他mode影响
3. kCFRunLoopCommonModes: 占位用的mode，作为标记kCFRunLoopDefaultMode和UITrackingRunLoopMode用
4. UIInitializationRunLoopMode: 启动App时进入的第一个mode，启动完成后就不再使用。
5. GSEventReceiveRunLoopMode: 接受系统事件的内部mode，通常用不到。

#### Runloop的运行流程

![012.png](https://github.com/pengwj/iOSInterview/blob/master/res/012.png)

#### Runloop的应用
+ NSTimer在子线程开启定时器
+ 控制定时器在特定模式下执行
+ 自动释放池



#### 参考资料
+ runloop：http://blog.csdn.net/hherima/article/details/51746125

