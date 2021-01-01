[TOC]

# block
## block的原理是怎样的？本质是什么？
封装了函数调用以及调用环境的OC对象
+ block本质上也是一个OC对象，内部含有isa指针
+ block是一个封装了函数调用以及函数调用环境的OC对象
+ block的底层结构是一个结构体

```
isa
flags
reserved
invoke
descriptor
variables
```

```
struct __main_block_impl_0 {
    struct __block_impl impl;
    struct __main_block_desc_0 *Desc;
}

struct __block_impl {
    void *isa;
    int Flags;
    int Reserved;
    void *FuncPtr;
}

struct __main_block_desc_0 {
    size_t reserved;
    size_t Block_size;
}

```

## Block的写法
```
typedef BOOL (^myBlock)(NSInteger, CGFloat);

@property (nonatomic, copy) myBlock block;

self.block = ^(NSInteger a, CGFloat b) {
        
    if ((CGFloat)a > b) {
        return YES;
    } else {
        return NO;
    }
};
```

## block的类型有哪些？
可以通过class方法查看具体类型，最终都是继承自NSBlock->NSObject
+ __NSGlobalBlock__ (_NSConcreteGlobalBlock)  存放在数据区（全局）
    + 没有访问auto变量（可以访问全局变量、静态变量，只要没访问auto变量就好）
+ __NSStackBlock__  (_NSConcreteStackBlock)  存放在栈区
    + 访问了auto变量（自动局部变量） 
+ __NSMallocBlock__ (_NSConcreteMallocBlock)  存放在堆区
    +  __NSStackBlock__调用了copy函数

__NSGlobalBlock__调用copy函数后，还是__NSGlobalBlock__类型

## block的继承关系？
__ NSGlobalBlock__ -> __ NSGlobalBlock -> NSBlock -> NSObject

## block在arc环境下什么情况会自动copy？
在arc环境下，编译器会根据情况自动将栈上的block复制到堆上，比如以下情况：
+ block作为函数返回值时
+ 将block赋值给__strong指针时
+ block作为Cocoa API中方法名含有usingBlock的方法参数时
+ block作为GCD API的方法参数时

## block的属性修饰词为什么是copy？使用block有哪些使用注意？
+ block一旦没有进行copy操作，就不会在堆上
+ 使用注意：循环引用问题

## block调用copy时实质做了什么？
+ 将block复制到了堆上
+ 将block中持有的auto变量也复制到了堆上，如果是对象就捕获其指针
+ block->person(Block_byref)->forwarding->person对象

## __block的作用是什么？有什么使用注意点？
__block的本质是，编译器会将__block变量包装成一个对象，然后将指针捕获到block结构体中，这样就可以在block内部持有/修改外部的auto变量。
+ __block可以用于解决block内部无法修饰auto变量值的问题
+ __block不能用来修饰全局变量、静态变量

## __block的__forwarding指针
+ 这样设计的主要原因是，当auto变量被多个block持有时，栈上的block和堆上的block可以访问同一个内存区域。（当block非全局，且被__weak修饰时为stackblock）

## block在修改NSMutableArray，需不需要添加__block？
不需要，往NSMutableArray中添加数据时没有修改该变量的值

## block会导致循环引用,解决方案是什么？
+ arc情况下
    + 用__weak、__unsafe_unretained解决
    + 用__block解决（我觉得这个解决方式很蹩脚，面试时不要说）
+ mrc情况下
    + 用__unsafe_unretained、__block解决(mrc情况下没有__weak、__block主要是借助runtime的一个特性)