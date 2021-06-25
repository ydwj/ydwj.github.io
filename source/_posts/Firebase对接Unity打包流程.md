---
title: Firebase对接Unity打包流程
date: 2021-06-25 16:58:02
tags: 
- Unity
- Android
- Firebase
categories:
- Unity 
keyword: 'Firebase'
description: FireBase分析工具对接Unity 2019.4.18 ，安卓打包流程
cover : https://z3.ax1x.com/2021/06/25/R1YdmV.png
---

官方准备工作教程：
[https://firebase.google.com/docs/unity/setup](https://firebase.google.com/docs/unity/setup)

---

## 准备工作

创建好分析页面，下载官方提供的`firebase_unity_sdk_7.1.0`包，根据自己.net 版本选择（2019一般都是.net4x)unity.package，这里选择`dotnet4/ FirebaseAnalytics.unitypackage`。


## 下载并导入firebase配置文件

配置文件直接扔到`Asset`目录里，下载页面在这：

![](https://img-blog.csdnimg.cn/20210422164831829.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

如果之后报错 "Database URL not set in the Firebase config "，就需要手动给配置文件添加链接

![](https://img-blog.csdnimg.cn/2021042216464098.png#pic_center)

链接在这获得：

![](https://img-blog.csdnimg.cn/20210422164930493.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

新建一个脚本挂在场景中，代码如下：

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Firebase;
using Firebase.Analytics;
public class FirebaseTest : MonoBehaviour
{
    void Start()
    {
        Firebase.FirebaseApp.CheckAndFixDependenciesAsync().ContinueWith(task => {
            var dependencyStatus = task.Result;
            if (dependencyStatus == Firebase.DependencyStatus.Available)
            {
                //安卓上会自动初始化
                // FirebaseAnalytics.SetAnalyticsCollectionEnabled(true); 
                //此调用仅确保fireBase 本身已经初始化
                var app = Firebase.FirebaseApp.DefaultInstance;
            }
            else
            {
                UnityEngine.Debug.LogError(System.String.Format(
                  "Could not resolve all Firebase dependencies: {0}", dependencyStatus));
                // Firebase Unity SDK is not safe to use here.
            }
        });
    }
}
```

## 打包查看信息：

打包后在移动端打开app，就可以在后台看到你的信息了：

![](https://img-blog.csdnimg.cn/20210422162443241.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

后台的报告一般24小时内才出一次，为了实时成测试事件是否生效，我们需要在Unity中下载Android Logcat，原工程package Manager可能无法联网报错，新建一个工程下载，然后将json文件注册信息更改强制下载就好了。

![](https://img-blog.csdnimg.cn/20210422162929569.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

## 自定义事件

然后点开window/ Analysis/Android logcat窗口，你会发现左下方显示DisConnected，这里我们需要把安装了App的安卓机连接上电脑，**并开启开发者模式权限。**

点击Tools/open Terminal。

![](https://img-blog.csdnimg.cn/20210422163355390.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

这里就看到控制台弹出来了，先检测是否正常连接，输入adb shell，显示已经连上设备了

![](https://img-blog.csdnimg.cn/20210422163516710.png#pic_center)

接下来找到自己unity 工程中的pakage name ,在依次在控制台输入：

```csharp
.\adb shell setprop debug.firebase.analytics.app XXXXXXXXXX（这里填自己的package name）

.\adb shell setprop log.tag.FA VERBOSE

.\adb shell setprop log.tag.FA_SVC VERBOSE
```
正常的话就可以在后台看到接近实时的事件触发数据辣！！比如在游戏开始的方法中添加这句话

```csharp
FirebaseAnalytics.LogEvent(FirebaseAnalytics.EventLevelStart);
```

之后就能在Firebase后台中看到触发的时间了：

![](https://img-blog.csdnimg.cn/2021042216430711.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

详细的事件定义方法可以查看官方文档：[https://firebase.google.com/docs/reference/unity](https://firebase.google.com/docs/reference/unity)
