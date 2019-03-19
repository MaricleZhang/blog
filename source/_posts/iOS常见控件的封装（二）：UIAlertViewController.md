---
title: iOS常见控件的封装（二）：UIAlertViewController
date: 2016.10.24 23:40
tags:
---

>UIAlertViewController类在iOS开发中经常使用，但是使用系统方法需要太多的代码，所以我自己封装了一个类。在一个block中实现点击事件。

<!-- more -->

- UIAlertController+Category.h

```
#import <UIKit/UIKit.h>

typedef void (^CallBackBlock)(NSInteger btnIndex);

@interface UIAlertController (Category)

/**
 自定义封装的UIAlertController方法

 @param viewController       显示的vc
 @param alertControllerStyle UIAlertControllerStyle 样式
 @param title                标题
 @param message              提示信息
 @param block                回调block
 @param cancelBtnTitle       取消button标题
 @param destructiveBtnTitle  红色的按钮
 @param otherBtnTitles       其他button标题
 */
+ (void)showAlertCntrollerWithViewController:(UIViewController*)viewController alertControllerStyle:(UIAlertControllerStyle)alertControllerStyle title:(NSString*)title message:(NSString*)message CallBackBlock:(CallBackBlock)block cancelButtonTitle:(NSString *)cancelBtnTitle
                    destructiveButtonTitle:(NSString *)destructiveBtnTitle
                         otherButtonTitles:(NSString *)otherBtnTitles,...NS_REQUIRES_NIL_TERMINATION;

@end
```

- UIAlertController+Category.h

```
#import "UIAlertController+Category.h"

@implementation UIAlertController (Category)

+(void)showAlertCntrollerWithViewController:(UIViewController *)viewController alertControllerStyle:(UIAlertControllerStyle)alertControllerStyle title:(NSString *)title message:(NSString *)message CallBackBlock:(CallBackBlock)block cancelButtonTitle:(NSString *)cancelBtnTitle destructiveButtonTitle:(NSString *)destructiveBtnTitle otherButtonTitles:(NSString *)otherBtnTitles, ...
{
    UIAlertController * alertController = [UIAlertController alertControllerWithTitle:title message:message preferredStyle:alertControllerStyle];
    //添加按钮
    if (cancelBtnTitle.length) {
        UIAlertAction * cancelAction = [UIAlertAction actionWithTitle:cancelBtnTitle style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
            block(0);
        }];
        [alertController addAction:cancelAction];
    }
    if (destructiveBtnTitle.length) {
        UIAlertAction * destructiveAction = [UIAlertAction actionWithTitle:destructiveBtnTitle style:UIAlertActionStyleDestructive handler:^(UIAlertAction * _Nonnull action) {
            block(1);
        }];
        [alertController addAction:destructiveAction];
    }
    if (otherBtnTitles.length) {
        UIAlertAction *otherActions = [UIAlertAction actionWithTitle:otherBtnTitles style:UIAlertActionStyleDefault handler:^(UIAlertAction *action) {
            (!cancelBtnTitle.length && !destructiveBtnTitle.length) ? block(0) : (((cancelBtnTitle.length && !destructiveBtnTitle.length) || (!cancelBtnTitle.length && destructiveBtnTitle.length)) ? block(1) : block(2));
        }];
        [alertController addAction:otherActions];
        /**
         *  va_list : （1）首先在函数里定义一具VA_LIST型的变量，这个变量是指向参数的指针；
         *            （2）然后用VA_START宏初始化变量刚定义的VA_LIST变量；
         *            （3）然后用VA_ARG返回可变的参数，VA_ARG的第二个参数是你要返回的参数的类型（如果函数有多个可变参数的，依次调用VA_ARG获取各个参数）；
         *            （4）最后用VA_END宏结束可变参数的获取。
         *   va_start :获取可变参数列表的第一个参数的地址;
         *   va_arg :获取当前参数，返回指定类型并将指针指向下一参数
         *   va_end :清空va_list可变参数列表：
         *
         *
         */
        va_list args;
        va_start(args, otherBtnTitles);
        if (otherBtnTitles.length) {
            NSString * otherString;
            int index = 2;
            (!cancelBtnTitle.length && !destructiveBtnTitle.length) ? (index = 0) : ((cancelBtnTitle.length && !destructiveBtnTitle.length) || (!cancelBtnTitle.length && destructiveBtnTitle.length) ? (index = 1) : (index = 2));
            while ((otherString = va_arg(args, NSString*))) {
                index ++ ;
                UIAlertAction * otherActions = [UIAlertAction actionWithTitle:otherString style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
                    block(index);
                }];
                [alertController addAction:otherActions];
            }
        }
        va_end(args);
    }
    [viewController presentViewController:alertController animated:YES completion:nil];
}
@end
```
其实主要的难点就是循环获取otherButtons,代码中有详细的介绍，不足之处希望大家指正。想要了解更多或者下载demo,请访问github:[https://github.com/Maricle1/ControlsPackage.git](https://github.com/Maricle1/ControlsPackage.git)
