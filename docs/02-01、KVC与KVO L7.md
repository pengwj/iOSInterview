[TOC]

# KVO （240-249）
## KVO的实现原理
使用KVO监听A对象属性时，runtime会动态的创建该对象的子类（NSKVONotifying_A），然后重写setter方法，在setter方法中调用```_NSSetXXXValueAndNotify```，最终调用willchangeValue、didchangeValue方法。

+ 利用runtime动态生成一个子类，并且让实例对象的isa指向这个全新的子类
+ 当修改实例对象的属性时，会调用Foundation的_NSSetXXXValueAndValueNotify函数 （_NSSetIntValueAndNotify）
+  _NSSetXXXValueAndValueNotify函数的内部实现如下：
    + willChangeValueForKey:
    + 父类原来的setter
    + didChangeValueForKey： 该方法内部会触发监听器（Oberser）的监听方法（observeValueForKeyPath：ofObject：change:context:）

## 如何手动触发通知
手动调用willChangeValueForKey：和didChangeValueForKey：

## 直接修改成员变量（_key方式）会触发KVO吗？
不会触发KVO

# KVC （250-252）
## 通过KVC修改属性会触发KVO么？
会触发KVO，就算没有实现setter方法（开发者、系统均没有实现，只有成员变量时），也会触发（推测KVC内部有主动调用didChangeValueForKey方法）

## KVC的赋值和取值过程是怎样的？原理是什么？
+ 赋值过程
    +  按照setKey:、_setKey:顺序查找方法
    +  如果没找到方法，并且accessInstanceVariablesDirectly返回YES，就开始查询成员变量
    +  按照_key、_isKey、key、isKey的顺序查找成员变量

+ 取值过程
    + 按照getKey、key、isKey、_key的顺序查找
    + 如果accessInstanceVariablesDirectly返回YES，就直接查询成员变量，否则直接崩溃
    + 按照_key、_isKey、key、isKey的顺序查找成员变量