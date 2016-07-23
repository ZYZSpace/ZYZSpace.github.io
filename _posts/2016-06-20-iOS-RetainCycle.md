---
layout: post
title: iOS中常见的循环引用
tags: iOS 循环引用
category: iOS
date: 2016-06-21 23:57:17
---

## iOS中内存管理机制

说到循环，首先要提一下Objective-C的内存管理机制。

- Objective-C延续了C语言中手动管理内存的方式，提出了一种机制来减少内存管理的难度，该机制内存计数。

- 在Objective-C中，凡是继承自NSObject的类都提供了两种方法，retain和release。当我们调用一个对象的retain时，这个对象的内存计数加1，反之，当我们调用release时，对象的内存计数减1，只有当对象内存计数为0时，这个对象才真正会被释放，此时，对象的delloc方法会被调用来做些内存回收前的工作。

- 在ARC出现之后，retain和release的操作不需要我们亲自完成，编译器帮我们完成并进行一定的优化，内存管理的确方便很多，但是并非绝对安全绝，依然有内存泄漏的可能。


## 循环引用

导致iOS对象无法按预期释放的一个重要原因就是循环引用。循环引用可以简单理解为A引用了B，而B又引用了A，双方都同时保持对方的一个引用，导致任何时候引用计数都不为0，始终无法释放。
iOS中容易产生循环引用的地方有以下三种：

- delegate

- block

- NSTimer

## delegate引起的循环引用

让我们先来看一个简单的例子

```objective-c

@class ZYZView;

@protocol ZYZViewDelegate <NSObject>

- (void)ZYZViewTapped:(ZYZView *)view;

@end

@interface ZYZView : UIView

@property (nonatomic, strong) id<ZYZViewDelegate> delegate;

@end
```

```
@interface ViewController ()<ZYZViewDelegate>

@property (nonatomic, strong) ZYZView *zyzView;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    _zyzView = [[ZYZView alloc] init];
    _zyzView.delegate = self;
}

#pragma mark - ZYZViewDelegate

- (void)ZYZViewTapped:(ZYZView *)view {
    NSLog(@"ZYZViewTapped.");
}
```

在该程序中，ZYZView的delegate是strong属性，强持有了自己的代理，也就是ViewController。而ViewController又强持有的ZYZView，因此两个对象互相持有，形成了循环引用，造成内存泄漏。

**解决办法**

修改delegate为弱属性既可

```
@property (nonatomic, weak) id<ZYZViewDelegate> delegate;
```
由于delegate是弱属性，不再持有ViewController，因此不会形成循环引用。

## block引起的循环引用
block会对block内部用到的对象进行强引用,因此对block使用的不当也会引起循环引用

```objective-c
@property (nonatomic, copy) void(^myBlock)(void);

self.myBlock = ^ {
  NSLog(@"%@", self.name);
};
```

在该例子中，由于self持有block，block对内部用到的对象进行了强引用，因此block也持有了self，形成了循环引用。

**解决办法**

```
__weak typeof(self) weakSelf = self;
self.myBlock = ^ {
  __strong typeof(weakSelf) strongSelf = weakSelf;
  NSLog(@"%@", self.name);
};

```

通过引入弱引用，打破了循环引用。
但是block里面为什么又要使用强引用呢？
这是因为防止在多线程环境下，self被其他线程释放，从而导致的程序出错。

## NSTimer引起的循环引用
NSTimer会retain它的target，这样如果在控制器的一个属性是NSTimer的target指定为self，则会引起循环引用。

**解决办法**
为NSTimer添加一个Category方法

```
#import <Foundation/Foundation.h>
@interface NSTimer (BlockSupport)
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)interval block:(void(^)())block repeats:(BOOL)repeats;
@end
#import "NSTimer+BlockSupport.h"
@implementation NSTimer (BlockSupport)

+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)interval block:(void(^)())block repeats:(BOOL)repeats { 
return [self scheduledTimerWithTimeInterval:interval target:self selector:@selector(blockInvoke:) userInfo:[block copy] repeats:repeats];
}

+ (void)blockInvoke:(NSTimer *)timer { 
  void (^block)() = timer.userInfo; 
  if (block) { 
    block(); 
  }
}
@end
```


这个分类支持使用Block创建NSTimer，把操作传递到了万能对象userInfo里面，之后在控制器当中以这样的方式创建：

```
__block typeof(self) weakSelf = self;
_timer = [NSTimer scheduledTimerWithTimeInterval:0.5 block:^{ 
  __block typeof(self) strongSelf = weakSelf;
  [strongSelf doSth];
} repeats:YES];
```

先定义了一个弱引用，令其指向self，然后使块捕获这个引用，而不直接去捕获普通的self变量。也就是说，self不会为计时器所保留。当块开始执行时，立刻生成strong引用，以保证实例在执行期间持续存活。


## 参考资料
[正确使用Block避免Cycle Retain和Crash](http://tanqisen.github.io/blog/2013/04/19/gcd-block-cycle-retain/)
[用Block解决NSTimer循环引用](http://www.jianshu.com/p/1dbd7a228a22)



