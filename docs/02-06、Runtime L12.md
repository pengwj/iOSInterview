[TOC]

# 什么是runtime？
[苹果官方解答](https://developer.apple.com/documentation/objectivec/objective-c_runtime)
[官方文档翻译](https://blog.csdn.net/coyote1994/article/details/52441513)

[运行系统](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008048)
+ runtime是一个运行时库，它为Objective-C语言的动态特性提供支持。OC语言中，尽可能的将对象的类别、调用方法，从编译链接时推迟到运行时，在OC代码被编译后，还需要runtime来配合执行编译后的代码。
+ runtime是一个运行时库，它为Objective-C语言的动态特性提供支持。OC语言中，尽可能的将对象的类别、调用方法，从编译链接时推迟到运行时，为了达到这个效果runtime起了很重要的作用(比如动态的创建类、生成类、给类添加属性方法)。

---
苹果官网：
+ 将尽可能多的决策从编译时和链接时推迟到运行时。
+ 充当着OC语言的操作系统，它使语言能够正常工作。
---

+ runtime是一个运行时库，他为OC语言的动态特性提供支持。可以动态的创建类、添加修改类的属性方法等，实现消息发送和消息转发。

+ runtime是一个运行时库，是由C、C++、汇编实现的API，他为OC语言的动态特性提供支持。调用的OC方法时，本质调用了Runtime 的API。


+ 运行时库：程序运行的时候所需要依赖的库

关键词：运行时库、OC语言的动态特性，动态的创建类、添加修改类的属性方法等。

## OC的动态特性：
+ OC的动态特性表现为了三个方面：动态类型、动态绑定、动态加载
### 动态类型
+ 动态类型，说简单点就是id类型。动态类型是跟静态类型相对的。像内置的明确的基本类型都属于静态类型(int、NSString等)。静态类型在 编译的时候就能被识别出来。所以，若程序发生了类型不对应，编译器就会发出警告。而动态类型就编译器编译的时候是不能被识别的，要等到运行时(run time)，即程序运行的时候才会根据语境来识别。所以这里面就有两个概念要分清：编译时跟运行时。

### 动态绑定
+ 动态绑定(dynamic binding)貌似比较难记忆，但事实上很简单，只需记住关键词@selector/SEL即可。先来看看“函数”，对于其他一些静态语言，比如 c++,一般在编译的时候就已经将将要调用的函数的函数签名都告诉编译器了。静态的，不能改变。而在OC中，其实是没有函数的概念的，我们叫“消息机 制”，所谓的函数调用就是给对象发送一条消息。这时，动态绑定的特性就来了。OC可以先跳过编译，到运行的时候才动态地添加函数调用，在运行时才决定要调 用什么方法，需要传什么参数进去。这就是动态绑定，要实现他就必须用SEL变量绑定一个方法。最终形成的这个SEL变量就代表一个方法的引用。这里要注意 一点：SEL并不是C里面的函数指针，虽然很像，但真心不是函数指针。SEL变量只是一个整数，他是该方法的ID，@selector()就是取类方法的编号。以前的函数调用，是根据函数名，也就是 字符串去查找函数体。但现在，我们是根据一个ID整数来查找方法，整数的查找字自然要比字符串的查找快得多！所以，动态绑定的特定不仅方便，而且效率更 高。

+ 由于OC的动态特性，在OC中其实很少提及“函数”的概念，传统的函数一般在编译时就已经把参数信息和函数实现打包到编译后的源码中了，而在OC中最常使 用的是消息机制。调用一个实例的方法，所做的是向该实例的指针发送消息，实例在收到消息后，从自身的实现中寻找响应这条消息的方法

### 动态加载
+ 根据需求加载所需要的资源，这点很容易理解，对于iOS开发来说，基本就是根据不同的机型做适配。最经典的例子就是在Retina设备上加载@2x的图片，而在老一些的普通屏设备上加载原图。

# Runtime用来干什么？
+ 在程序运行过程中，动态的创建类，动态添加、修改类的属性和方法
+ 遍历一个类中所有的成员变量、属性、以及所有方法
+ 消息传递、转发，方法交换

# Runtime的应用
## 系统的应用
+ KVC、KVO

## 常见应用
+ 动态交换两个方法的实现（比如hock系统方法）
+ 获取对象的属性、私有属性
    + 通过runtime方法打印Ivar数组
    + 字典转模型（利用runtime遍历所有的属性或者成员变量，再利用KVC赋值）
    + 自动归档解档(NSKeyedArchiver归档、NSKeyedUnarchiver解档)
+ crash崩溃防护（利用消息转发机制）（数组、字典、Unrecognize Selector、字符串越界、KVO、通知）
+ 给系统类添加属性、方法(分类+关联对象，或者直接通过addIvar等方法)
+ 动态的创建类，NSClassFromString class<->字符串 （反射机制）

# OC的消息机制
+ oc中的方法调用其实都是转换成了objc_msgSend函数的调用，给receriver（方法调用者）发送消息（selector方法）
+ objc_msgSend底层有三个阶段
    + 消息发送（当前类、父类中的查找）
    + 消息转发
        + 动态方法解析
        + 完整的消息转发机制
 
## 消息发送阶段（消息查找）
+ 先判断消息接受者是否为nil，如果为nil直接退出
+ 从消息接受者的cache（方法缓存）中查找，如果查到就直接调用
+ 再从消息接受者的class_rw_t（方法列表）中查找，如果查到就直接调用，并缓存到方法缓存列表中
+ 再从父类的缓存中查找，如果找到就直接调用，并缓存到消息接受者的缓存中
+ 再从父类的方法列表中查找，如果找到就直接调用，并缓存到消息接受者的缓存中
+ 重复以上操作，如果直到基类还没找到方法的实现，就进入动态方法解析阶段


## 消息转发机制
消息转发分为三个阶段，动态方法解析阶段，快速转发阶段，以及完整的消息转发阶段。

[iOS Runtime 消息转发机制原理和实际用途](https://www.jianshu.com/p/fdd8f5225f0c)

![image](https://upload-images.jianshu.io/upload_images/1322408-8c20d8a1db5a2de8.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/690/format/webp)

### 动态方法解析阶段
+ 先判断是之前是否动态解析过，如果有就进入消息转发阶段
+ 反之，调用```+resolveInstanceMethod:```或者```+resolveClassMethod:```方法来动态解析方法
+ 标记为已经动态解析，并重启消息发送阶段
+ 可以在其中为当前类，动态的添加方法实现

### 快速消息转发阶段
+ 调用```forwardingTargetForSelector:```方法，如果返回值不为nil，就调用```objc_msgSend(返回值,SEL)```
+ 可以在其中返回一个可以处理该方法的对象（提供一个备用对象）

### 完整的消息转发机制
+ 调用```methodSignatureForSelector:```方法,在其中返回SEL方法的签名
+ 如果第一步返回值不为nil，调用```forwardInvocation:```方法，返回备用对象
+ 如果第二步返回值为空，调用```doesNotRecognizeSelector:```方法

## 浅谈iOS之weak底层实现原理
weak是Runtime维护了一个hash(哈希)表，用于存储指向某个对象的所有weak指针。weak表其实是一个hash（哈希）表，Key是所指对象的地址，Value是weak指针的地址（这个地址的值是所指对象指针的地址）数组。

### weak释放为nil过程
- weak被释放为nil，需要对对象整个释放过程了解，如下是对象释放的整体流程：
- 1、调用objc_release
- 2、因为对象的引用计数为0，所以执行dealloc
- 3、在dealloc中，调用了_objc_rootDealloc函数
- 4、在_objc_rootDealloc中，调用了object_dispose函数
- 5、调用objc_destructInstance
- 6、最后调用objc_clear_deallocating。

对象准备释放时，调用clearDeallocating函数。clearDeallocating函数首先根据对象地址获取所有weak指针地址的数组，然后遍历这个数组把其中的数据设为nil，最后把这个entry从weak表中删除，最后清理对象的记录。

[浅谈iOS之weak底层实现原理](https://blog.csdn.net/future_one/article/details/81606895)

https://www.jianshu.com/p/f331bd5ce8f8

https://blog.csdn.net/streamery/article/details/107653266

# runtime的api
## 类相关的api
+ 动态创建一个类（参数：父类，类名，额外的内存空间）
    + Class objc_allocateClassPair(Class superclass,const char* name, size_t extraBytes)

+ 注册一个类（要在类注册之前添加成员变量）
    + void objc_registerClassPair(Class cls)
+ 销毁一个类
    + void objc_disposeClassPair(Class cls)
+ 获取isa指向的Class
    + Class object_getClass(id obj)
+ 设置isa指向的Class
    + Class object_setClass(id obj,Class cls)
+ 判断一个OC对象是否为Class
    + BOOL object_isClass(id obj)
+ 判断一个Class是否为元类
    + BOOL class_isMetaClass(Class cls)
+ 获取父类
    + Class class_getSuperclass(Class cls)
    
### 成员变量相关的api
+  获取一个实例变量信息
    + Ivar class_getInstanceVariable(Class cls,const char *name)
+ 拷贝实例变量列表（最后需要调用free释放）
    + Ivar *class_copyIvarList(Class cls, unsigned int *outCount)
+ 设置和获取成员变量的值
    + void object_setIvar(id obj, Ivar ivar,id value)
    + id object_getIvar(id obj, Ivar ivar)
+ 动态添加成员变量（已经注册的类不能动态添加成员变量）
    + BOOL class_addIvar(Class cls, const char* name, size_t size,uint8_t alignment,const char *types)
+ 获取成员变量的相关信息
    + const char *ivar_getName(Ivar v)
    + const char *ivar_getTypeEncoding(Ivar v)

## 属性相关的api
+ 获取一个属性
    + objc_property_t class_getProperty(Class cls, const char *name)
+ 拷贝属性列表（最后需要调用free释放）
    + objc_property_t class_copyPropertyList(Class cls,unsigned int *outCount)
+ 动态添加属性
    + BOOL class_addProperty(Class cls,const char *name, const objc_property_attribute_t *attributes,unsigned int attributeCount)
+ 动态替换属性
    + void class_replaceProperty(Class cls,const char* name, const objc_property_attribute_t *attributes,unsigned int attributeCount)
+ 获取属性的一些信息
    + const char *property_getName(objc_property_t property)
    + const char *property_getAttributes(objc_property_t property)

## 方法相关的api
+ 获取一个实例方法
    + Method class_getInstanceMethod(Class cls,SEL name)
    + Method class_getClassMethod(Class cls,SEL name)
+ 方法实现相关操作
    + IMP clas_getMethodImplementation(Class cls, SEL name)
    + IMP method_setImplementation(Method m,IMP imp)
    + void method_exchangeImplementations(Method m1,Method m2)
+ 拷贝方法列表（最后需要调用free释放）
    + Method *class_copyMethodList(Class cls,unsigned int *outCount)
+ 动态添加方法
    + BOOL class_addMethod(Class cls,SEL name,IMP imp,const char *types)
+ 动态替换方法
    + IMP class_replaceMethod(Class cls, SEL name, IMP imp, const char *types)
+ 获取方法的相关信息（带有copy的需要调用free去释放）
    + SEL method_getName(Method m)
    + IMP method_getImplementation(Method m)
    + const char *method_getTypeEncoding(Method m)
    + unsigned int method_getNumberOfArguments(Method m)
    + char *method_copyReturnType(Method m)
    + char *method_copyArgumentType(Method m,unsigned int index)
+ 选择器相关
    + const char *sel_getName(SEL sel)
    + SEL sel_registerName(const char *str)
+ 用block作为方法实现
    + IMP imp_implementationWithBlock(id block) 
    + id imp_getBlock(IMP anImp)
    + BOOL imp_removeBlock(IMP anImp)

# 常见数据结构
## Class的结构体
```
struct objc_class {
    Class isa;          
    Class superclass;
    cache_t cache;                      // 方法缓存
    class_data_bits_t bits;             // 用于获取具体的类信息
}

// bits & FAST_DATA_MASK
struct class_rw_t {
    uint32_t flags;
    uint32_t version
    const class_ro_t *ro;
    method_list_t *methods;             //方法列表 
    property_list_t *properties;        //属性列表
    const protocol_list_t *protocols;   //协议列表
    Class firstSubclass;
    Class nextSiblingClass;
    char *demangledName;
}

struct class_ro_t {
    uint32_t flags;
    uint32_t instanceStart;
    uint32_t instanceSize;              // instance对象占用的内存空间
#ifdef __LP64__
    uint32_t reserved;
#endif
    const uint8_t *ivarLayout;
    const char * name;                  // 类名
    method_list_t * baseMethodList;
    protocol_list_t * baseProtocols;
    const ivar_list_t *ivars;           //成员变量
    const uint8_t *weakIvarLayout;
    property_list_t *baseProperties;
}
```

## isa详解
+ arm64之前，isa是一个指针，存储着Class、Meta-Class对象的内存地址
+ arm64之后，isa被优化成了一个共用体（union）结构，使用位域存储更多信息
```
union isa_t {
    Class cls;
    uintptr_t bits;
    struct {
        uintptr_t nonpointer        : 1;
        uintptr_t has_assoc         : 1;
        uintptr_t has_cxx_dtor      : 1;
        uintptr_t shiftcls          : 33;
        uintptr_t magic             : 6;
        uintptr_t weakly_referenced : 1;
        uintptr_t extra_rc          : 19;
    }
}
```

+ nonpointer 
    + 0，代表普通的指针，存储着Class、Meta-Class对象的内存地址
    + 1，代表优化过，使用位域存储更多的信息
+ has_assoc
    + 是否有设置过关联对象，如果没有，释放时会更快

+ has_cxx_dtor
    + 是否有C++析构函数（.cxx_destruct），如果没有，是否会更快

+ shiftcls
    + 存储着Class、Meta-Class对象的内存地址信息

+ magic
    + 用于在调试时分辨对象是否未完成初始化

+ weakly_referenced
    + 是否有被弱引用指向过，如果没有，释放时会更快

+ deallocting
    + 对象是否正在释放

+ extra_rc
    + 里面存储的值是引用计数器减一
    
+ has_sidetable_rc
    + 引用计数器是否过大，无法存储在isa中
    + 如果为1，那么引用计数会存储在一个叫SideTable的类的属性中
    
## class_rw_t
class_rw_t里面的methods、properties、protocols是二维数组，是可读可写的，包含了类的初始内容、分类的内容
```
struct class_rw_t {
    method_array_t methods;
    property_array_t properties;
    protocol_array_t protocols;
}
```

## class_ro_t
class_ro_t里面的baseMethodList、baseProtocols、ivars、baseProperties是一维数组，是只读的，包含了类的初始内容
```
struct class_ro_t {
    method_list_t *baseMethodList;
    protocol_list_t *baseProtocols;
    const ivar_list_t *ivars;
    property_list_t *baseProperties;
}
```


## 方法结构
method_t是对方法/函数的封装
```
struct method_t {
    SEL name;           // 函数名
    const char *types;  // 编码（返回值类型、参数类型）
    IMP imp;            // 指向函数的指针（函数地址）
}
```

## 方法缓存
Class内部结构中有个方法缓存（cache_t），用散列表（哈希表）来缓存曾经调用过的方法，可提高方法查找速度。
```
struct cache_t {
    struct bucket_t *_buckets;  // 散列表
    mask_t _mask;               // 散列表的长度-1
    mask_t _occupied;           // 已经缓存的方法数量
}

struct bucket_t {
    cache_key_t _key;           // SEL作为key
    IMP _imp;                   // 
}
```

## 使用runtime Associate方法关联的对象，在什么时候释放么？
1. 调用 -release ：引用计数变为零
对象正在被销毁，生命周期即将结束. 
不能再有新的 __weak 弱引用，否则将指向 nil.
调用 [self dealloc]

2. 父类调用 -dealloc 
继承关系中最直接继承的父类再调用 -dealloc 
如果是 MRC 代码 则会手动释放实例变量们（iVars）
继承关系中每一层的父类 都再调用 -dealloc

3. NSObject 调 -dealloc 
只做一件事：调用 Objective-C runtime 中object_dispose() 方法

4. 调用 object_dispose()
为 C++ 的实例变量们（iVars）调用 destructors
为 ARC 状态下的 实例变量们（iVars） 调用 -release 
解除所有使用 runtime Associate方法关联的对象 
解除所有 __weak 引用 
调用 free()