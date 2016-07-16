---
layout: post
title: "ProgrammingWithObjC阅读笔记"
date: 2016-07-16
---

> 发表时间：2016-07-16 10:00   
> 文章分类：读书(`ProgrammingWithObjectiveC`)笔记   
> 原文地址：http://faichou.space/notes/2016/07/16/ProgrammingWithObjC%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0

## 目录
1. Objective-C介绍
2. 类的使用
3. 对象的使用
4. 属性相关
5. 类的扩充

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致。
转载本博客文章必须也遵循[署名-非商业用途-保持一致](https://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议。

### [#1.]() Objective-C介绍

Objective-C是一种通用、高级、面向对象的编程语言。它扩展了标准的ANSI C编程语言，将Smalltalk式的消息传递机制加入到ANSI C中。目前主要支持的编译器有GCC和Clang（采用LLVM作为前端）。

Objective-C的商标权属于苹果公司，苹果公司也是这个编程语言的主要开发者。苹果在开发NeXTSTEP操作系统时使用了Objective-C，之后被OS X和iOS继承下来。现在Objective-C与Swift是OS X和iOS操作系统、及与其相关的API、Cocoa和Cocoa Touch的主要编程语言。

> Xcode is the IDE(`Integrated Development Environment`) used to build apps for iOS and OS X; you’ll use it to write your code, design your app's user interface, test your application, and debug any problems. 

我们编写iOS或macOS系统应用程序的时候，苹果给我们制定了良好的集成开发环境Xcode。


> Objective-C apps use reference counting to determine the lifetime of objects. 

在OC中对象的生存周期是由引用计数决定的。

> In addition to the compiler, the Objective-C language uses a runtime system to enable its dynamic and object-oriented features. 

程序的编译和runtime机制使得OC成为了一个良好的动态和面向对象的编程语言。

Objective-C是C语言的严格超集－－任何C语言程序不经修改就可以直接通过Objective-C编译器，在Objective-C中使用C语言代码也是完全合法的。

#### Hello World

```
#import <Foundation/Foundation.h>

int main(int argc, char *argv[]) {

    @autoreleasepool {
        NSLog(@"Hello World!");
    }

   return 0;
} // based on Xcode Version 7.3.1
```

### [#2.]() 类的使用

> When you write software for OS X or iOS, most of your time is spent working with objects. Objects in Objective-C are just like objects in other object-oriented programming languages: they package data with related behavior. 

当你编写iOS程序时候，大多数时间是在和对象打交道。Objective-C是和其他面向对象编程语言一样，它的对象是对相关行为数据的包装。

> An app is built as a large ecosystem of interconnected objects that communicate with each other to solve specific problems, such as displaying a visual interface, responding to user input, or storing information. For OS X or iOS development, you don’t need to create objects from scratch to solve every conceivable problem; instead you have a large library of existing objects available for your use, provided by Cocoa (for OS X) and Cocoa Touch (for iOS). 

可以将应用程序看做大的生态系统，里面有各种各样相关的对象。对象之间相互交流，一起完成某项任务。比如展示一个页面，获得用户的输入，存储用户输入的数据；这里就需要对象之间相互关联传递信息。开发iOS程序，你不需要担心这很繁琐，因为大多数的框架已经给你准备好了。

> Classes Are Blueprints for Objects

类是对象的蓝图。

> Mutability Determines Whether a Represented Value Can Be Changed

```
NSNumber *numA = @1;
numA           = @2;
NSLog(@"numA = %@", numA);

---console
2016-07-16 11:05:08.750 FaiChouTest[1041:51277] numA = 2

```
这里的`numA`是不可变的，而代码中`numA`由1变为2，并不是不可变特性，而是字面量（`literal syntax`）重新初始化。

> Classes Inherit from Other Classes

继承能够使子类保持父类的基本属性和行为，子类还可以重写父类的方法。

> One of the many benefits of object-oriented programming is the idea mentioned earlier—all you need to know in order to use a class is how to interact with its instances. More specifically, an object should be designed to hide the details of its internal implementation. 

OOP的其中一个优点是我们只需要知道类的实例如何交互，并不需要关心如何实现的，并且，一个对象应该将实现的细节隐藏深处。

> Objective-C provides a preprocessor directive, #import, for this purpose. It’s similar to the C #include directive, but makes sure that a file is only included once during compilation. 

`#import`保证编译期文件只被`include`一次。


### [#3.]() 对象的使用

> Objects Send and Receive Messages

对象是收发消息的主体，被称作**消息传递**。Objective-C里，与其说对象互相调用方法，不如说对象之间互相传递消息更为精确。在Objective-C，类别与消息的关系比较松散，调用方法视为对对象发送消息，所有方法都被视为对消息的回应。所有消息处理直到运行时（runtime）才会动态决定，并交由类别自行决定如何处理收到的消息。也就是说，一个类别不保证一定会回应收到的消息，如果类别收到了一个无法处理的消息，程序只会抛出异常，不会出错或崩溃。

```
[somePerson sayHello];
```

典型的C++意义解读是“调用somePerson类别的sayHello方法”。若somePerson类别里头没有定义sayHello方法，那编译肯定不会通过。但是Objective-C里，我们应当解读为“发提交一个sayHello的消息给somePerson对象”，sayHello是消息，而somePerson是消息的接收者。somePerson收到消息后会决定如何回应这个消息，若somePerson类别内定义有sayHello方法就运行方法内之代码，若somePerson内不存在sayHello方法，则程序依旧可以通过编译，运行期则抛出异常。*`勘误·1`*

此二种风格各有优劣。C++强制要求所有的方法都必须有对应的动作，且编译期绑定使得函数调用非常快速。缺点是仅能借由virtual关键字提供有限的动态绑定能力。Objective-C天生即具备鸭子类型之动态绑定能力，因为运行期才处理消息，允许发送未知消息给对象。可以送消息给整个对象集合而不需要一一检查每个对象的型态，也具备消息转送机制。同时空对象nil接受消息后默认为不做事，所以送消息给nil也不用担心程序崩溃。

Objective-C的方法调用因为运行期才动态解析消息，一开始消息比C++ virtual成员函数调用速度慢上三倍。但经由IMP缓存改善，目前已经比C++的virtual function快上50％。

Objective-C创建对象需通过alloc以及init两个消息。alloc的作用是分配内存，init则是初始化对象。

```
id someObject = @"Hello, World!";
[someObject removeAllObjects];
```

这两行代码能够通过编译，运行出错。能够看到`id`的坏处，用`id`声明的变量，编译器不会有这个变量的方法，所以在编译期就不会及时给出错误的提醒。


### [#4.]() 属性相关

```
@property (readonly) NSString *fullName;
@property (getter=isFinished) BOOL finished;
```

`@Property`是声明属性的语法，它可以快速方便的为实例变量创建存取器，并允许我们通过点语法使用存取器。

> 存取器（accessor）：指用于获取和设置实例变量的方法。用于获取实例变量值的存取器是`getter`，用于设置实例变量值的存取器是`setter`。

-

> Most Properties Are Backed by Instance Variables 

实例变量支持大多数属性。

> You Can Customize Synthesized Instance Variable Names 

我们可以自己综合实例变量的名字

```
@implementation YourClass
@synthesize propertyName = instanceVariableName; // 自定义
@synthesize firstName    = ivar_firstName; // 
@synthesize lastName; // 不分配名称就会使用本身，下划线也省了（without an underscore）
@end
```

> You Can Implement Custom Accessor Methods

自定义存取器方法

> If you need to write a custom accessor method for a property that does use an instance variable, you must access that instance variable directly from within the method. For example, it’s common to delay the initialization of a property until it’s first requested, using a “lazy accessor,” like this: 

```
- (XYZObject *)someImportantObject {
    if (!_someImportantObject) {
        _someImportantObject = [[XYZObject alloc] init];
    }
    return _someImportantObject;
} // lazy accessor
```

> Properties Are Atomic by Default 

原子性能够使不同的线程同时对一个属性存取。

>  it’s faster to access a nonatomic property than an atomic one, and it’s fine to combine a synthesized setter

这就是为什么属性一般都要加`nonatomic`，因为它相比原子性`atomic`，能够更快获得属性，更好的编译综合一个`setter`。    `atomic`是线程安全的，至少在当前的存取器上是安全的。假如a线程更改 XYZPerson 的名和姓，b线程同时获取 XYPPerson 的名和姓，原子性`atomic`就容易导致b线程获取的姓名不匹配。当涉及到网络数据方面，这类问题瓶颈就会凸显了。

>  In Objective-C, an object is kept alive as long as it has at least one strong reference to it from another object

强引用会保持一个对象不被销毁。

>  two remaining strong relationships keep the two objects alive. This is known as a strong reference cycle.

我们编程时需要用`weak reference`避免循环强引用。

> Use Unsafe Unretained References for Some Classes   
> There are a few classes in Cocoa and Cocoa Touch that don’t yet support weak references, which means you can’t declare a weak property or weak local variable to keep track of them. These classes include NSTextView, NSFont and NSColorSpace; for the full list, see Transitioning to ARC Release Notes 

```
{
    @property (unsafe_unretained) NSObject *unsafeProperty; // property
}

{
    NSObject * __unsafe_unretained unsafeReference; // variable
}

```

> Copy Properties Maintain Their Own Copies 

```
NSMutableString *nameString = [NSMutableString stringWithString:@"John"]; 
self.badgeView.firstName = nameString; // without copy
```
> Although the badge view thinks it’s dealing with an NSString instance, it’s actually dealing with an NSMutableString. 

这里的属性`firstName`(`NSString`)没有加`copy`，当一个`NSMutableString`赋值给它时候，它自身属性就会发生改变；而当我们加上`copy`时候，属性就不会被外界影响，如果赋值一个`NSMutableString`变量，只会深复制(`deep copy`)其内容，不会将自身指针指向其它。


### [#5.]() 类的扩充

#### 类别 (Category)

> Categories Add Methods to Existing Classes

在Objective-C的设计中，一个主要的考虑即为大型代码框架的维护。结构化编程的经验显示，改进代码的一种主要方法即为将其分解为更小的片段。Objective-C借用并扩展了Smalltalk实现中的“分类”概念，用以帮助达到分解代码的目的。

一个分类可以将方法的实现分解进一系列分离的文件。程序员可以将一组相关的方法放进一个分类，使程序更具可读性。

若分类声明了与类中原有方法同名的函数，则分类中的方法会被调用。因此分类不仅可以增加类的方法，也可以代替原有的方法。这个特性可以用于修正原有代码中的错误，更可以从根本上改变程序中原有类的行为。若两个分类中的方法同名，则被调用的方法是不可预测的。

#### 延展(Extension)

> Class Extensions Extend the Internal Implementation

> Use Class Extensions to Hide Private Information

延展可以声明属性，添加方法。而且在`Implementation`区块中添加的属性方法的权限是private，这非常匹配OOP的封装原则，私有信息就不需要暴露于公开interface中。

#### 协议（Protocol）

协议是一组没有实现的方法列表，任何的类均可采纳协议并具体实现这组方法。

Objective-C在NeXT时期曾经试图引入多重继承的概念，但由于协议的出现而没有实现之。

协议经常应用于Cocoa中的委托及事件触发。

> To make the view as reusable as possible, all decisions about the information should be left to another object, a data source. This means that multiple instances of the same view class could display different information just by communicating with different sources. 

让 view 尽可能的重用，因为一个 view 可能有许多实例，而这些实例跟不同的数据源交互联系时候会展示不同的内容，所以我们选择了协议！

> If you attempt to call the respondsToSelector: method on an id conforming to the protocol as it’s defined above, you’ll get a compiler error that there’s no known instance method for it. Once you qualify an id with a protocol, all static type-checking comes back; you’ll get an error if you try to call any method that isn’t defined in the specified protocol. One way to avoid the compiler error is to set the custom protocol to adopt the NSObject protocol. 

我们为什么要继承 NSObject 协议呢？
因为当我们写一个 @optional 协议方法时候，就需要判断一个对象是否接受了这个协议并且写了这个方法，例如` if ([self.dataSource respondsToSelector:@selector(titleForSegmentAtIndex:)])`，而这个`respondsToSelector:`是定义在 NSObject 协议中的方法，所以我们大多协议都需要继承 NSObject 协议。




#### 运行时（runtime）

因为Objc是一门动态语言，所以它总是想办法把一些决定工作从编译连接推迟到运行时。也就是说只有编译器是不够的，还需要一个运行时系统 (runtime system) 来执行编译后的代码。这就是 Objective-C Runtime 系统存在的意义，它是整个Objc运行框架的一块基石。

RunTime简称运行时。OC就是运行时机制，其中最主要的是消息机制。对于C语言，函数的调用在编译的时候会决定调用哪个函数。对于OC的函数，属于动态调用过程，在编译的时候并不能决定真正调用哪个函数，只有在真正运行的时候才会根据函数的名称找到对应的函数来调用。

Runtime基本是用C和汇编写的，可见苹果为了动态系统的高效而作出的努力。你可以在这里下到苹果维护的开源代码。苹果和GNU各自维护一个开源的runtime版本，这两个版本之间都在努力的保持一致。
#### 转发(Forwarding)

Objective-C允许对一个对象发送消息，不管它是否能够响应之。除了响应或丢弃消息以外，对象也可以将消息转发到可以响应该消息的对象。转发可以用于简化特定的设计模式，例如观测器模式或代理模式。





### 参考链接
---

- [Objective-C](https://zh.wikipedia.org/wiki/Objective-C#.E8.BD.AC.E5.8F.91)
- [Objective-C中的@property](http://www.devtalking.com/articles/you-should-to-know-property/)











