---
layout: post
title: "Cocoa Touch"
date: 2016-05-09
---

---

## 1. 事件处理
```
NSObject:
	UIResponder:
		SKNode
		UIApplication
		UIView
		UIViewController
```

`UIResponder`给对象定义了响应和处理事件的接口。
它是`UIApplication`, `UIView`的父类，是`UIWindow`的爷类，(~LOL)。

通常的两种事件：触摸事件（`touch events`）和运动事件（`motion events`）。

**触摸事件**的方法：

```
touchesBegan:withEvent:
touchesMoved:withEvent:
touchesEnded:withEvent:
touchesCancelled:withEvent:
```
这些方法的参数与触摸事件紧密相连，例如刚开始触摸或者触摸发生改变，因此对象可以跟踪和处理这些事件。任何时间点，当你触摸屏幕，滑动屏幕，或者从屏幕挪开手指，都会生成一个`UIEvent`。这个事件对象包含了你对屏幕所做的任何事情，定义在`UITouch`类中。

**运动事件**的方法：

```
iOS 3.0:
 motionBegan:withEvent: 
 motionEnded:withEvent:
 motionCancelled:withEvent:
 canPerformAction:withSender:
iOS 4.0:
 remoteControlReceivedWithEvent:
```

在iOS 3.0苹果提供了对运动事件的处理方法，例如摇晃设备。


#### 管理响应者链
```
- nextResponder
- isFirstResponder
- canBecomeFirstResponder
- becomeFirstResponder
- canResignFirstResponder
- resignFirstResponder
```


#### 管理inputViews
```
inputView (Property)
inputViewController (Property)
inputAccessoryView (Property)
inputAccessoryViewController (Property)
- reloadInputViews
```

#### 触摸事件响应
```
- touchesBegan:withEvent:
- touchesMoved:withEvent:
- touchesEnded:withEvent:
- touchesCancelled:withEvent:
- touchesEstimatedPropertiesUpdated:
```

#### 运动事件响应

```
- motionBegan:withEvent:
- motionEnded:withEvent:
- motionCancelled:withEvent:
```

#### 按压事件响应

```
- pressesBegan:withEvent:
- pressesCancelled:withEvent:
- pressesChanged:withEvent:
- pressesEnded:withEvent:
```

#### 远程控制事件响应

```
- remoteControlReceivedWithEvent:
```



## 2. UIApplication

程序运行期间`UIApplication`完成集中控制和协调的工作。

每个app都必须有且仅有一个`UIApplication`的实例。

当app开始运行时，系统会调用`UIApplicationMain`方法来启动，它会创建一个`UIApplication`的单例。

UIApplication 的一个主要工作是处理用户事件，它会把所有用户事件都放入一个队列中，逐个处理；在处理的时候，它会将事件发送到一个合适的目标控件。

此外，UIApplication 实例还维护一个在本应用中打开的 window 列表（UIWindow 实例），这样它就可以接触应用中的任何一个 UIView 对象。


UIApplication 实例会被赋予一个代理对象，以处理应用程序的生命周期事件（比如程序启动和关闭）、系统事件（比如来电、记事项警告）等等。


### UIApplication生命周期

|状态|说明|
|:---|:---|
|**Not running**|程序没有启动|
|**Inactive**|前台运行，但无事件响应|
|**Active**|前台运行，且有事件响应|
|**Background**|后台运行，可以处理一些事件|
|**Suspended**|在后台不能处理事件，内存不够app就会被清除|



### 常见代理方法

- `(void)applicationWillResignActive:(UIApplication *)application`

当应用程序将要入非活动状态执行，在此期间，应用程序不接收消息或事件，比如来电话了

- `(void)applicationDidBecomeActive:(UIApplication *)application`

当应用程序入活动状态执行，这个刚好跟上面那个方法相反

- `(void)applicationDidEnterBackground:(UIApplication *)application`

当程序被推送到后台的时候调用。所以要设置后台继续运行，则在这个函数里面设置即可

- `(void)applicationWillEnterForeground:(UIApplication *)application`

当程序从后台将要重新回到前台时候调用，这个刚好跟上面的那个方法相反。

- `(void)applicationWillTerminate:(UIApplication *)application`

当程序将要退出是被调用，通常是用来保存数据和一些退出前的清理工作。这个需要设置 UIApplicationExitsOnSuspend 的键值。

- `(void)applicationDidReceiveMemoryWarning:(UIApplication *)application`

iPhone 设备只有有限的内存，如果为应用程序分配了太多内存操作系统会终止应用程序的运行，在终止前会执行这个方法，通常可以在这里进行内存清理工作防止程序被终止.

- `(void)applicationSignificantTimeChange:(UIApplication*)application`

当系统时间发生改变时执行

- `(void)applicationDidFinishLaunching:(UIApplication*)application`

当程序载入后执行

<br />

App生命周期示意图：

![App生命周期示意图](http://github.com/FaiChou/faichou.github.io/blob/master/img/UIApplication-Delegate-Messaging-with-Multitasking.png)

## 3. UIView

## 4. UIViewController

## 5. 动画

## 6. 网络编程

## 7. 并发编程

## 8. 文件系统

## 9. 设计模式

## 10. 性能

## 11. 网站 & Blogs

### 学习资料
 - [NSHipster](http://nshipster.cn/)
 - [objc中国](http://objccn.io/)
 - [Jamin's BLog](http://oncenote.com/)
 - [iOS禅](https://github.com/100mango/zen)
 - [禅与Objective-C开发艺术](https://github.com/oa414/objc-zen-book-cn)
 - [一些 iOS / Web 开发相关的翻译或原创博客文章](https://github.com/nixzhu/dev-blog)
 - [苹果官方 Sample 代码集合](https://github.com/robovm/apple-ios-samples)
 - [iOS应用架构](http://casatwy.com/iosying-yong-jia-gou-tan-kai-pian.html)
 - [iOS保持界面流畅](http://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)
 - [开发流程总结](https://github.com/leecade/ios-dev-flow)
 - [iOS最佳实践](https://github.com/DaiYue/iOS-good-practices-in-Chinese)
 - [中文iOS/Mac开发博客列表](https://github.com/tangqiaoboy/iOSBlogCN)
 - [iOS学习资料整理](https://github.com/Aufree/trip-to-iOS)
 - [iOS 性能优化](http://www.hrchen.com/2013/05/performance-with-instruments/)
 - [实用Demo仓库](https://github.com/huang303513/iOSBasicCommonDemos)
 - [框架笔记](https://github.com/seedante/iOS-Note)
 - [iOS开源App整理](http://duxinfeng.com/2015/07/14/iOS%E5%BC%80%E6%BA%90App%E6%95%B4%E7%90%86/)
 

### 编码规范
 - [Introduction to Coding Guidelines for Cocoa](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html#//apple_ref/doc/uid/10000146-SW1)
 - [The official raywenderlich.com Objective-C style guide.](https://github.com/raywenderlich/objective-c-style-guide)
 - [Objective-C style guide](https://github.com/github/objective-c-style-guide)
 
### 常用的库
 - [Github-iOS备忘](http://github.ibireme.com/github/list/ios/)
 - [TimLiu-iOS](https://github.com/Tim9Liu9/TimLiu-iOS)
 - [Awesome-iOS](https://github.com/vsouza/awesome-ios)
 - [Awesome-iOS-UI](https://github.com/cjwirth/awesome-ios-ui)
 

 
