- UIApplication、VC、View生命周期
- 界面相关的知识点

# UIApplication、VC、View生命周期
## UIApplication的生命周期
```
// 应用程序启动完成的时候调用
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions: (NSDictionary *)launchOptions {
    return YES;
}
// 当我们应用程序即将失去焦点的时候调用
- (void)applicationWillResignActive:(UIApplication *)application {
NSLog(@"%s",__func__);
}
// 当我们应用程序完全进⼊后台的时候调用
- (void)applicationDidEnterBackground:(UIApplication *)application{
NSLog(@"%s",__func__);
}
// 当我们应用程序即将进⼊前台的时候调用
- (void)applicationWillEnterForeground:(UIApplication *)application {
NSLog(@"%s",__func__);}
// 当我们应用程序完全获取焦点的时候调用
// 只有当一个应用程序完全获取到焦点,才能与用户交互.
- (void)applicationDidBecomeActive:(UIApplication *)application {
- NSLog(@"%s",__func__);
}
// 当我们应用程序即将关闭的时候调用
- (void)applicationWillTerminate:(UIApplication *)application {
NSLog(@"%s",__func__);
}
```

![iOS启动原理图](https://upload-images.jianshu.io/upload_images/2252551-4c489571cabf79b7.png?imageMogr2/auto-orient/strip|imageView2/2/w/990/format/webp)

## UIViewController 生命周期
```
#pragma mark --- life circle

// 非storyBoard(xib或非xib)都走这个方法
- (instancetype)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil {
    NSLog(@"%s", __FUNCTION__);
    if (self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil]) {
    
    }
    return self;
}

// 如果连接了串联图storyBoard 走这个方法
- (instancetype)initWithCoder:(NSCoder *)aDecoder {
     NSLog(@"%s", __FUNCTION__);
    if (self = [super initWithCoder:aDecoder]) {
        
    }
    return self;
}

// xib 加载 完成
- (void)awakeFromNib {
    [super awakeFromNib];
     NSLog(@"%s", __FUNCTION__);
}

// 加载视图(默认从nib)
- (void)loadView {
    NSLog(@"%s", __FUNCTION__);
    self.view = [[UIView alloc] initWithFrame:[UIScreen mainScreen].bounds];
    self.view.backgroundColor = [UIColor redColor];
}

//视图控制器中的视图加载完成，viewController自带的view加载完成
- (void)viewDidLoad {
    NSLog(@"%s", __FUNCTION__);
    [super viewDidLoad];
}


//视图将要出现
- (void)viewWillAppear:(BOOL)animated {
    NSLog(@"%s", __FUNCTION__);
    [super viewWillAppear:animated];
}

// view 即将布局其 Subviews
- (void)viewWillLayoutSubviews {
    NSLog(@"%s", __FUNCTION__);
    [super viewWillLayoutSubviews];
}

// view 已经布局其 Subviews
- (void)viewDidLayoutSubviews {
    NSLog(@"%s", __FUNCTION__);
    [super viewDidLayoutSubviews];
}

//视图已经出现
- (void)viewDidAppear:(BOOL)animated {
    NSLog(@"%s", __FUNCTION__);
    [super viewDidAppear:animated];
}

//视图将要消失
- (void)viewWillDisappear:(BOOL)animated {
    NSLog(@"%s", __FUNCTION__);
    [super viewWillDisappear:animated];
}

//视图已经消失
- (void)viewDidDisappear:(BOOL)animated {
    NSLog(@"%s", __FUNCTION__);
    [super viewDidDisappear:animated];
}

//出现内存警告  //模拟内存警告:点击模拟器->hardware-> Simulate Memory Warning
- (void)didReceiveMemoryWarning {
    NSLog(@"%s", __FUNCTION__);
    [super didReceiveMemoryWarning];
}

// 视图被销毁
- (void)dealloc {
    NSLog(@"%s", __FUNCTION__);
}
```