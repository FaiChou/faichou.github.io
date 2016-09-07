---
layout: post
title: "runtime in Objective-C"
date: 2016-09-07
---

### 综合网络上对runtime的评价

> * Objective-C是C语言的扩展，并加入了面向对象特性和Smalltalk式的消息传递机制, 这个扩展的核心就是一个用C和汇编语言写的RunTime库

> * 这个库所做的事情就是加载类信息，进行方法的分发和转发，正是这个库赋予了Objective-C的动态特性

> * Objective-C 是一种面向runtime(运行时)的语言，也就是说，它会尽可能地把代码执行的决策从编译和链接的时候，推迟到运行时


### 代码分析

`id returnValue = [someObject messageName:parameter]`

- someObject叫做接受者receiver
- messageName叫做选择子selector
- 俩合起来称作消息message

在`<objc/message.h>`中有方法定义 `id objc_msgSend(id self, SEL op, ...)`, 官方注释如下:

```
/** 
 * Sends a message with a simple return value to an instance of a class.
 * 
 * @param self A pointer to the instance of the class that is to receive the message. // self 是一个指向接受消息的实例的指针
 * @param op The selector of the method that handles the message. // op 是处理消息的方法选择器
 * @param ... 
 *   A variable argument list containing the arguments to the method. // 方法的参数们
 * 
 * @return The return value of the method. // objc_msgSend的返回值和方法的返回值是一样的
 * 
 * @note When it encounters a method call, the compiler generates a call to one of the
 *  functions \c objc_msgSend, \c objc_msgSend_stret, \c objc_msgSendSuper, or \c objc_msgSendSuper_stret.
 *  Messages sent to an object’s superclass (using the \c super keyword) are sent using \c objc_msgSendSuper; 
 *  other messages are sent using \c objc_msgSend. Methods that have data structures as return values
 *  are sent using \c objc_msgSendSuper_stret and \c objc_msgSend_stret.
 */
```

经过编译链接，会转化成以下代码:

```
id returnValue = objc_msgSend(someObject,
							  @selector(messageName:),
							  parameter);
```

### 由浅入深，分析源码

在`<objc/objc.h>`中可以找到:

```
/// An opaque type that represents an Objective-C class.
typedef struct objc_class *Class;

/// Represents an instance of a class.
struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};

/// A pointer to an instance of a class.
typedef struct objc_object *id;
```
可以发现，对象是一个C的结构体，类和方法也是结构体。对象的结构体里有**`Class`**类型的`isa`, *`typedef struct objc_class *Class;`*给出了它的定义，是一个指针，指向`objc_class`，所以，类的实例的`isa`指针指向本类。

在`<objc/runtime.h>`中可以找到:

```
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE; // 父类
    const char *name                                         OBJC2_UNAVAILABLE; // 类名
    long version                                             OBJC2_UNAVAILABLE; // 类的版本信息，默认为0
    long info                                                OBJC2_UNAVAILABLE; // 类信息，供运行时期使用的一些位标识
    long instance_size                                       OBJC2_UNAVAILABLE; // 该类的实例变量大小
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE; // 该类的成员变量链表
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE; // 方法定义链表
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE; // 方法缓存
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE; // 协议链表
#endif

} OBJC2_UNAVAILABLE;
/* Use `Class` instead of `struct objc_class *` */


struct objc_method {
    SEL method_name                                          OBJC2_UNAVAILABLE; // 方法名
    char *method_types                                       OBJC2_UNAVAILABLE; // 函数类型的字符串
    IMP method_imp                                           OBJC2_UNAVAILABLE; // 函数的实现IMP
}                                                            OBJC2_UNAVAILABLE;

struct objc_method_list {
    struct objc_method_list *obsolete                        OBJC2_UNAVAILABLE;

    int method_count                                         OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
    /* variable length structure */
    struct objc_method method_list[1]                        OBJC2_UNAVAILABLE;
}                                                            OBJC2_UNAVAILABLE;

```

#### `objc_msgSend`执行顺序

1. 通过`someObject`的`isa`指针找到它的`Class`, `someClass`
2. 在`someClass`的`objc_method_list`中找到方法`messageName:`
3. 如果找不到，继续往它的`superClass`中找
4. 一旦找到`messageName:`这个函数，就去执行它的实现IMP

如果所有方法都是按照这个顺序，那么Objective-C肯定是死了，因为一个`class`往往只有 20% 的函数会被经常调用，可能占总调用次数的 80% 。每个消息都需要遍历一次`objc_method_list`并不合理。为了提高效率，苹果为`objc_class`添加了另一个重要成员`objc_cache` ,cache要做的事就是在找到`messageName:`之后，把`messageName:`的`method_name`作为key ，`method_imp`作为 value 给存起来。当再次收到`messageName:`消息的时候，可以直接在cache里找到，避免去遍历`objc_method_list`。

