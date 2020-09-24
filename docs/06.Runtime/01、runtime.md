### 一个objc对象如何进行内存布局？（考虑有父类的情况）
+ 所有父类的成员变量和自己的成员变量都会放在该对象所对应的存储空间中
+ 每个对象内部都有一个isa指针，指向他的类对象，类对象中存放着本对象的：
	+ 对象方法列表（对象能够接受的消息列表，保存在它所对应的类对象中）
	+ 成员变量的列表
	+ 属性列表

Ivar布局结构如下：

![013.png](/Users/admin/Code/study/iOSInterview/res/013.png)

翻译为中文：

![014.png](/Users/admin/Code/study/iOSInterview/res/014.png)

### 一个objc对象的isa指针指向什么？有什么作用？
指向他的类对象，从而可以找到对象上的方法


![015.png](/Users/admin/Code/study/iOSInterview/res/015.png)

### runtime如何通过selector找到对应的IMP地址？（分别考虑类方法和实例方法）
每一个类对象中都有一个方法列表，方法列表中记录着方法名称、方法实现、参数类型，其实selector本质就是方法名称，通过这个方法名称就可以在方法列表中找到对应的方法实现。


### 什么是消息查找
+ objc_msgSend在“消息传递”中的作用。在“消息传递”过程中，objc_msgSend的动作比较清晰：
	1. 首先在Class中的缓存查找IM,如果查找到了就直接调用
	2. 如果没找到，则在Class的方法列表中查找,,如果查找到了就添加到缓存列表并调用IMP
	3. 如果没找到就继续去父类的Class中查找，也是先查缓存后查方法列表
	4. 如果找到了就缓存到自己的Class缓存中
	5. 如果一直查到根类依旧没有实现，则调用_objc_msgForward方法，进入消息转发流程

### _objc_msgForward函数是做什么的，直接调用他将会发生什么？
+ _objc_msgForward是一个函数指针（和IMP的类型一样），用于消息转发的。当向一个对象发送一条消息，但它没有实现的时候，_objc_msgForward会尝试做消息转发。

### 什么是消息转发
+ Method resolution：
	+ objc运行时会调用```+resolveInstanceMethod: ```或者```+resolveClassMethod: ```，让你有机会提供一个函数实现。如果你添加了函数实现，那么运行时系统会重新启动一次消息发送过程。否则，运行时会移动到下一步，消息转发（Message Forwarding）
+ Fast forwarding
	+ 如果目标对象实现了```-forwardingTargetForSelector:```,Runtime这时就会调用这个方法，给你把这个消息转发给其他对象的机会。只要这个方法返回的不是nil和self，整个消息发送的过程就会被重启，而消息发送的对象会变成你返回的那个对象。否则，就会继续下一步（Normal forwarding）
+ Normal forwarding
	+ 这一步时Runtime提供的最后一次机会。首先它会发送```-methodSignatureForSelector：```消息获得函数的参数和返回值类型。如果```-methodSignatureForSelector:```返回nil，Runtime则会发出```-doseNotRecognizeSelector：```消息，程序随后崩溃。如果返回一个函数签名，runtime就会将methodSignatureForSelector中取到的方法签名包装成一个NSInvocation对象并发送```-forwardInvocation:```消息给目标对象

上面前4个方法都是模版方法，开发者可以override，由runtime来调用。最常见的实现消息转发：就是重写```methodSignatureForSelector：```和```forwardInvocation:```方法，其中主要涉及以下五个方法：
1. resolveInstanceMethod:方法（或resolveClassMethod:）
2. forwardingTargetForSelector:方法
3. methodSignatureForSelector:方法
4. forwardInvocation:方法
5. doesNotRecognizeSelector:方法

### runtime 如何实现weak变量的自动置nil？
+ runtime对注册的类，会进行布局，对于weak对象会放入一个hash表中。用weak指向的对象内存地址作为key，当此对象的引用计数为0的时候会dealloc，假如weak指向的对象内存地址是a，那么就会以a为键，在这个weak表中搜索，找到所有以a为键的weak对象，从而设置为nil。

### weak 实现原理
+ Runtime维护了一个weak表，用于存储指向某个对象的所有weak指针。weak表其实是一个hash（哈希）表，Key是所指对象的地址，Value是weak指针的地址（这个地址的值是所指对象指针的地址）数组。

### weak 实现步骤
+ 1、初始化时：runtime会调用objc_initWeak函数，初始化一个新的weak指针指向对象的地址。
+ 2、添加引用时：objc_initWeak函数会调用 objc_storeWeak() 函数， objc_storeWeak() 的作用是更新指针指向，创建对应的弱引用表。
+ 3、释放时，调用clearDeallocating函数。clearDeallocating函数首先根据对象地址获取所有weak指针地址的数组，然后遍历这个数组把其中的数据设为nil，最后把这个entry从weak表中删除，最后清理对象的记录。
+ https://www.jianshu.com/p/13c4fb1cedea


