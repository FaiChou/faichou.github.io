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

### 事件分发

第一响应者（First responder）指的是当前接受触摸的响应者对象（通常是一个 UIView 对象），即表示当前该对象正在与用户交互，它是响应者链的开端。响应者链和事件分发的使命都是找出第一响应者。

iOS 系统检测到手指触摸 (Touch) 操作时会将其打包成一个 UIEvent 对象，并放入当前活动 Application 的事件队列，单例的 UIApplication 会从事件队列中取出触摸事件并传递给单例的 UIWindow 来处理，UIWindow 对象首先会使用 `hitTest:withEvent:`方法寻找此次 Touch 操作初始点所在的视图(View)，即需要将触摸事件传递给其处理的视图，这个过程称之为 hit-test view。

`hitTest:withEvent:`方法的处理流程如下:

- 首先调用当前视图的 pointInside:withEvent: 方法判断触摸点是否在当前视图内；
- 若返回 NO, 则 `hitTest:withEvent:` 返回 nil，若返回 YES, 则向当前视图的所有子视图 (subviews) 发送 `hitTest:withEvent: `消息，所有子视图的遍历顺序是从最顶层视图一直到到最底层视图，即从 subviews 数组的末尾向前遍历，直到有子视图返回非空对象或者全部子视图遍历完毕；
- 若第一次有子视图返回非空对象，则   ` hitTest:withEvent: `方法返回此对象，处理结束；
- 如所有子视图都返回空，则    ` hitTest:withEvent:` 方法返回自身 (self)。

示例性代码：

```
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event{
    UIView *touchView = self;
    if ([self pointInside:point withEvent:event] &&
       (!self.hidden) &&
       self.userInteractionEnabled &&
       (self.alpha >= 0.01f)) {

        for (UIView *subView in self.subviews) {
            [subview convertPoint:point fromView:self];
            UIView *subTouchView = [subView hitTest:subPoint withEvent:event];
            if (subTouchView) {
                touchView = subTouchView;
                break;
            }
        }
    }else{
        touchView = nil;
    }

    return touchView;
}
``` 

说明：

1. 如果最终 hit-test 没有找到第一响应者，或者第一响应者没有处理该事件，则该事件会沿着响应者链向上回溯，如果 UIWindow 实例和 UIApplication 实例都不能处理该事件，则该事件会被丢弃（这个过程即上面提到的响应值链）；
2. `hitTest:withEvent: `方法将会忽略隐藏 (hidden=YES) 的视图，禁止用户操作 `(userInteractionEnabled=NO)` 的视图，以及 alpha 级别小于 0.01(alpha<0.01)的视图。如果一个子视图的区域超过父视图的 bound 区域(父视图的 `clipsToBounds` 属性为 NO，这样超过父视图 bound 区域的子视图内容也会显示)，那么正常情况下对子视图在父视图之外区域的触摸操作不会被识别, 因为父视图的 `pointInside:withEvent:` 方法会返回 NO, 这样就不会继续向下遍历子视图了。当然，也可以重写 `pointInside:withEvent:` 方法来处理这种情况。
3. 我们可以重写 `hitTest:withEvent:` 来达到某些特定的目的。




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