> Objective-C的方法调用因为运行期才动态解析消息，一开始消息比C++ virtual成员函数调用速度慢上三倍。但经由IMP缓存改善，目前已经比C++的virtual function快上50％

### 动态方法解析和转发

在上面的例子中，如果`messageName:`没有找到会发生什么？通常情况下，程序会在运行时挂掉并抛出 *`unrecognized selector sent to … `*的异常。但在异常抛出前，Objective-C 的运行时会给你三次拯救程序的机会：

1. method resolution
2. first forwarding
3. last forwarding

#### method resolution

首先，Objective-C 运行时会调用 `+resolveInstanceMethod: `或者 `+resolveClassMethod:`，让你有机会提供一个函数实现。

如果你添加了函数并返回 **`YES`**， 那运行时系统就会重新启动一次消息发送的过程。以`messageName:`为例，可以这么实现：

```
void foo(id obj, SEL _cmd)  
{
    NSLog(@"Call a method");
}

+ (BOOL)resolveInstanceMethod:(SEL)aSEL
{
    if(aSEL == @selector(messageName:)){
        class_addMethod([self class], aSEL, (IMP)foo, "v@:");
        return YES;
    }
    return [super resolveInstanceMethod];
}
```

如果 `resolve` 方法返回 `NO` ，运行时就会移到下一步：**消息转发**（message forwarding）.

#### first forwarding

如果目标对象实现了 `-forwardingTargetForSelector:` ，Runtime 这时就会调用这个方法，给你把这个消息转发给其他对象的机会。

```
- (id)forwardingTargetForSelector:(SEL)aSelector
{
    if(aSelector == @selector(messageName:)){
        return alternateObject;
    }
    return [super forwardingTargetForSelector:aSelector];
}
```

只要这个方法返回的不是 `nil` 和 `self`，整个消息发送的过程就会被重启，当然发送的对象会变成你返回的那个对象。否则，就会继续 last fowarding.

#### last forwarding

这一步是 Runtime 最后一次给你挽救的机会。首先它会发送 `-methodSignatureForSelector:` 消息获得函数的参数和返回值类型。

如果 `-methodSignatureForSelector: `返回 `nil` ，Runtime 则会发出 `-doesNotRecognizeSelector:` 消息，程序这时也就挂掉了。

如果返回了一个函数签名，Runtime 就会创建一个 `NSInvocation` 对象并发送 `-forwardInvocation:` 消息给目标对象。


NSInvocation 实际上就是对一个消息的描述，包括selector 以及参数等信息。
所以你可以在 `-forwardInvocation:` 里修改传进来的 NSInvocation 对象，然后发送 `-invokeWithTarget:` 消息给它，传进去一个新的目标：

```
- (void)forwardInvocation:(NSInvocation *)invocation
{
    SEL sel = invocation.selector;

    if([alternateObject respondsToSelector:sel]) {
        [invocation invokeWithTarget:alternateObject];
    } 
    else {
        [self doesNotRecognizeSelector:sel];
    }
}
```

#### 消息转发的全部流程，一图以弊之

![message_handle](http://o7bkcj7d7.bkt.clouddn.com/message_handle.png)


### 总结

Objective-C给对象发送消息的过程如下：

1. 在对象类的 dispatch table 中寻找该方法。如果找到了，跳到相应的函数IMP去执行实现代码
2. 如果没有找到，Runtime 会发送 `+resolveInstanceMethod:` 或者 `+resolveClassMethod:` 尝试去 resolve 这个消息
3. 如果 resolve 方法返回 `NO`，Runtime 就发送 `-forwardingTargetForSelector:` 允许你把这个消息转发给另一个对象
4. 如果没有新的目标对象返回， Runtime 就会发送 `-methodSignatureForSelector:` 和 `-forwardInvocation:` 消息


### 参考

- [iOS Runtime Programming](https://felixha.wordpress.com/2015/01/22/runtime-programming/)

- [理解 Objective-C Runtime](http://justinyan.me/post/1624)

- [Associated Objects](http://nshipster.cn/associated-objects/)

- [Objective-C对象模型及应用](http://blog.devtang.com/2013/10/15/objective-c-object-model/)

- [浅谈OC运行时(RunTime)](http://honglu.me/2014/12/29/%E6%B5%85%E8%B0%88OC%E8%BF%90%E8%A1%8C%E6%97%B6-RunTime/)

- [Objective-C Runtime](http://tech.glowing.com/cn/objective-c-runtime/)

- [Effective Objective-C Notes：理解消息传递机制](https://www.zybuluo.com/MicroCai/note/64270)

- [Runtime全方位装逼指南](http://www.jianshu.com/p/efeb33712445)











