---
layout: post
title: iOS中的事件传递与响应机制
tags: iOS
category: iOS
date: 2016-06-19 20:47:16
---

## iOS中事件的类型

iOS中的事件主要分为三种：

- 触摸事件（Milti-Touch Events）：单点、多点触控以及各种手势操作
- 加速事件（Motion Events）：重力、加速度传感器等
- 远程控制事件（Remote Control Events）：远程遥控iOS设备多媒体播放等


本文将主要针对触摸事件的传递，进行详细的介绍。

## 响应者对象

- 响应者是能够响应并处理事件的对象，是构成响应链和事件传递链的节点。
- 在iOS中不是任何对象都能处理事件，只有继承了UIResponder的对象才能接受并处理事件，我们称之为“响应者对象”。
- UIApplication、UIViewController、UIView都继承自UIResponder，因此它们都是响应者对象，都能够接收并处理事件。
- UIResponder 声明了用于处理事件的接口，并定义了默认的行为。

```objective-c
//UIResponder内部提供了以下方法来处理事件触摸事件
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event;
- (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event;
```

## 事件的传递

- 当我们点击iOS设备屏幕的时候，UIKit就会生成一个事件对象UIEvent，然后会把这个Event分发给当前active的App。系统会将该事件加入到一个由UIApplication管理的事件队列中
- UIApplication会从事件队列中取出最前面的事件，并将事件分发下去以便处理，通常，先发送事件给应用程序的主窗口
- 主窗口会在视图层次结构中找到一个最合适的视图来处理触摸事件

  **如何找到最合适的控件来处理事件？**

   - 自己是否能接收触摸事件
   - 触摸点是否在自己身上
   - 如果符合条件，从后往前遍历子控件，重复前面的两个步骤
   - 如果没有符合条件的子控件，那么就自己最适合处理

  如下这么一个UI布局,一种有 ABCDE 五个视图

  假设用户触摸了上图的 View E 区域，那么 iOS 将会按下面的顺序反复检测 subview 来寻找 Hit-Test View
  
  ![](http://7xw5tm.com1.z0.glb.clouddn.com/ABCDE.png)

  - 触摸区域在视图 A 内，所以检测视图 A 的 subview B 和 C；
  - 触摸区域不在视图 B 内，但是在视图 C 内，所以检查视图 C 的 subview D 和 E；
  - 触摸区域不在视图 D 内，在视图 E 中；
  - 视图 E 在整个视图体系中是 lowest view，所以视图 E 就是 Hit-Test View 。

  **寻找最合适的view的方法**
  
  iOS中使用Hit-Testing方法来找出这个触摸点下面的View，Hit-Testing会检测这个点击的点是不是发生在这个View的边界里面，如果是的话，就会去遍历这个View的subviews，直到找到最小的能够处理事件的view，如果整了一圈没找到能够处理的view，则返回自身。

  **用代码实现的大致思路如下**
  
```objective-c
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event
{
    // 1.判断当前控件能否接收事件
    if (self.userInteractionEnabled == NO || self.hidden == YES || self.alpha <= 0.01) return nil;
    // 2. 判断点在不在当前控件
    if ([self pointInside:point withEvent:event] == NO) return nil;
    // 3.从后往前遍历自己的子控件
    NSInteger count = self.subviews.count;
    for (NSInteger i = count - 1; i >= 0; i--) {
        UIView *childView = self.subviews[i];
        //把当前控件上的坐标系转换成子控件上的坐标系
        CGPoint childP = [self convertPoint:point toView:childView];
        UIView *fitView = [childView hitTest:childP withEvent:event];
        if (fitView) { 
          //寻找到最合适的view
          return fitView;
        }
    }
    // 循环结束,表示没有比自己更合适的view
    return self;
}
```

## 响应者链

响应者链，就是由一系列相互关联的响应者组成的链条。从firstResponder开始，到UIApplication结束。当一个事件发生后，找到最合适处理的视图，该视图就是firstResponder，如果自己不重写该响应者对事件的处理方法，那么该事件会被默认传递给nextResponder。依次传递，如果传递到UIApplication对象都没有进行处理，就会被抛弃。

## 下一个响应者
- 如果该视图是控制器的视图,那么控制器是下一响应者
- 否则，下一响应者是该视图的父视图
- 控制器的下一响应者是控制器视图的父视图

整个传递过程，官方文档给出了一张形象的图
![](http://7xw5tm.com1.z0.glb.clouddn.com/iOS_responder_chain_2x.png)

## 参考文章

[Event Handling Guide for iOS](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/event_delivery_responder_chain/event_delivery_responder_chain.html)
[史上最详细的iOS之事件的传递和响应机制](http://www.jianshu.com/p/2e074db792ba)

