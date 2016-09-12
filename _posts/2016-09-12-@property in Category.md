---
layout: post
title: "@property in Category"
date: 2016-09-12
---

------

### 前言

我们是可以在`Category`中添加属性的，就像以下代码

```
/** FaiChouView.h */
#import <UIKit/UIKit.h>

@interface FaiChouView : UIView

@end

@interface FaiChouView (fcHeight)
@property CGFloat fcHeight;
@end

/** FaiChouView.m */
#import "FaiChouView.h"

@implementation FaiChouView
@end
@implementation FaiChouView (fcHeight)

- (CGFloat)fcHeight {
    return self.fcHeight;
}
- (void)setFcHeight:(CGFloat)fcHeight {
    self.fcHeight = fcHeight;
}
@end

/** main.m */
#import "FaiChouView.h"
int main(int argc, char * argv[]) {
    @autoreleasepool {
    	FaiChouView *fcView = [[FaiChouView alloc] init];
        fcView.fcHeight = 20.; 
        NSLog(@"%f", fcView.fcHeight);
    }
}
```

毫无疑问，它会崩溃的。问题出在哪？如何修改？一步一步来。

### 到底可不可以添加属性到`Category`？

我们可以在[苹果官方文档](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/CustomizingExistingClasses/CustomizingExistingClasses.html)中找到以下说明:

> Categories can be used to declare either instance methods or class methods but are not usually suitable for declaring additional properties. It’s valid syntax to include a property declaration in a category interface, but it’s not possible to declare an additional instance variable in a category. This means the compiler won’t synthesize any instance variable, nor will it synthesize any property accessor methods. You can write your own accessor methods in the category implementation, but you won’t be able to keep track of a value for that property unless it’s already stored by the original class.

文档中明确指出`It’s valid syntax to include a property declaration in a category interface`，我们可以在接口文件中声明属性，但是编译器不会自动合成实例变量(Ivars)和存取方法。
我们应该自己实现存取方法。

我们的代码到底错在什么地方？

`won’t be able to keep track of a value for that property`

我们的代码`return self.fcHeight;`已经无法无天了。

### 如何修改代码让其正常获取`view`的高度？

```
@implementation FaiChouView (fcHeight)

- (CGFloat)fcHeight {
    // return self.fcHeight;
    
    return self.frame.size.height;
}
- (void)setFcHeight:(CGFloat)fcHeight {
    
    // self.fcHeight = fcHeight;
    
    CGRect newframe = self.frame;
    newframe.size.height = fcHeight;
    self.frame = newframe;
}

@end
```

官方文档的说明，我们在`Category`中的属性是不能够保存其值的，但是我们可以自定义存取方法，`fcHeight`返回的是`view`本身的高度, `setter`方法也不将值赋值给`fcHeight`，而间接的给`view.frame`，这样在`main.m`中的`fcView.fcHeight = 20.; `只是借助了`fcHeight`的存取方法给`view`本身的赋值。


通过以上，我们验证了那句古话
> 不能在`Category`中添加实例变量

### 学而不思则罔，思而不学则殆

只要挖到`runtime`，任何东西都是一目了然的。
有些博客看了一遍看不懂，那就多看几遍，博客讲的只是一个方面，上面有许多许多知识点，这一面映射出来的所有点没有必要全部掌握，发现问题的关键，带着问题考虑，总会学到很多知识的。

> 少谈些主义，多研究些问题。 ———— 胡适


objc所有类和对象都是c结构体，category当然也一样，下面是`runtime`中Category的结构：

```
struct _category_t {
	const char *name; // 类名
	struct _class_t *cls; //
	const struct _method_list_t *instance_methods; // 实例方法 -
	const struct _method_list_t *class_methods; // 类方法 +
	const struct _protocol_list_t *protocols; // 
	const struct _prop_list_t *properties; //
};
```

`properties`这个category所有的property，这也是category里面可以定义属性的原因，不过这个property不会合成实例变量和存取方法。

一个普通的属性(`fcTestProperty`)，经过编译器编译过后，会添加以下:

1. 在`_ivar_list_t`中添加了`_fcTestProperty变量`
2. 在`_method_list_t`中添加了`fcTestProperty`和`setFcTestProperty`两个方法
3. 在`_prop_list_t`中添加了`fcTestProperty`这个property

一个`Category`中的属性(`fcTestProperty`),经过编译器编译后，只会在`_prop_list_t`中增加`fcTestProperty`这个属性。

### 如何在`Category`中添加实例变量呢？

在`MJRefresh`中我们可以找到以下代码(摘要):

```
/** UIScrollView+MJRefresh.h */

#import <UIKit/UIKit.h>
#import "MJRefreshConst.h"

@class MJRefreshHeader, MJRefreshFooter;

@interface UIScrollView (MJRefresh)
/** 下拉刷新控件 */
@property (strong, nonatomic) MJRefreshHeader *mj_header;

...

@end


/** UIScrollView+MJRefresh.m */

#pragma mark - header
static const char MJRefreshHeaderKey = '\0';
- (void)setMj_header:(MJRefreshHeader *)mj_header
{
    if (mj_header != self.mj_header) {
        // 删除旧的，添加新的
        [self.mj_header removeFromSuperview];
        [self insertSubview:mj_header atIndex:0];
        
        // 存储新的
        [self willChangeValueForKey:@"mj_header"]; // KVO
        objc_setAssociatedObject(self, &MJRefreshHeaderKey,
                                 mj_header, OBJC_ASSOCIATION_ASSIGN);
        [self didChangeValueForKey:@"mj_header"]; // KVO
    }
}

- (MJRefreshHeader *)mj_header
{
    return objc_getAssociatedObject(self, &MJRefreshHeaderKey);
}

...

```

他是通过runtime中的关联对象实现的，关于`Associated Objects`可以学习[NSHipster的魔鬼的交易](http://nshipster.cn/associated-objects/)这一篇。

最后我们的代码调整为:

```
/** FaiChouView.m */
#import "FaiChouView.h"
#import <objc/runtime.h>

@implementation FaiChouView
@end

@implementation FaiChouView (fcHeight)
@dynamic fcHeight;
- (CGFloat)fcHeight {
    // return self.fcHeight;
    
    // return self.frame.size.height;
    return [objc_getAssociatedObject(self, @selector(fcHeight)) floatValue];
}
- (void)setFcHeight:(CGFloat)fcHeightNew {
    
    // self.fcHeight = fcHeight;
    
//    CGRect newframe = self.frame;
//    newframe.size.height = fcHeight;
//    self.frame = newframe;
    NSNumber *fcHeightFloatNumber = @(fcHeightNew);
    objc_setAssociatedObject(self, @selector(fcHeight), fcHeightFloatNumber, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

@end
```

> 用`objc_setAssociatedObject`和`objc_getAssociatedObject`方法绑定的实例变量与一个普通的实例变量完全是两码事。

### 参考链接

- [Category中property的命运](http://morisunshine.com/ios/the_destiny_of_property_in_category/)

- [objc category的秘密](http://blog.sunnyxx.com/2014/03/05/objc_category_secret/)

- [Associated Objects](http://nshipster.cn/associated-objects/)

- [刨根问底Objective－C Runtime（3）－ 消息 和 Category](http://chun.tips/blog/2014/11/06/bao-gen-wen-di-objective%5Bnil%5Dc-runtime(3)%5Bnil%5D-xiao-xi-he-category/)

- [iOS Category中添加属性和成员变量的区别](http://www.jianshu.com/p/535d1574cb86)

- [iOS使用Category添加@property变量](http://www.jianshu.com/p/922cd6220e4e)
