---
title: iOS-消息转发机制
date: 2018.06.27 17:04
tags:
---
> iOS开发过程中我们经常会碰到这样的报错：unrecognized selector sent to instance ** 原因就是调用一个该对象没有实现的方法。用OC消息机制来说就是：消息的接收者不过到对应的selector，这样就启动了消息转发机制，我们可以通过代码在消息转发的过程中告诉对象应该如何处理未知的消息，默认实现是抛出下面的异常

<!-- more -->
### 消息发送

给一个对象发送一条消息：

```
[obj messageName:parameter] -> objc_msgSend(obj,SEL, parameter)
```

1. 通过 NSObject 的 isa 指针找到对应的 Class
2. 在 Class 的方法列表中找到对应的 selector
3. 如果在当前 Class 中未能找到 selector 则往父类的方法列表中继续查找
4. 如果能找到对应的 selector 则去执行对象的方法实现（IMP）
5. 如果还是没找到就要开始进入动态方法解析和消息转发

在上述流程中如果不能找对对应的 selector 时，这时候就会进入消息转发机制。消息转发机制可分为两个阶段，在这两个阶段中，有 3 次机会来处理之前未能处理 selector，越往后所花费的代价将越大，处理的灵活程度也就越高。如下图所示：

![messageforward.png](https://github.com/MaricleZhang/reasource/blob/master/message_forward.png?raw=true)

### 动态特性：方法解析和消息转发

#### 动态方法解析阶段

在该阶段中，可以动态的为类添加一个方法，从而让动态添加的方法来处理之前未能处理的消息。可重写类以下方法：

```
+ (BOOL)resolveInstanceMethod:(SEL)sel
```
如果是类的静态方法，可重写以下方法：

```
+ (BOOL)resolveClassMethod:(SEL)sel
```

SEL 就是未能处理的 selector，返回值为 BOOL 表示是否增加了新的方法来处理该 selector

#### 备援接收阶段

在该阶段中我们可以将未知的 selector 转发给其他对象来处理。运行时提供两次机会，来做消息的转发，第一次是重写以下方法：

```
- (id)forwardingTargetForSelector:(SEL)aSelector
```
#### 完整消息转发

如果不重写该方法，运行时将把方法调用的所有细节封装到 NSInvocation 对象中，进入完整的消息转发机制中，运行时将继续调用一下方法来进行消息的派发：

```
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector
- (void)forwardInvocation:(NSInvocation *)anInvocation
```

方法中的 NSInvocation 参数，包含了所有方法调用的细节，包括 selector/target/参数 等，重写该方法后我们可以将 anInvocation 转发给多个对象来处理该消息。在该阶段我们可以用来实现 “多重继承” 或者多重代理等。

如果在两个阶段都不做任何处理的话，运行时将会把 selector 交由 doesNotRecognizeSelector 方法来处理，从而抛出异常导致 Crash 

### 应用

#### 崩溃的规避

创建 `NSObject+CashHandle`的分


```
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector
{
    //方法签名
    return [NSMethodSignature signatureWithObjCTypes:"v@:@"];
}

- (void)forwardInvocation:(NSInvocation *)anInvocation
{
    NSLog(@"NSObject+CrashLogHandle---在类:%@中 未实现该方法:%@",NSStringFromClass([anInvocation.target class]),NSStringFromSelector(anInvocation.selector));
}

```
[Demo](https://github.com/MaricleZhang/MessageForwardingDemo.git)
