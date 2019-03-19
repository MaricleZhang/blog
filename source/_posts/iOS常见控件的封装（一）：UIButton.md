---
title: iOS常见控件的封装（一）：UIButton
date: 2016.10.17 20:56
tags:
---

>UIButton类在使用时需要一行行的来设置属性，代码过于冗余。每个点击事件都要创建一个方法，不利于阅读。下面就是我用block封装的UIButton的分类。

<!-- more -->

- UIButton+Block.h

```
 #import <UIKit/UIKit.h>
typedef void(^tapActionBlock)(UIButton *button);

@interface UIButton (Block)
@property(nonatomic,copy)tapActionBlock actionBlock;

/**
 通过block对button的点击事件封装
 
 @param frame       frame
 @param title       标题
 @param bgImageName 背景图片
 @param actionBlock 点击事件回调block
 
 @return button
 */
+ (UIButton *)createBtnFrame:(CGRect)frame title:(NSString *)title bgImageName:(NSString *)bgImageName action:(tapActionBlock)actionBlock;

@end
```
- UIButton+Block.m

```
#import "UIButton+Block.h"
#import <objc/runtime.h>

@implementation UIButton (Block)
static NSString *keyOfUseCategoryMethod;//用分类方法创建的button，关联对象的key
static NSString *keyOfBlock;

+ (UIButton *)createBtnFrame:(CGRect)frame title:(NSString *)title bgImageName:(NSString *)bgImageName action:(tapActionBlock)actionBlock
{
    
    UIButton *button = [UIButton buttonWithType:UIButtonTypeRoundedRect];
    button.frame = frame;
    [button setTitle:title forState:UIControlStateNormal];
    [button setBackgroundImage:[UIImage imageNamed:bgImageName] forState:UIControlStateNormal];
    [button addTarget:button action:@selector(tapAction:) forControlEvents:UIControlEventTouchUpInside];
    
    /**
     *用runtime中的函数通过key关联对象
     *
     *objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
     *id object                     表示关联者，是一个对象，变量名理所当然也是object
     *const void *key               获取被关联者的索引key
     *id value                      被关联者，这里是一个block
     *objc_AssociationPolicy policy 关联时采用的协议，有assign，retain，copy等协议，一般使用OBJC_ASSOCIATION_RETAIN_NONATOMIC
     
     */
    objc_setAssociatedObject (button , &keyOfUseCategoryMethod , actionBlock, OBJC_ASSOCIATION_COPY_NONATOMIC );
    
    return button;
}
- (void)tapAction:(UIButton*)sender
{
    /**
     * 通过key获取被关联对象
     *objc_getAssociatedObject(id object, const void *key)
     *
     */
    tapActionBlock block = ( tapActionBlock )objc_getAssociatedObject (sender , &keyOfUseCategoryMethod );
    
    if (block) {
        
        block(sender);
        
    }
}



- (void)setActionBlock:(tapActionBlock)actionBlock
{
    objc_setAssociatedObject (self , &keyOfBlock , actionBlock, OBJC_ASSOCIATION_COPY_NONATOMIC );
    
}

- (tapActionBlock)actionBlock
{
    return objc_getAssociatedObject (self , &keyOfBlock );
}


@end
```

这个封装的主要难点在于用runtime关联block，代码注释中有详细的解释，不足之处希望大家指正。想要了解更多或者下载demo,请访问github:https://github.com/Maricle1/ControlsPackage.git
