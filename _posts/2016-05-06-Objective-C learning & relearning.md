---
layout: post
title: "Objective-C learning & relearning"
date: 2016-05-06
---

### 代码风格

- 指针'*'号的位置

　　　　`NSString *varName`

 - 方法的声明和定义
- 在 - OR + 和返回值之间留1个空格,方法名和第一个参数间不留空格。
-  当参数过长时,每个参数占用一行,以冒号对齐。
-  如果方法名比参数名短,每个参数占用一行,至少缩进4个字符,且为垂直对齐(而非使用冒号 对齐)。
-  在Objective-C中,只允许使用BOOL

```
- (void)doSomethingWithString:(NSString *)theString {
    
 
...}

- (void)doSomethingWith:(GTMFoo *)theFoo
                   rect:(NSRect)theRect
               interval:(float)theInterval {


...}

- (void)short:(GTMFoo *)theFoo
    longKeyword:(NSRect)theRect
    evenLongerKeyword:(float)theInterval {
         

...}


- (BOOL)isBold {
     return [self fontTraits] & NSFontBoldTrait;
} // 禁止

- (BOOL)isValid {
     return [self stringValue];
} // 禁止

- (BOOL)isBold {
     return ([self fontTraits] & NSFontBoldTrait) ? YES : NO;
} // 对头 

- (BOOL)isValid {
    return [self stringValue] != nil;
} // 对头

- (BOOL)isEnabled {
    return [self isValid] && [self isBold];
} // 对头 



```

-  避免显式的调用`+new`方法，使用`alloc`和`init`方法。
-  如果类声明中包含多个protocal，每个protocal占用一行，缩进2个字符。

```
@interface RootViewController : UITableViewController<
  UITableViewDelegate,
  UITableViewDataSource,
  UITextFieldDelegate,
  UITextViewDelegate >

```

<br />

<br />







### 结构体、枚举、宏

- 结构体

```
struct Books
{
   NSString *title;
   NSString *author;
   NSString *subject;
   int   book_id;
} book;

//此声明声明了拥有3个成员的结构体，分别为整型的a，字符型的b和双精度的c
//同时又声明了结构体变量s1
//这个结构体并没有标明其标签
struct 
{
    int a;
    char b;
    double c;
} s1;

//此声明声明了拥有3个成员的结构体，分别为整型的a，字符型的b和双精度的c
//结构体的标签被命名为SIMPLE,没有声明变量
struct SIMPLE
{
    int a;
    char b;
    double c;
};
//用SIMPLE标签的结构体，另外声明了变量t1、t2、t3
struct SIMPLE t1, t2[20], *t3;

//也可以用typedef创建新类型
typedef struct
{
    int a;
    char b;
    double c; 
} Simple2;
//现在可以用Simple2作为类型声明新的结构体变量
Simple2 u1, u2[20], *u3;


```

- 枚举

```
typedef enum  
{  
    //以下是枚举成员  
    TestA = 0,  
    TestB,  
    TestC,  
    TestD  
}Test;//枚举名称  

typedef NS_ENUM(NSInteger, Test1)  
{  
    //以下是枚举成员  
    Test1A = 0,  
    Test1B = 1,  
    Test1C = 2,  
    Test1D = 3  
}; // suggest

typedef NS_ENUM(NSInteger, Test)  
{  
    TestA       = 1,      //1   1   1  
    TestB       = 1 << 1, //2   2   10      转换成 10进制  2  
    TestC       = 1 << 2, //4   3   100     转换成 10进制  4  
    TestD       = 1 << 3, //8   4   1000    转换成 10进制  8  
    TestE       = 1 << 4  //16  5   10000   转换成 10进制  16  
}; // 位运算
/*
什么时候要用到这种方式呢?
那就是一个枚举变量可能要代表多个枚举值的时候. 其实给一个枚举变量赋予多个枚举值的时候,原理只是把各个枚举值加起来罢了.
当加起来以后,就获取了一个新的值,那么为了保证这个值的唯一性,这个时候就体现了位运算的重要作用.
位运算可以确保枚举值组合的唯一性.
因为位运算的计算方式是将二进制转换成十进制,也就是说,枚举值里面存取的是 计算后的十进制值.
打个比方:
通过上面的位运算方式设定好枚举以后,打印出来的枚举值分别是: 1 2 4 8 16
这5个数字,无论你如何组合在一起,也不会产生两个同样的数字.
手工的去创建位运算枚举,还有稍微有点花时间的,好在Apple已经为我们准备的uint.所以,用下面这种方式来初始化一个位运算枚举吧:
*/
typedef NS_ENUM(uint, Test)  
{  
    TestA,  
    TestB,  
    TestC,  
    TestD,  
    TestE    
}; 

/*
在iOS6和Mac OS 10.8以后Apple引入了两个宏来重新定义这两个枚举类型，实际上是将enum定义和typedef合二为一，并且采用不同的宏来从代码角度来区分。
NS_OPTIONS一般用来定义位移相关操作的枚举值，我们可以参考UIKit.Framework的头文件，可以看到大量的枚举定义。
*/
typedef NS_ENUM(NSInteger, UIViewAnimationTransition) {  
    UIViewAnimationTransitionNone,//默认从0开始  
    UIViewAnimationTransitionFlipFromLeft,  
    UIViewAnimationTransitionFlipFromRight,  
    UIViewAnimationTransitionCurlUp,  
    UIViewAnimationTransitionCurlDown,  
};  
  
typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {  
    UIViewAutoresizingNone                 = 0,  
    UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,  
    UIViewAutoresizingFlexibleWidth        = 1 << 1,  
    UIViewAutoresizingFlexibleRightMargin  = 1 << 2,  
    UIViewAutoresizingFlexibleTopMargin    = 1 << 3,  
    UIViewAutoresizingFlexibleHeight       = 1 << 4,  
    UIViewAutoresizingFlexibleBottomMargin = 1 << 5  
};  
// NS_ENUM是通用情况，NS_OPTIONS一般用来定义具有位移操作或特点的情况（bitmask)。

```

