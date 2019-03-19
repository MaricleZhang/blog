
---
title: XCode-10-升级问题总结
date: 2018.09.29 16:45
tags:
---

### library not found for -lstdc++.6.0.9

xcode 10 中删除了内置 `libstdc++.6.0.9.tbd`，工程中一些SDK依赖这个库，需要把xcode 9.4 的` libstdc++.6.0.9.tbd ` 添加到xcode中，重启xcode。
libstdc++.6.0.9.tbd 的下载地址：[libstdc++.6.0.9](https://github.com/MaricleZhang/libstdc-.6.0.9.tbd.git)
<!-- more -->
#### 真机

```
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk/usr/lib/

```

#### 模拟器
```
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/CoreSimulator/Profiles/Runtimes/iOS.simruntime/Contents/Resources/RuntimeRoot/usr/lib/

/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk/usr/lib/
```

###  Multiple commands produce'/Users/user/Library/Developer/Xcode/DerivedData...

- #####方法1
xcode 10 默认是新的编译模式，选择File > Workspace Settings > Build System > Legacy Build System.，改变为之前的编译模式Legacy Build system。

- #####方法2
Open target -> Build phase > Copy Bundle Resource  删除output files 中的脚本，如果存在 info.plist 也一起删除。

![delete_output.png](https://upload-images.jianshu.io/upload_images/2403444-dc2bcdb459d1b5d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




