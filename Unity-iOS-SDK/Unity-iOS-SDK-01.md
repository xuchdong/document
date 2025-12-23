#### `Unity 游戏开发之`  `iOS SDK` 接入（一） 

本教程为系列教程，指导从零到完整接入`iOS SDK`，本文只会教你怎么接入，并不会讲具体的原理，分为以下几部分：

- `C#` 与 `Objective-C` 交互
- `SDK`生命周期接入
- 进阶`UnityFramework` 和 `Unity iPhone`交互
- 进阶`Unity iPhone SDK` 生命周期接入

要接`SDK`, 就必须实现`C#`与`Objective-c` 交互，本篇就讲解这一部分，[点击这里查看`Unity`教程](https://docs.unity3d.com/2022.3/Documentation/Manual/ios-native-plugin-create.html)。

1. 创建`C#`文件，定义拓展方法

```c#
public class NativePlugin
{
#if UNITY_IOS
        [DllImport("__Internal")]
        public static extern void CallNativeFunc(string type, string param);
    	[DllImport("__Internal")]
    	public static extern string CallNativeFuncRs(string type, string param);
#endif
}
```

2. 然后在`Assets/Plugins/iOS`中创建一个`NativePlugin.m`文件，文件内容如下：

```objective-c
#import <Foundation/Foundation.h>

void CallNativeFunc(const char *type, const char *param)
{
}

const char* CallNativeFuncRs(char *type, char *param)
{
    return null;
}
```

3. 在`C#`代码中调用具体函数即可：

```c#
class SDKMgr
{
    public void Login()
    {
        NativePlugin.CallNativeFunc("login", "");
    }
}
```

至此就完成了`C#`和`Objective-c`的交互，如果`SDK`没有跟生命周期相关的接入，到这里就结束了，否则就请继续往下看。