![App生命周期示意图](http://o7bkcj7d7.bkt.clouddn.com/UIApplicationDelegateMessagingWithMultitasking.png-fc.normalImage)


## 3. UIView


UIView 在屏幕上定义了一个矩形区域，并且为如何管理此矩形区域创造了接口。
它在iOS App中占有绝对重要的地位，因为iOS中几乎所有可视化控件都是由UIView或其子类构成。

UIView 的任务：

1. 绘制和动画
2. 布局和子视图管理
3. 事件处理


### 绘制和动画

#### 视图绘制

UIView 是按需绘制的，当整个视图或者视图的一部分由于布局变化，变成可变的，系统会要求视图进行绘制。对于那些需要使用UIKit或者CoreGraphics进行自定义绘制的视图，系统会调用`drawRect:`方法进行绘制。

当视图内容发生变化，需要调用`setNeedsDisplay`或者`setNeedsDisplayInRect:`方法，告诉系统该重新绘制这个视图。调用此方法之后，系统会在下一个绘制周期更新这个视图的内容。由于系统要等到下一个绘制周期才真正进行绘制，可以一次性对多个视图调用`setNeedsDisplay`，它们会同时被更新。

#### 视图的几何属性

视图有 frame, center, bounds 等几个基本几何属性，其中：

- frame 使用的最多，其坐标位置都是相对于父视图的，可以用于确定本视图在父视图中的位置和其自身的大小。
- center 的坐标位置也是相对于父视图的，通常用于移动，旋转等动画操作，而不会改变其自身的大小。
- bounds 是相对于自身的，通常情况下就是`(0, 0, width, height)`, bounds 的含义可以认为是当前view被允许绘制的范围。

#### 视图的 ContentMode

视图在初次绘制完成后，系统会对绘制结果进行快照，之后尽可能的使用快照，避免重新绘制。如果视图的几何属性发生变化，系统会根据视图的contentMode来决定如何改变显示效果。

默认的contentMode是`UIViewContenModeScaleToFill`，系统会拉伸当前的快照，使其符合新的frame尺寸。大部分contentMode都会对当前的快照进行拉伸或者移动等操作。如果需要重新绘制，可以把contentMode设置为`UIVeiwContentModeRedraw`，强制视图在改变大小之类的操作时调用`drawRect:`重绘。

#### 动画

可以以动画的形式改变视图的下面这些属性，只需要告诉系统动画开始和结束时的数值，系统会自动处理中间的过渡程度。

- frame
- bounds
- center
- transform
- alpha
- backgroundColor
- contentStretch

### 布局和子视图管理

除了提供视图本身内容之外，一个视图也可以表现的想一个容器。当一个视图包含其他视图时，两个视图之间就创建了一个父子关系。在这个关系中子视图被称作subView，父视图被称作superView。一个视图可以包含多个子视图，它们被存放在这个视图的subViews数组里。添加，删除，以及调动操作这些子视图的函数如下：

- `addSubview:`
- `insertSubview:...`
- `bringSubviewToFront:`
- `sendSubviewToBack:`
- `exchangeSubviewAtIndex: withSubviewAtIndex:`
- `removeFromSuperView`

### AutoResizing 和 Constraint

当一个视图的大小改变时，它的子视图的位置和大小也需要相应地改变。UIView支持自动布局，也可以手动对子视图进行布局。

当下列这些事件发生时，需要进行布局操作：

- 视图的bounds 大小改变
- 用户界面旋转，通常会导致根视图控制器的大小改变
- 视图的layer 层的 Core Animation sublayers 发生改变
- 程序调用视图的`setNeedsLayout`或`layoutIfNeeded`方法
- 程序调用视图layer 的`setNeedsLayout`方法

#### Autoresizing

视图的`autoresizesSubviews`属性决定了在视图大小发生变化时，如何自动调节子视图。

掩码如下：

- UIViewAutoresizingNone
- UIViewAutoresizingFlexibleHeight
- UIViewAutoresizingFlexibleWidth
- UIViewAutoresizingFlexibleLeftMargin
- UIViewAutoresizingFlexibleRightMargin
- UIViewAutoresizingFlexibleBottomMargin
- UIViewAutoresizingFlexibleTopMargin

可以通过位运算符组合他们，例如：`UIViewAutoresizingFlexibleHeight | UIViewAutoresizingFlexibleWidth`

#### Constraint

Constraint 时另一种用于自动布局的方法，本质上Constraint就是对UIView之间两个属性之间的一个约束：

    attribute1 == multiplier * attribute2 + constant
    
其中方程两边不一定时等于，也可以时大于等于之类的关系。

Constraint比AutoResizing更加灵活和强大，可以实现复杂的子视图布局。

#### 自定义layout

UIView 中提供了一个`layoutSubviews`函数，UIView的子类可以重载这个函数，以实现更加复杂和精细的子View布局。

苹果文档专门强调了，应该只在上面提到的 Autoresizing 和 Constraint 机制不能实现所需要的效果时，才使用 layoutSubviews。而且，layoutSubviews 方法只能被系统触发调用，程序员不能手动直接调用该方法。

### 事件处理

UIView时UIResponder的子类，可以响应触控事件。

通常可以使用`addGestureRecongnizer:`添加手势识别器来响应触控事件，如果需要手动处理，则按需要重载下面的函数：

- `touchesBegan: withEvent:`
- `touchesMoved: withEvent:`
- `touchesEnded: withEvent:`
- `touchesCancelled: withEvent:`

## 4. UIViewController

UIViewController(视图控制器)是 iOS Apps 管理视图的基本工具。它是MVC设计模式中的C部分。UIViewController在UIKit中主要功能是用于控制画面的切换，其中的`view`属性(UIView类型)管理整个画面的外观。

### UIViewController 生命周期

UIViewController生命周期的第一步是初始化。不过具体调用的方法在SB和纯代码中还有所不同，这里只讲代码。代码中我们可以使用`init:`函数进行初始化，`init:`函数在实现过程中还会调用`initWithNibName:bundle:`。我们应该尽量避免在VC外部调用`initWithNibName:bundle:`，而是把它放在VC的内部。原因如下：

**苹果官方要求**

假如我们这样：

```
FCViewController *fcVC = [[FCViewController alloc] initWithNibName:@"Myview" bundle:nil];
```

那么我可以告诉你，这是不规范的，违反了类的封装原则。

我们应该这样写：

```
FCViewController *fcVC = [[FCViewController alloc] init];
```

在 `FCViewController`里：

```
- (id)init
{
   self = [super initWithNibName:@"Myview" bundle:nil];
   if (self != nil)
   {
       // Further initialization if needed
   }
   return self;
}

- (id)initWithNibName:(NSString *)nibName bundle:(NSBundle *)bundle
{
    NSAssert(NO, @"Initialize with -init");
    return nil;
}
```

这样就会将主权交还给对象，防止破坏封装原则。如果你修改了NIB的名字，就不需要在每个实例的初始化地方修改，只需要在一个VC里修改就可以了。


---

初始化完成后，VC的生命周期会经过下面几个函数：

```
(void)loadView
(void)viewDidLoad
(void)viewWillAppear
(void)viewWillLayoutSubviews
(void)viewDidLayoutSubviews
(void)viewDidAppear
(void)viewWillDisappear
(void)viewDidDisappear
```


假设现在有一个 AViewController(简称 Avc) 和 BViewController (简称 Bvc)，通过 navigationController 的 push 实现 Avc 到 Bvc 的跳转，下面是各个方法的执行执行顺序：

```
1. A viewDidLoad  
2. A viewWillAppear  
3. A viewDidAppear  
4. B viewDidLoad  
5. A viewWillDisappear  
6. B viewWillAppear  
7. A viewDidDisappear  
8. B viewDidAppear  
```

如果再从 Bvc 跳回 Avc，会产生下面的执行顺序：

```
1. B viewWillDisappear  
2. A viewWillAppear  
3. B viewDidDisappear  
4. A viewDidAppear  
```

可见 viewDidLoad 只会调用一次，再第二次跳回 Avc 的时候，AViewController 仍然存在于内存中，也就不需要 load 了。



## 5. 动画

## 6. 网络编程

## 7. 并发编程

iOS 中的多线程，是 Cocoa 框架下的多线程，通过 Cocoa 的封装，可以让我们更为方便的进行多线程编程。

Cocoa 中封装了 NSThread, NSOperation, GCD 三种多线程编程方式。

- NSThread

> NSThread 是一个控制线程执行的对象，通过它我们可以方便的得到一个线程并控制它。NSThread 的线程之间的并发控制，是需要我们自己来控制的，可以通过 NSCondition 实现。它的缺点是需要自己维护线程的生命周期和线程的同步和互斥等，优点是轻量，灵活。

- NSOperation

> NSOperation 是一个抽象类，它封装了线程的细节实现，不需要自己管理线程的生命周期和线程的同步和互斥等。只是需要关注自己的业务逻辑处理，需要和 NSOperationQueue 一起使用。使用 NSOperation 时，你可以很方便的设置线程之间的依赖关系。这在略微复杂的业务需求中尤为重要。

- GCD

> GCD(Grand Central Dispatch) 是 Apple 开发的一个多核编程的解决方法。在 iOS4.0 开始之后才能使用。GCD 是一个可以替代 NSThread 的很高效和强大的技术。当实现简单的需求时，GCD 是一个不错的选择。

而 NSThread 大多被淘汰了，苹果推荐使用 GCD 和 NSOperation。

### GCD

[GCD 深入理解](https://github.com/nixzhu/dev-blog/blob/master/2014-04-19-grand-central-dispatch-in-depth-part-1.md)

[理解 iOS 开发中 GCD 相关的同步（synchronization）\ 异步（asynchronization），串行（serial）\ 并行（concurrency）概念](http://huuang.com/?p=83)

[并发编程之GCD](https://www.chenghu.me/?p=942)

关于死锁问题，可以看我简书的[一篇](http://www.jianshu.com/p/f91773c21a21)。


### NSOperation 和 NSOperationQueue

NSOperation 和 NSOperationQueue 并不一定要一起使用。
NSOperation 本身是可以单独使用的，不过单独使用的话并不能体现出 NSOperation 的强大之处（从下面的部分你就能看出单独用 NSOperation 真的是做不了什么事情），通常还是使用 NSOperationQueue 来执行 NSOperation。

NSOperation 是一个抽象类，我们需要继承它并且实现我们的子类。

#### 并发和非并发

#### 在 NSOperationQueue 中运行

##### Dependency

##### Cancellation

##### maxConcurrentOperationCount

##### Queue 的优先级


### GCD 与 NSOperation 的对比

这是面试中经常会问到的一点，这两个都很常用，也都很强大。对比它们可以从下面几个角度来说：

- 首先要明确一点，NSOperationQueue 是基于 GCD 的更高层的封装，从 OS X 10.10 开始可以通过设置 `underlyingQueue` 来把 operation 放到已有的 dispatch queue 中。
- 从易用性角度，GCD 由于采用 C 风格的 API，在调用上比使用面向对象风格的 NSOperation 要简单一些。
- 从对任务的控制性来说，NSOperation 显著得好于 GCD，和 GCD 相比支持了 Cancel 操作，支持任务之间的依赖关系，支持同一个队列中任务的优先级设置，同时还可以通过 KVO 来监控任务的执行情况。这些通过 GCD 也可以实现，不过需要很多代码，使用 NSOperation 显得方便了很多。
- 从第三方库的角度，知名的第三方库如 AFNetworking 和 SDWebImage 背后都是使用 NSOperation，也从另一方面说明对于需要复杂并发控制的需求，NSOperation 是更好的选择。



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
 

 
