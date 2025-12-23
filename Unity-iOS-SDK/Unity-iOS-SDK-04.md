#### `Unity 游戏开发之`  `iOS SDK` 接入（四） 

接上文，我们上文所说的`NativePlugin`要放在`UnityFramework`中的，`UnityFramewrok`中如何调用`Unity-iPhone`中`SDK`相关的函数？

在游戏开发过程中常用的策略模式，我们定义好流程以及接口，具体的实现由不同的功能实现。

1. 添加一个`NativePluginProtocol.h`

```objective-c
#ifndef NativePluginProtocol_h
#define NativePluginProtocol_h

@protocol NativePluginProtocol <NSObject>

- (void) callSDK:(NSString*)type withParam:(NSString*)param;

- (NSString*) callSDKRs:(NSString*)type withParam:(NSString*)param;

@end
```

2. 在`NativePlugin` 中添加以下内容：

```objective-c
@interface NativePlugin : NSObject

+ (void)setNativePluginProtocol:(id<NativePluginProtocol>)protocol;

@end
    
static id<NativePluginProtocol> _protocol;

@implementation NativePlugin

+ (void)setNativePluginProtocol:(id<NativePluginProtocol>)protocol {
    _protocol = protocol;
}

char* StringCopy(const char* string) {
    if (string == NULL) return NULL;
    char* res = (char*)malloc(strlen(string) + 1);
    strcpy(res, string);
    return res;
}

void CallNativeFunc(const char *type, const char *param) {
    if (_protocol == NULL) return;

    NSString* t = [NSString stringWithFormat:@"%s", type];
    NSString* p = [NSString stringWithFormat:@"%s", param];
    [_protocol callSDK:t withParam:p];
}

const char* CallNativeFuncRs(char *type, char *param) {
    if (_protocol == NULL) return NULL;

    NSString* t = [NSString stringWithFormat:@"%s", type];
    NSString* p = [NSString stringWithFormat:@"%s", param];
    return StringCopy([[_protocol callSDKRs:t withParam:p] UTF8String]);
}
@end
```

即`NativePlugin`不再具体的实现相关的功能，而是调用遵循`NativePluginProtocol`协议的实例。

3. 实现具体的`NativePluginProtocol`实例`MyNativePlugin`，添加到`Unity-iPhone`中：

```objective-c
#import "NativePluginProtocol.h"
@interface MyNativePlugin : NSObject<NativePluginProtocol>

+ (MyNativePlugin *) sharedInstance;

@end

@implementation MyNativePlugin

+ (MyNativePlugin *) sharedInstance {
    static MyNativePlugin *_sharedInstance = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _sharedInstance = [[MyNativePlugin alloc] init];
    });
    return _sharedInstance;
}

@end
```

4. 将`MyNativePlugin`实例设置给`PluginNative`

这个时候需要注意，`MyNativePlugin`是在`Unity-iPhone`中，但是`NativePlugin`在`UnityFramewrok`中，而上文中我们知道`UnityFramework`是动态加载执行的，所以在`Unity-iPhone`中不能直接调用`NativePlugin`的`setNativePluginProtocol`方法，而是要通过反射去调用，如下：

```objective-c
Class cls = NSClassFromString(@"NativePlugin");
if (cls && [cls respondsToSelector:@selector(setNativePluginProtocol:)]) {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Warc-performSelector-leaks"
    [cls performSelector:@selector(setNativePluginProtocol:)
               withObject:[MyNativePlugin sharedInstance]];
#pragma clang diagnostic pop
}
```

至此我们完成了从`UnityFramewrok`中如何调用`Unity-iPhone`中`SDK`相关的函数。

