#### `Unity 游戏开发之`  `iOS SDK` 接入（三） 

继续接上文，使用`Unity`导出`iOS`工程之后，大家可以看到跟`Unity`相关的内容都放到了`UnityFramework`中，所以我们上文相关的接入以及`SDK`的引入就必须放到`UnityFramewrok`中，然后再在`Unity-iPhone main.m`中动态加载执行，如下：

```objective-c
#include <UnityFramework/UnityFramework.h>

UnityFramework* UnityFrameworkLoad()
{
    NSString* bundlePath = nil;
    bundlePath = [[NSBundle mainBundle] bundlePath];
    bundlePath = [bundlePath stringByAppendingString: @"/Frameworks/UnityFramework.framework"];

    NSBundle* bundle = [NSBundle bundleWithPath: bundlePath];
    if ([bundle isLoaded] == false) [bundle load];

    UnityFramework* ufw = [bundle.principalClass getInstance];
    if (![ufw appController])
    {
        // unity is not initialized
        [ufw setExecuteHeader: &_mh_execute_header];
    }
    return ufw;
}

int main(int argc, char* argv[])
{
    @autoreleasepool
    {
        id ufw = UnityFrameworkLoad();
        [ufw runUIApplicationMainWithArgc: argc argv: argv];
        return 0;
    }
}

```

 所以我们上文相关的接入以及`SDK`的引入就必须放到`UnityFramewrok`中。

当然上述接入完全没有问题。

但是当我们一个项目需要接入多个`SDK`的时候，或者我们有多个项目想公用一个中间层的模版，然后每个项目接入自己的`SDK`，为了接入或者管理方便，又或者`SDK`跟`UnityFramework`冲突的时候。

我们就必须将`SDK`相关的接入放到`Unity-iPhone`中。

这个时候就涉及到2个问题：

1. 我们上文所说的`NativePlugin`要放在`UnityFramework`中的，`UnityFramewrok`中如何调用`Unity-iPhone`中`SDK`相关的函数？
2. `SDK`里面的生命周期函数又怎么接入？