- 宏


　　创建一个import所有宏相关的文件Macros.h  

　　在xcode项目的pch文件中，导入Macros.h文件

```
Macros.h
#import "UtilsMacros.h"
#import "APIStringMacros.h"
#import "DimensMacros.h"
#import "NotificationMacros.h"
#import "SharePlatformMacros.h"
#import "StringMacros.h"
#import "UserBehaviorMacros.h"
#import "PathMacros.h"
```

```
XcodeProjectName-Prefix.pch
#ifdef __OBJC__
    #import <UIKit/UIKit.h>
    #import <Foundation/Foundation.h>
    #import "Macros.h"
#endif
```

1. 定义尺寸类的宏
2. 定义沙盒目录文件的宏
3. 工具类的宏
4. 通知Notification相关的宏
5. 服务端API接口的宏
6. ...


<br />

<br />


### NSString, Array, Dictionary

- NSString

　　　[官方文档](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/ObjC_classic/index.html#//apple_ref/doc/uid/20001091)
　　　

　　　[iOS-Blog](http://www.ios-blog.co.uk/tutorials/objective-c-strings-a-guide-for-beginners/)


　　　[NSString常用基础方法](http://www.jianshu.com/p/17110e181d84)

<br />

- NSArray

　　　[官方文档](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/ObjC_classic/index.html#//apple_ref/doc/uid/20001091)

　　　[NSArray类简介](http://www.jianshu.com/p/bd697a6b8720)

　　　[NSArray](http://www.jianshu.com/p/74cadfd06097)

　　　[NSArray和NSMutableArray](http://www.tuicool.com/articles/nIrEJv)

　　　[NSMutableArray](http://www.jianshu.com/p/b96a20d6e38c)


- NSDictionary

　　　[官方文档](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/ObjC_classic/index.html#//apple_ref/doc/uid/20001091)

　　　[NSDictionary](http://www.jianshu.com/p/9b66d8f2669d)

　　　[NSMutableDictionary](http://www.jianshu.com/p/7129232c873c)

　　　[NSMutableDictionary的基本使用](http://www.jianshu.com/p/7e02fa48d7bc)

　　　[NSDictionary和NSMutableDictionary常用方法总结](http://www.aichengxu.com/view/62409)


<br />

<br />


### 文件读写操作


1. [iOS开发之沙盒机制（SandBox)](http://www.superqq.com/blog/2015/07/20/ioskai-fa-zhi-sha-he-ji-zhi-%28sandbox/)

　　　这是一篇关于沙盒的基础知识教程。简述沙盒的作用，对Documents、Library、tmp之间的区别做了介绍。通过两种方法打开沙盒，查看其中的内容。


2. [iOS开发之获取沙盒路径](http://www.superqq.com/blog/2015/07/22/ioskai-fa-zhi-huo-qu-sha-he-lu-jing/)

　　　沙盒里的文件夹包括Documents、Library、tmp。文章介绍了如何获取Documents、Library、Caches、tmp的路径。

3. [如何查看真机的沙盒（图文教程）](http://www.superqq.com/blog/2015/07/23/ru-he-cha-kan-zhen-ji-de-sha-he-%28tu-wen-jiao-cheng-%29/)

　　　通过图文的方式详细讲解如何查看真机沙盒。

4. [NSFileManager文件操作的十个小功能](http://www.superqq.com/blog/2015/07/24/nsfilemanagerwen-jian-cao-zuo-de-shi-ge-xiao-gong-neng/)

　　　NSFileManager是一个单列类，也是一个文件管理器。可以通过NSFileManager创建文件夹、创建文件、写文件、读文件内容等等基本功能。





<br />

<br />



### XML, JSON解析

- [XML解析](http://www.jianshu.com/p/2c63bcd29913)
- [JSON解析](http://www.jianshu.com/p/d46d75307192)
- [XML/JSON数据解析](http://www.jianshu.com/p/a54d367adb2a#)



<br />

<br />






### 对象，属性，封装，继承，多态

- [类与对象](https://hit-alibaba.github.io/interview/iOS/ObjC-Basic/Class.html)
- [对象](http://blog.devtang.com/2013/10/15/objective-c-object-model/)
- [属性](http://www.jianshu.com/p/b05d346c171b)
- [封装，继承，多态](http://www.cocoachina.com/ios/20141211/10609.html)


<br />

<br />









### 堆栈，内存管理

> 操作系统iOS 中应用程序使用的计算机内存不是统一分配空间，运行代码使用的空间在三个不同的内存区域，分成三个段：“text segment “，“stack segment ”，“heap segment ”。

<br />

- [内存管理1](http://www.jianshu.com/p/3129ce12e020)
- [内存管理2](https://gist.github.com/tedzhou/8899519)
- [内存管理3](https://hit-alibaba.github.io/interview/iOS/ObjC-Basic/MM.html)



<br />

<br />










### Category, Extension, Protocol

- [Category, Extension and Protocol](http://loupman.sinaapp.com/ios-learning-category-extension-and-protocol/)
- [Class](https://github.com/HIT-Alibaba/interview/blob/master/source/iOS/ObjC-Basic/Class.md)


<br />

<br />











### 便利构造器


- [便利构造器](http://www.lingxiblogs.com/?p=272)
- [继承与便利构造器](http://www.jianshu.com/p/4182ec5c9b36)

<br />

<br />









### Singleton

- [iOS中的单例模式](https://segmentfault.com/a/1190000003941840)
- [Singleton](http://www.superqq.com/blog/2015/06/13/ios-she-ji-mo-shi-xi-lie-:singleton-dan-li-mo-shi/)
- [单例模式](http://www.jianshu.com/p/7486ebfcd93b)

<br />

<br />










### block


- [OC 中的 Block](https://www.pupboss.com/objective-c-block/)
- [谈Objective-C block的实现](http://blog.devtang.com/2013/07/28/a-look-inside-blocks/)
- [Objective-C中的Block](https://onevcat.com/2011/11/objc-block/)
- [Block 基础](https://hit-alibaba.github.io/interview/iOS/ObjC-Basic/Block.html)

<br />

<br />






### 多线程



- [关于iOS多线程，你看我就够了（已更新）](http://www.cocoachina.com/ios/20150731/12819.html)
- [iOS多线程总结之GCD](http://linfeng1009.gitcafe.io/2015/09/29/thread-summary-gcd/)
- [OS X 和 iOS 中的多线程技术](http://www.infoq.com/cn/articles/os-x-ios-multithread-technology)
- [iOS多线程编程——GCD与NSOperation总结](https://bestswifter.com/multithreadconclusion/)


<br />
 
<br />



### 同步异步GetPost网络

- [网络](http://www.cnblogs.com/iCocos/p/4550267.html)




<br />
 
<br />




### KVO, KVC, NSNotification




- [观察者模式](http://www.cnblogs.com/ziyi--caolu/p/4867339.html)
- [KVC和KVO操作](http://www.wjdiankong.cn:8888/blog/?p=76)
- [NSNotificationCenter 使用姿势详解](http://www.jianshu.com/p/a4d519e4e0d5)



<br />
 
<br />




### 属性，setter, getter, copy, retain, assign

- [@property](http://www.devtalking.com/articles/you-should-to-know-property/)
- [属性关键字详解](http://www.wugaojun.com/blog/2015/07/25/at-propertyshu-xing-guan-jian-zi-xiang-jie/)
- [常用关键字的使用与区别](http://www.cnblogs.com/iCocos/p/4462744.html)



<br /><br /><br />













