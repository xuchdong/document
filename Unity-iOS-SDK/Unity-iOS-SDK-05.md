#### `Unity 游戏开发之`  `iOS SDK` 接入（五） 

接上文，我们完成了从`UnityFramewrok`中如何调用`Unity-iPhone`中`SDK`相关的函数的功能，接下来我们就要实现`SDK`里面的生命周期函数接入。

这里分别用到了一下知识点：

- `Objective-c` 类扩展；
- `method_exchangeImplementations`

即我们扩展原来的`UnityAppController`，然后用新的方法替换旧的方法，具体的实现如下：

1. 创建`UnityAppController+Native.h`

```objective-c
#include "UnityAppController.h"

@interface UnityAppController (Native)

@end
```

2. 创建`UnityAppController+Native.m`

```objective-c
#import "UnityAppController+Native.h"

@implementation UnityAppController (Native)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        SEL finishLaunchingOriginal = @selector(application:didFinishLaunchingWithOptions:);
        SEL finishLaunchingSwizzled = @selector(__application:didFinishLaunchingWithOptions:);
        [UnityAppController swizzleSelector:finishLaunchingSwizzled original:finishLaunchingOriginal];
    });
}

+ (BOOL)swizzleSelector:(SEL)swizzledSelector original:(SEL)originalSelector {
    Method origMethod = class_getInstanceMethod(self, originalSelector);
    Method altMethod = class_getInstanceMethod(self, swizzledSelector);
    if (!altMethod) {
        return NO;
    }
    
    class_addMethod(self,
                    originalSelector,
                    class_getMethodImplementation(self, originalSelector),
                    method_getTypeEncoding(origMethod));
    class_addMethod(self,
                    swizzledSelector,
                    class_getMethodImplementation(self, swizzledSelector),
                    method_getTypeEncoding(altMethod));
    
    method_exchangeImplementations(class_getInstanceMethod(self, originalSelector), class_getInstanceMethod(self, swizzledSelector));
    return YES;
}

- (BOOL)__application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)options {
    NSLog(@"swizzle => %@", @"__application:didFinishLaunchingWithOptions:");
    /*
    这里调用 SDK 的生命周期函数
    */
    BOOL result = [self __application:application didFinishLaunchingWithOptions:options];
    return result;
}

@end
```

需要其他的生命周期函数直接自行添加即可。

至此，完整的`SDK`接入流程完结。

