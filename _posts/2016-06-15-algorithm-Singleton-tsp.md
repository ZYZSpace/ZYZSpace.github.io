---
layout: post
title: iOS下的单例设计模式
tags: iOS 单例设计模式
category: iOS
date: 2016-06-15 22:45:26
---

## 单例模式简介

单例模式是一个类在系统中只有一个实例对象,一般情况下会使用延时加载的策略，只在第一次需要使用的时候初始化。

在 iOS 中单例模式很常见，例如：

```objective-c
[NSUserDefaults standerUserDefaults];
[UIApplication sharedApplication];
[UIScreen mainScreen];
[NSFileManager defaultManager];
[UIDevice currentDevice];
```

这些都是单例模式。

## 为什么需要使用单例

- 节省内存开销。如果某个对象需要被多个其它对象使用，那可以考虑使用单例，因为这样该类只使用一份内存资源。例如UIDevice，每一个UIDevice的实例存储的信息都一样，因此可以使用单例，节约内存。
-  避免存在多个实例引起程序逻辑错误的场合。比如，大多数的软件都有一个（甚至多个）属性文件存放系统配置。这样的系统应当由一个对象来管理这些属性文件。

## 单例模式的正确实现
实现代码如下

```objective-c
#import <Foundation/Foundation.h>

@interface Singleton : NSObject

@property(nonatomic,strong) NSString *name;

+ (Singleton*)sharedInstance;

@end
```
```objective-c
#import "Singleton.h"

@implementation Singleton

static Singleton *sharedInstance = nil;

/**
 *  单例模式的接口，使用dispatch_once确保block里面的代码以线程安全的方式仅执行一次
 */
+(Singleton*)sharedInstance
{
    static dispatch_once_t token;
    dispatch_once(&token, ^{
        if(sharedInstance == nil)
        {
            sharedInstance = [[self alloc] init];
        }
    });
    return sharedInstance;
}

/**
 *  当用户调用alloc方法时，alloc方法会调用该方法，因此覆写该方法
 */
+(id)allocWithZone:(struct _NSZone *)zone
{
    static dispatch_once_t token;
    dispatch_once(&token, ^{
        if(sharedInstance == nil)
        {
            sharedInstance = [super allocWithZone:zone];
        }
    });
    return sharedInstance;
}

/**
 *  进行一些初始化工作
 */
- (instancetype)init
{
    self = [super init];
    if(self)
    {
        self.name = @"Singleton";
    }
    return self;
}

/**
 *  使用copy方法时返回相同对象
 */
- (id)copyWithZone:(struct _NSZone *)zone
{
    return self;
}

/**
 *  使用mutableCopy方法时返回相同对象
 */
- (id)mutableCopyWithZone:(struct _NSZone *)zone
{
    return self;
    
}

/**
 *  自定义输出信息，用于调试
 */
- (NSString *)description
{
    return [NSString stringWithFormat:@"memeory address:%p,property name:%@",self,self.name];
}

@end
```

测试

```objective-c
Singleton *sharedInstance =[Singleton sharedInstance];    
NSLog(@"sharedInstance:%@",sharedInstance);
Singleton *allocInstance = [[Singleton alloc] init];
NSLog(@"allocInstance:%@",allocInstance);
Singleton *copyInstance = [sharedInstance copy];
NSLog(@"copySingleton:%@",copyInstance);
Singleton *mutebleCopyInstance = [sharedInstance mutableCopy];
NSLog(@"mutebleCopyInstance:%@",mutebleCopyInstance);
```

输出结果如下

```objective-c
2016-06-16 21:24:42.765 Singleton[737:30080] sharedInstance:memeory address:0x7fc9b3405240,property name:Singleton
2016-06-16 21:24:42.766 Singleton[737:30080] allocInstance:memeory address:0x7fc9b3405240,property name:Singleton
2016-06-16 21:24:42.766 Singleton[737:30080] copySingleton:memeory address:0x7fc9b3405240,property name:Singleton
2016-06-16 21:24:42.766 Singleton[737:30080] mutebleCopyInstance:memeory address:0x7fc9b3405240,property name:Singleton
```

## 避免单例模式的滥用

单例模式给我们带来方便的同时也有一定的副作用，因为单例对象一旦创建，对象指针是保存在静态区的，单例对象在堆中分配的内存空间只有在程序终止后才会释放。这会引起一些问题：

- 如果过多的单例必定会增大我们消耗的内存，所以只有当我们确实需要唯一的使用对象时才需要考虑单例模式。

- 单例应该只用来保存全局的状态，并且不能和任何作用域绑定。如果这些状态的作用域比一个完整的应用程序的生命周期要短，那么这个状态就不应该使用单例来管理。
以阅读器为例，可以设计一个单例来保存用户的一些设置，如字体大小、背景颜色等。但是当更换用户时，这些设置应该更改为新用户的默认设置，然而，单例的生命周期却和应用程序的生命周期相同。因此，此处使用单例模式不合理。
