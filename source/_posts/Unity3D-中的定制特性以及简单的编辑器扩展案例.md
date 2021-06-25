---
title: Unity3D 中的定制特性以及简单的编辑器扩展案例
date: 2021-06-25 17:21:02
tags: 
- Unity
- C# 
- 编辑器扩展 
categories:
- Unity 
keyword: 'Unity编辑器扩展' 
description: 简单的unity编辑器扩展原理说明和案例
cover : https://z3.ax1x.com/2021/06/25/R1UX0x.jpg
---

`陈嘉栋老师的《Unity3D脚本编程》读书笔记~
外加一些网上Unity编辑器扩展说明`

---

[Unity 编译说明](https://blog.csdn.net/u011926026/article/details/53982101?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-6.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-6.channel_param)：

对于大型项目来说，这确实是大家经常遇到的情况。一般来说，`Unity Editor`会按照脚本的依赖关系编译代码，其主要分为以下四个步骤：

+ 编译`Standard Assets`、`Pro Standard Assets`和`Plugins`文件夹中的`Runtime Script`；
+ 编译以上三个文件夹中`Editor`文件夹下的`Script`；
+ 编译项目中所有剩余的`Runtime Script`（Editor文件夹以外Script；
+ 编译剩余`Script`（即Editor文件夹中Script）。

知道了Unity编辑器的脚本编译特性后，我们则建议研发团队可以将一些长时间不需要改动的脚本代码（比如各种插件代码）放入到`Standard Assets`、`Pro Standard Assets`或`Plugins`文件夹中，这样这些代码只需要编译一次，后续的时间就都能节省下来。

---

## 初识特性---Attribute

定义一个类型或者类型内部的成员的时候，我们往往会通过`public` ，`private`等特性去描述它们

**我们是否可以自己去定义特性呢？**

比如指定某个类型是用来生成Editor的，某个类型是可以被序列化的，某个方法是依赖某个类型的。

实现这些自定义的特性也需要一些前提条件：`编译器必须理解这些特性才能对这些特性作出正确的反应。`

但编译器代码门槛太高，C#提供了一种机制支持开发者们的自定义特性，同时重要的一点是它们在程序设计时和运行时都能发挥作用。

本质上自定义特性只是为了某个目标元素提供了和一些额外附加信息的关联。编译器会在托管模块的元数据额外嵌入这些信息。**事实上，大多数特性对编译器并无意义，编译器只是机械地检测源码中的特性，并生成对应的元数据。**

---

## DllImport特性

`DllImport`特性适用于方法，告诉运行时的程序：该方法位于指定的DLL的非托管代码中。（DllImporter是在DLL为非托管代码时才会使用的），在Unity3D中引用外部DLL的一个主要目的是方便集成一些外部插件，以便调用现有的外部动态库链接。

关于托管DLL和非托管DLL 解释：

[https://blog.csdn.net/TangLinCSDN/article/details/45719851](https://blog.csdn.net/TangLinCSDN/article/details/45719851)

这里目前俺用不到，略。

## Serializable特性

`DllImporter`主要应用于方法，`Serializable`特性应用于类型。使用`Serializable`特性来告诉格式化程序，一个类型的字段可以进行序列化和反序列化的操作。

```csharp
using System;
using System.Collection;

[Serializable]//使用[Serializable]告诉编译器，该类实例可被序列化
public class serializableClass{
	public string name;
	public int age;
	public bool isHero;
}
```
以下代码可以更加深刻的体会序列化：被序列化的类的`name =xiba` ，在运行过程中改变它的值，之后再反序列化出来，得到的name依旧为 `xiba` ;

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.Runtime.Serialization.Formatters.Binary;
using UnityEngine;

[Serializable]
public class SerializableTest
{
    public string name;

    public SerializableTest(string name)
    {
        this.name = name;
    }

}
public class SerializableTest1 : MonoBehaviour
{
    public SerializableTest serializableTest;
    void Start()
    {
        this.serializableTest = new SerializableTest("xiba");
    }

    private void OnGUI()
    {
        if (GUI.Button(new Rect(10, 10, 150, 100), "Serializable"))
        { 
            string filename= Application.dataPath + "/Resources/serializable.dat";
            Stream fStream = new FileStream(filename, FileMode.Create, FileAccess.ReadWrite);

            //创建二进制序列化器
            BinaryFormatter binFormat = new BinaryFormatter();
            binFormat.Serialize(fStream, this.serializableTest);
            fStream.Close();

            this.serializableTest.name = "猪皮骚";
            Debug.Log("the class name is :" + this.serializableTest.name);
        }

        if (GUI.Button(new Rect(300, 10, 150, 100), "Deserialize"))
        { 
            string filename= Application.dataPath + "/Resources/serializable.dat";
            Stream fStream = new FileStream(filename, FileMode.Open, FileAccess.Read);
            BinaryFormatter binaryFormatter = new BinaryFormatter();
            this.serializableTest = binaryFormatter.Deserialize(fStream) as SerializableTest;

            fStream.Close();
            Debug.Log("after Deserilizable the class name is :"+this.serializableTest.name);
        }
    }
}

```

## 定制特性到底是谁

前两个小节介绍了两个定义在基础类库中的，比较有代表性的特性，但特性自己到底是如何定义的？

简单的说，定制特性其实是一个类型的实例。Mono之所以能跨平台原因便是其符合”公共语言规范（Common Language Specification (CLS)）“的要求，根据公共语言规范定制特性类必须直接或者间接从公共抽象类`System.Attribute`派生。 

---

## Unity3D 中提供的常用定制特性

现在知道了C#中的特性是从`System.Attribute` 派生而来的一个类的实例。同样，Unity中C#的游戏脚本也有派生自`System.Attribute`中的特性。

Unity将特性分别定义在了`UnityEngine`和`UnityEditor`这两个命名空间中。 

假设你是Unity的设计人员，想要剔红自定义编辑器菜单的功能，那么应该这样：

```csharp

namespace UnityEngine
{
	public class AddComponentMenu:System.Attribute
	{
		public AddComponentMenu(string componentName)
		{
			....
		}
	}
}
```

当然，Unity3D 是真实存在AddComponentMenu这个特性的。这个AddComponentMenu属性允许你在”Component“菜单中放置一个脚本，无需看这个脚本在哪：

```csharp
[AddComponentMenu("Transform/xiba ")]

public Class Test
{
}
```

首先需要说明的是`UnityEditor`类，这是一个编辑器类，想要使用需要将其放在`Asset/Editor`文件夹下。使用脚本时，需要`using UnityEditor`引用；

`MenuItem `特性允许你添加菜单项和检视面板上下文菜单，**并且MenuItem特性会将所有静态方法转化为菜单命令**

```csharp
[MenuItem("Mymenu/Dosomething")]
static void Dosomthing()
{
	Debug.Log("Do Something");
}
```

详细用法还有很多，这里暂不赘述。

---

## 定义自己的定制特性类

```csharp
//显式指出CustomAttribute继承了System.Attribute.这样Customattribute便符合CLS 关于定制特性的要求规范了

//告诉编译器CustomAttibute的使用范围，并将其应用于引用类型和方法
[AttributeUsage(AttributeTargets.Class,Inherited =false )]
public class CustomAttribute :System.Attribute
{
    string name;
    public CustomAttribute(string name)
    {
        this.name = name;
    }
} 
```

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System;

[CustomAttribute("saopizhu")]
public class SaoClass : MonoBehaviour
{
    public static void Main()
    {
        SaoClass a = new SaoClass();
    }

    //判断是否应用CustomAttribute特性
    private void Start()
    {
        if (this.GetType().IsDefined(typeof(CustomAttribute), false))
        {
            Debug.Log("已经产生关联");
        }
    }
}
```

---

## 亲手扩展Unity3D 的编辑器

首先新建一个脚本：

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

//需要继承Editor类
public class EditorTest : MonoBehaviour
{
    public Rect mRectValue;
    public Texture texture;
}
```

将其挂载在场景物体中，为了能动态编辑它，需要在编辑器中增加对应功能:

```csharp
using UnityEngine;
using UnityEditor;


[CustomEditor(typeof(EditorTest))]
[ExecuteInEditMode]
public class MyEditor : Editor
{
    public override void OnInspectorGUI()
    {
        //target :公共对象目标   
        //获取EditorTest类实例
        EditorTest editorTest = (EditorTest)target;

        //绘制一个窗口
        editorTest.mRectValue = EditorGUILayout.RectField("窗口坐标", editorTest.mRectValue);

        //绘制一个贴图槽
        editorTest.texture = EditorGUILayout.ObjectField("增加一个贴图", editorTest.texture, typeof(Texture), true) as Texture;
    }

```
![](https://img-blog.csdnimg.cn/20200923153151975.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

为了满足使用的需求，Unity支持通过[ExecuteInEditMode]或[ExecuteAlways]两种参数使脚本在Play Mode以外的状态下被执行，[ExecuteEditMode]支持脚本在Edit Mode下运行，[ExecuteAlways]是在Unity2018.3及以后的版本新加入的功能，能够支持脚本一直运行。（ps:由于[ExecuteInEditMode] 并没有考虑Prefab Mode，严格意义上讲Prefab Mode也属于Edit Mode，所以这个功能会逐渐被Unity弃用，最后应该会被[ExecuteAlways]所替代）

除了能为Unity3D中的编辑器的控件进行绘制，还可以自定义创建新的窗口，并设置它的窗口布局。

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEditor;

public class EditorWindowTest : EditorWindow
{
    [MenuItem("MyMenu/OpenWindow")]
    static void OpenWindow()
    {
        Rect wr = new Rect(0,0,500,500);
        EditorWindowTest window = (EditorWindowTest)EditorWindow.GetWindowWithRect(typeof(EditorWindowTest), wr, true, "测试创建窗口");
        window.Show();
        
    }
    
    private string text;
    private Texture texture;

    public void Awake()
    {
        texture = Resources.Load("1") as Texture;
    }

    private void OnGUI()
    {
        if (GUILayout.Button("打开通知", GUILayout.Width(200)))
        {
            //打开一个通知栏
            this.ShowNotification(new GUIContent("This is a Notification"));
        }

        if (GUILayout.Button("关闭通知栏", GUILayout.Width(200)))
        {
            //关闭通知栏
            this.RemoveNotification();
        }

        //文本框显示鼠标在窗口的位置
        EditorGUILayout.LabelField("鼠标在窗口的位置", Event.current.mousePosition.ToString());

        //选择贴图
        texture = EditorGUILayout.ObjectField("添加贴图", texture, typeof(Texture), true) as Texture;

        if (GUILayout.Button("关闭窗口", GUILayout.Width(200)))
        {
            this.Close();
        }
    }

    private void OnFocus()
    {
        Debug.Log("当窗口获取焦点时调用一次");
    }

    private void OnLostFocus()
    {
        Debug.Log("当窗口丢失焦点的时调用一次");
    }

    private void OnHierarchyChange()
    {
        Debug.Log("当Hierarchy视图中的任何对象发生改变的时候调用一次");
    }

    private void OnProjectChange()
    {
        Debug.Log("当Project 视图中的资源发生改变时调用一次");
    }


    private void OnInspectorUpdate()
    {
        //这里开启窗口的重绘，不然窗口信息不会刷新
        this.Repaint();
    }

    private void OnSelectionChange()
    {
        //当窗口处于开启状态，并且在Hirearchy视图中选择某游戏对象时调用
        foreach (var item in Selection.transforms)
        {
            Debug.Log("当前选中对象名" + item.name);
        }
    }
    private void OnDestroy()
    {
        Debug.Log("窗口关闭");
    }
}

```
![](https://img-blog.csdnimg.cn/20200923153446504.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)
 
 在Unity3D的编辑器区，还有一个十分重要的区域便是Scene区域。Scene同样可以扩展

实际上，对于`Scene`区域的扩展要基于一个基础，那便是对象。那就是作为开发者，必须在`Hierarchy`视图中选择一个当前区域的`Scene`的对象才行。`Hierarchy`视图中选择不同的游戏对象，也可以有不一样的视图。

下面新建一个脚本：

```csharp
using UnityEngine;

public class SceneTest : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        
    }

    // Update is called once per frame
    void Update()
    {
        
    }
}
```
将SceneTest挂载在一个物体上，为了能在Scene视图中能编辑游戏物体，还需要另外一个脚本来实现这个功能：

```csharp
using UnityEditor;
using UnityEngine;

[CustomEditor(typeof(SceneTest))]
public class SceneEditor : Editor
{
    private void OnSceneGUI()
    {
        //得到test脚本的对象
        SceneTest test = (SceneTest)target;

        //绘制文本框
        Handles.Label(test.transform.position+Vector3.up*2, test.transform.name + test.transform.position.ToString());

        //开始绘制GUI
        Handles.BeginGUI();

        //规定GUI显示区域
        GUILayout.BeginArea(new Rect(100,100,200,200));

        //GUI绘制一个按钮
        if (GUILayout.Button("这是一个按钮"))
        {
            Debug.Log("SceneTest ");
        }

        //GUI 绘制文本框
        GUILayout.Label("我在编辑Scene视图");
        GUILayout.EndArea();
        Handles.EndGUI();
    }
   
}

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200923162728916.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)
