#### `Unity 游戏开发之`  `iOS SDK` 接入（二） 

接上篇，`SDk`的接入并不是简单`C#`与`Objective-c`的交互，很多`SDK`都涉及到生命周期，我们知道`iOS`的生命周期代理实现都在`UIApplicationDelegate`实现。

`Unity`也会实现，具体实现的地方在`UnityAppController.m`文件中。

但是我们不能直接在`UnityAppController`中直接去调用`SDK`的生命周期函数，这样不符合我们写代码的原则，即侵入式写代码，也不方便我们后续维护，每次导出可能导致覆盖之类。

`Unity`给我们提供了一个途径，继承`UnityAppController`，然后用自己的`MyAppController`替换`UnityAppController`。

1. 创建`MyAppController.h`

```objective-c
#import "UnityAppController.h"
@interface MyAppController : UnityAppController
@end
```

2. 创建`MyAppController.m`

```objective-c
#import "unityAppController.h"
#import "MyAppController.h"

/* 用 MyAppController 替换 UnityAppController */
IMPL_APP_CONTROLLER_SUBCLASS(MyAppController)

@implement MyAppController

- (BOOL)application:(UIApplication*)application willFinishLaunchingWithOptions:(NSDictionary*)launchOptions
{
    /*
    这里调用 SDK 的生命周期函数
    */
    return [super application:application willFinishLaunchingWithOptions:launchOptions];
}

@end
```

到这里，我们结合上篇，就能实现完整的`iOS SDK` 接入相关的工作了。 