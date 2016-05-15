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
 

 
