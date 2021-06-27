---
title: Unity编辑器扩展：使用xNode制作自己的可视化工具（2）
date: 2021-06-27 19:12:50
tags: 
- Unity
- C# 
- xNode
categories:
- Unity 
keyword: 'xNode' 
description: 可视化剧情制作工具
cover: https://z3.ax1x.com/2021/06/27/RYg3zd.jpg
---

大家好，我是炎拳。

上一篇文章：[使用Xnode制作可视化剧本编辑插件（1）](https://zhuanlan.zhihu.com/p/342322276)，简单的展示了我使用xNode 制作的小工具，这一篇就来分享下Xnode的用法，以及在开发过程中收获的一些编辑器扩展的知识。

在使用xNode 开发前，还是需要略微了解一些Unity3D中的定制特性：例如当你不想在面板上显示你的公开字段，你可以在字段上添加[HideInInspector]，它的功能是在Inspector面板中隐藏public属性，没有序列化的功能。

![](1.png)

我简单写了一篇读书笔记，给同样是新手的朋友做个参考：
[Unity3D 中的定制特性以及简单的编辑器扩展案例](https://auniquepig.com/2021/06/25/Unity3D-%E4%B8%AD%E7%9A%84%E5%AE%9A%E5%88%B6%E7%89%B9%E6%80%A7%E4%BB%A5%E5%8F%8A%E7%AE%80%E5%8D%95%E7%9A%84%E7%BC%96%E8%BE%91%E5%99%A8%E6%89%A9%E5%B1%95%E6%A1%88%E4%BE%8B/)

引用书中的一段话简单解释就是：`定制特性其实是一个类型的实例。Mono之所以能跨平台原因便是其符合”公共语言规范（Common Language Specification (CLS)）“的要求，根据公共语言规范定制特性类必须直接或者间接从公共抽象类System.Attribute派生。`

![F12【HideInInspector】](2.png)

知道了C#中的特性是从System,Attribute 派生而来的一个类的实例。同样，Unity中C#的游戏脚本也有派生自system.Attribute中的特性，Xnode 中的定制特性同样如此。

---

## xNode的用途和下载途径

再简单的介绍下xNode，它是一个完全开源的免费插件，官方文档的介绍很吸引人：

> xNode是超级用户友好的，直观的，它将帮助您快速了解节点图。由于占用的空间很小，它非常适合作为自定义状态机、对话系统、决策树的基础架构。

![](3.png)

xNode官方文档也较为齐全，节省了我很多学习时间，在Github上获取最新的xNode工程，扔到你的Unity工程下就可以使用了，地址在这：

[Siccity/xNode](https://github.com/Siccity/xNode)
​

## xNode的核心概念

援引自官方文档，一个xNode项目由这三个部分组成：

### Graph

每个xNode项目都从创建Graph开始，Graph可以看成是你图形化界面的入口，新建一个自定义Graph也很简单，以我的工具为例：

```csharp
using XNode;
//通过Unity菜单来创建自定义Graph
 [CreateAssetMenu(fileName = "DialogueGraph")]
    public class DialogueGraph : NodeGraph
    {
    }
```
然后通过菜单生成新的Graph，Graph中包含了项目中Node列表的信息，可以在面板上直接进行操作：

![](4.png)

![](5.png)

接着看看自定义Graph的父类NodeGraph，可以发现它也同样是继承了Unity ScriptableObject的抽象子类，并且提供了丰富的Node相关虚方法：

![](6.png)

对ScriptableObject感兴趣的朋友可以自行搜索资料，不懂也没关系，你只要知道它的用法就好：在编辑器会话期间，将数据保存为项目中的资源，这样方便在项目运行的时候使用。

### Node

自定义Graph继承自NodeGraph，自定义的Node继承自Node，一个标准的Node包含单个输入端口和输出端口，通过对任意公开字段添加[Input]或[Output]属性生成：

```csharp
public class SimpleNode : Node {
    [Input] public float value;
    [Output] public float result;
}
```
Node同样是一个继承自ScriptableObject的抽象类，从Node派生出的任何自定义Node子类，都是有效节点，同时会默认添加到Graph的上下文菜单中，如图所示：

![](7.png)

也可以自定义Node的样式和名称

```csharp

  [CreateNodeMenu("SimpleNode1111111")]
    [NodeWidth(400)]
    [NodeTint(73, 236, 209)]//Node颜色
    public class SimpleNode : Node
    {
        [Input] public float value;
        [Output] public float result;
    }
```

![](8.png)

### Port

端口（port）是节点（Node）之间通信的大门，他们既可以是输入，也可以是输出。一个Node可以包含多个port。

一般只要Unity能序列化的类型都可以作为端口，当然我们可能需要在Node上动态添加或者删除端口，这里就需要去定义单独Node上端口的渲染了，之后会详细说。

这里放个端口的常见用法（在Node类中）：获取其连接的node
```csharp

// Get the connected node.
NodePort otherPort = GetOutputPort("myOutput").Connection
if (otherPort != null) {
  MyNode nextNode = otherPort.node as MyNode;
}
```

---

## StoryEditor制作思路 & 魔改xNode

介绍完基础概念，接下来分享下制作这个剧本编辑插件的过程和一些有意思的问题：

首先，我的需求是能快速制作一段类似《星露谷物语》中过场演出，虽然玩家触发剧情的时间和方式都比较随机，但还是每段剧情本身还是遵循传统的树状叙事结构，通过玩家的选择来触发剧情：

![树状叙事结构](9.png)

但在《星露谷物语》中，玩家和NPC对话的过程会时不时穿插一些人物的动作和表情，来丰富演出效果，伴随的可能有对话UI短暂消失等待人物动画播放完成，终止UI点击事件等需求。`所以我需要为每一段对话提供一些功能选项，方便快速配置这些功能。`

如何定义每段对话也很重要，Xnode Graph所采用的渲染方法来自Unity的EditorWindow类，移动鼠标时会强制调用OnGUI（）进行刷新，所以Graph中的Node越多，刷新次数就越多，`不少开发者也遇到了大量Node存在时刷新卡顿的问题`。如果每段对话都使用一个Node，那200句话可能就要新建一个Graph，并且修改起来十分麻烦，所以最终我希望能在一个Node中尽可能的添加对话，所以对话Node应该是这样：

![](10.png)

功能如下：对话依次向下播放，根据对话类型，对话框展示出不同的效果。如果一段对话过长或者遇到了选择分支，也可以更改对话类型来主动生成一个输出端口进行跳转。

思路清晰了，但问题是怎么在Xnode上渲染出这样的界面呢？xNode提供了解决方案，首先是更加深入的自定义Node外观，官方提供了一个简单的加法器案例：

```csharp
// SimpleNode.cs

public class SimpleNode : Node{
    public int a;
    public int b;
    [Output] public int sum;


//获取对应端口的值    
public override object GetValue(NodePort port) {
        return GetSum();
    }

    public float GetSum() {
        return a + b;
    }
}
```
```csharp
// Editor/SimpleNodeEditor.cs

//用于关联你需要自定义的Node
[CustomNodeEditor(typeof(SimpleNode))]

public class SimpleNodeEditor : NodeEditor {
    private SimpleNode simpleNode;

    public override void OnBodyGUI() {
        if (simpleNode == null) simpleNode = target as SimpleNode;

        // Update serialized object's representation
        serializedObject.Update();

        NodeEditorGUILayout.PropertyField(serializedObject.FindProperty("a"));
        NodeEditorGUILayout.PropertyField(serializedObject.FindProperty("b"));
        UnityEditor.EditorGUILayout.LabelField("The value is " + simpleNode.GetSum());
        NodeEditorGUILayout.PropertyField(serializedObject.FindProperty("sum"));

        // Apply property modifications
        serializedObject.ApplyModifiedProperties();
    }
}

```
![](11.png)

自定义内容的问题貌似解决了，照葫芦画瓢就好，但如何让xNode根据一个对话列表来生成多个端口呢？官方也提供了一个实验性质的方法-Dynamic Port List：

```csharp
public class SimpleNode : Node {
    [Output(dynamicPortList = true)] public float[] myArray;
}

```
![](12.png)

看起来图中的节点作为剧本中的选择项不错，但作为对话节点还是过于混乱了，我并不需要每个对话元素都显示一个端口。

好在xNode 给我们提供了自定义绘制列表的方法，实际上最新版本的xNode中已经集成了部分Odin的功能（不知道它怎么打通关节的哈哈），Odin同样是一款绘制Unity界面非常好用的工具，比方说Unity和C#并未支持字典的序列化，在Odin的支持下我们可以很轻松的让字典在Unity Inspector面板中显示并修改：

![使用Odin序列化将字典显示在Inspector面板上，Odin直接完成了对应元素的排版，太舒服了](13.png)

在xNode的界面同样支持Odin，但我对Odin的了解不多，还是没法达成想要的效果，最终还是选择使用Unity自身的编辑器拓展方法，原因也很简单，xNode本身绘制列表的方法也是使用Unity ReorderableList（可重排序列表），其中元素可以自由拖动，非常方便，感兴趣的朋友可以看看这篇文章：

[Unity编辑器拓展之二：ReorderableList可重新排序的列表框（复杂使用）_静风霁-CSDN博客](https://blog.csdn.net/qq_26999509/article/details/77801852)

最终我做出来的效果如下：

![](14.gif)

代码还挺长的，感兴趣的朋友可以直接去上一篇文章中下载工程看看，只提一点很重要的，xNode本身并未考虑到我这样奇葩的需求，一开始我发现并不能通过修改类型自由生成Port，最后在老师的帮助下发现了原因，稍微改了下DynamicPortList这个方法，有需要的朋友自取:

```csharp

   public static void DynamicPortList(string fieldName, Type type, SerializedObject serializedObject, XNode.NodePort.IO io, XNode.Node.ConnectionType connectionType = XNode.Node.ConnectionType.Multiple, XNode.Node.TypeConstraint typeConstraint = XNode.Node.TypeConstraint.None, Action<ReorderableList> onCreation=null, bool forceReset=false) {
            XNode.Node node = serializedObject.targetObject as XNode.Node;

            var indexedPorts = node.DynamicPorts.Select(x => {
                string[] split = x.fieldName.Split(' ');
                if (split != null && split.Length == 2 && split[0] == fieldName) {
                    int i = -1;
                    if (int.TryParse(split[1], out i)) {
                        return new { index = i, port = x };
                    }
                }
                return new { index = -1, port = (XNode.NodePort) null };
            }).Where(x => x.port != null);
            List<XNode.NodePort> dynamicPorts = indexedPorts.OrderBy(x => x.index).Select(x => x.port).ToList();

            node.UpdatePorts();


            //在这里发现DynamPortList本身会将List缓存到rlc字典里，并且之后每次刷新，不会对list进行更改
            ReorderableList list = null;
            Dictionary<string, ReorderableList> rlc;
            if (reorderableListCache.TryGetValue(serializedObject.targetObject, out rlc)) {
                if (!rlc.TryGetValue(fieldName, out list)) list = null;
            }

            //重写判定，强制对其进行更改
            // If a ReorderableList isn't cached for this array, do so.
            if (list == null || forceReset) {
                SerializedProperty arrayData = serializedObject.FindProperty(fieldName);
                list = CreateReorderableList(fieldName, dynamicPorts, arrayData, type, serializedObject, io, connectionType, typeConstraint, onCreation);
                if (!reorderableListCache.ContainsKey(serializedObject.targetObject))
                {
                    reorderableListCache.Add(serializedObject.targetObject, new Dictionary<string, ReorderableList>() { { fieldName, list } });
                }
                else
                {
                    //对list进行替换
                    rlc[fieldName] = list;
                }
                //if (reorderableListCache.TryGetValue(serializedObject.targetObject, out rlc)) rlc.Add(fieldName, list);
                //else reorderableListCache.Add(serializedObject.targetObject, new Dictionary<string, ReorderableList>() { { fieldName, list } });
            }
            list.list = dynamicPorts;
            list.DoLayoutList();

        }

```

```csharp

对话列表中元素的类：
	/*准备用于序列化的对象必须设置 [System.Serializable] 标签
        ，该标签指示一个类可以序列化
        */
        [Serializable]
	public class SingleChatClass
	{
		public int name;
		
		public Sprite emoji;

		public ChatType chatType;
		
		public string content;

	}
```
```csharp
自定义绘制List及其中元素的方法：
public class MyNodeEditor : NodeEditor {
    public override void OnBodyGUI() {
        // Draw GUI
        NodeEditorGUILayout.DynamicPortList(
            "myFloatList", // field name
            typeof(float), // field type
            serializedObject, // serializable object
            NodePort.IO.Input, // new port i/o
            Node.ConnectionType.Override, // new port connection type
            OnCreateReorderableList); // onCreate override. This is where the magic happens.
    }

//这里你可以选择使用Unity的方法来绘制元素，再传入DynamicportList方法中，这是完全可行的
    void OnCreateReorderableList(ReorderableList list) {
        // Override drawHeaderCallback to display node's name instead
        list.drawHeaderCallback = (Rect rect) => {
            string title = serializedObject.targetObject;
            EditorGUI.Label(rect, title);
        };
    }
}
```

## 程序实现思路 & Story Editor

最后再聊聊程序实现的思路，实际上和树状叙事一样，首先得有一个StartNode，用于定位这段剧本从哪开始；再进入ChatNode或是OptionNode，根据每个元素的类型决定如何播放对话，之后在程序中获取所有的Node，根据类型依次执行就好了:

![](15.png)

我在其中又增加了自由触发方法的对话类型，只需要提前在面板上手动注册过方法，就可以在对话过程中自由调用，通过这种方法和Unity Timeline的Signer配合使用，最后大致完成了想要的过场演出:

![](16.png)

![](17.png)

![](18.gif)

总结下来，实际上这个工具配合Timeline还是有很多不足，因为暂停和播放TimeLine都是由Graph中的数据控制的，实际上应该由Timeline来主导这一切。这导致我对Timeline播放的时间无法直观的控制，之后可能还会改进。

市面上已经有不少完善强大的剧本编辑插件，但自己造轮子的过程确实收获颇多，最后放个xNode文档链接：

[xNode文档](https://github.com/Siccity/xNode/wiki)
