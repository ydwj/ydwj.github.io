---
title: 用Unity DOTS制作4万飞剑的太极剑阵！
date: 2021-06-21 14:49:24
tags: 
- Unity
- c# 
- DOTS
- ECS 
categories:
- Unity 
keyword: 'UnityDots ECS'
description: 上手Unity DOTS的小案例
cover : https://i.loli.net/2021/06/21/IHO8uvw4QBpdmYE.jpg
---

大家好，我是炎拳。

Unity DOTS从发布到现在已经过去两年了，虽然距离发布正式版依旧遥遥无期，但官方放出的几个案例所展现出的性能仍然令人神往，然而DOTS 相关package不同版本变动很大，许多老的教程也已经过时，给想要探索的小伙伴制造了不少麻烦。

于是我便尝试上手最新的DOTS，制作了这样一个由42804把飞剑组成的炫酷剑阵，每次点击地板，都会有10000把飞剑飞出大阵攻击目标点后返回。算是致敬了古龙小说中的“剑气纵横三万里 ，一剑光寒十九洲”的光景（笑）。并将制作过程和学习经历分享出来，希望能给同样探索DOTS的小伙伴一些参考。

![](https://z3.ax1x.com/2021/06/22/ReYboq.gif) 

---

文章较长，对准备工作没兴趣的小伙伴直接跳到5~

## 1.创建工程

首先我们下载好 Unity 2020.3.3f1，选择 Universal Render Pipeline 创建工程。

DOTs的相关package并未发布，也无法在 Package Manager中搜索到，我们需要手动去下载这几个包。

## 2.下载Dots相关包

打开上方菜单栏，点击在左上角＋号图标选择Add package from git URL，依次输入：
+ com.unity.entities
+ com.unity.rendering.hybrid
+ com.unity.physics

<div style="width:90%;margin:auto">{% asset_img 3.png %}</div>

等待一会儿包就下载好了，但我们的准备工作还没做完。

## 3.动态合批设置

由于我们将要生成大量相同材质的飞剑，所以将他们合批处理降低Drawcall是有必要的。

首先点击Edit >ProjectSetting>Quality>查看当前使用的Rendering设置文件，关闭SRP Batcher：

<div style="width:90%;margin:auto">{% asset_img 4.png %}</div>
再新建一个材质，勾选Enable GPU Instancing。

<div style="width:90%;margin:auto">{% asset_img 5.png %}</div>
你就会发现Unity将拥有同一此材质的物体合批渲染了：

{% asset_img 6.png %} 
值得一提的是，本工程中的飞剑已经被我用Blender手动将顶点数降低到105个了，因为我最后大致要生成4万把飞剑，原本的飞剑模型有上千个顶点，庞大的定点数会导致我的场景近乎卡死，最后测试我的电脑能顶住的最大顶点数大概是10M左右。

{% asset_img 7.png %}

```csharp

 public SpriteRenderer spriteRenderer;
    //像素点相对位置
    public List<int2> posList;
    [Header("Drawing")]
    //剑阵密度
    public int drawDensity ;
    //剑阵离散程度
    public int disperseMin;
    public static GetPixel Instance;
    //图片宽高
    private int width;
    private int height;

    void Start()
    {
        Instance = this;
        width = spriteRenderer.sprite.texture.width;
        height = spriteRenderer.sprite.texture.height;
        Debug.Log("图片宽度" + width + "图片高度" + height);
        GetPixelPos();     
    }

    public void GetPixelPos()
    {
        int halfHeight= height / 2;
        int halfWidth = width / 2;
        int2 tempPos;
        for (int i = 0; i < height; i += drawDensity)
        {
            for (int j = 0; j < width; j += drawDensity)
            {
                //获取每个位置像素点的颜色
                Color32 c = spriteRenderer.sprite.texture.GetPixel(j, i);
                tempPos.y = (j-halfHeight)*disperseMin; 
               // Debug.Log("RGBA:" + c);
               //如果对应位置颜色不为透明，则记录坐标到List中
                if (c.a != 0)
                {
                    tempPos.x = (i-halfWidth)* disperseMin;
                    posList.Add(tempPos);
                }
            }
        }
    }


```



+ 玉无价

1. 测试1 
2. 测试2 




## 1 










