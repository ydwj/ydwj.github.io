---
title: 用Unity还原星露谷物语：移动，界面篇（1）
date: 2021-06-25 17:55:33
tags: 
- Unity
- C# 
categories:
- Unity 
keyword: 'Stardew Valley' 
description: 用Unity复刻我最爱的星露谷~
cover : https://z3.ax1x.com/2021/06/26/RGEHkq.gif
---
大家好，我是再无悲喜的炎拳。

作为爆卖1000万份的模拟农场RPG，星露谷自然吸引了我这样的休闲玩家，本期待能愉悦的搭建自己的小农场，结果却沉迷钓鱼下矿无法自拔，过起了007的日子，最后只能大叹西八！

![熬夜伤身体~](1g.gif)

当然，这些只是玩笑话，星露谷中丰富的人物互动事件，有趣的耕种建造系统，以及春夏秋冬独有节日和大量的彩蛋，让玩家刚入便能迅速融入这个可爱的小镇。听着小曲，结束一天的冒险后，带着路边摘得野花回家，路上遇到喜欢的人，顺手将花送给她，知道她对你的爱心又多了一个。晚上在家门口钓钓鱼，回家摸摸宠物入睡。。这样充实愉悦的生活，不由让我感叹，比起城里和乡下，还是做一只星露谷中的老鼠好啊~

![夏日结束的时候，记得来看月光水母起舞~](2g.gif)

废话不多说，进入主题，原作者Eric Barone花费了四年时间，独立开发了星露谷物语，可见里面看似简单的功能还是工作量蛮大的。实际上在复刻星露谷的过程中，我也逐渐察觉到了许多在玩的时候，并未在意的丰富的细节。尝试还原自己喜爱的游戏，是一个有趣的过程，你能更为透彻的感受这个世界的肌理，以及制作者的心情~

---

作为一款2D像素游戏，首先自然是移动，网上找的星露谷素材包里人物的精灵图分割的挺有条理，我选择了阿比盖尔作为主角。

使用前需要对素材进行设置，过滤模式设置为Point，默认选项设置为不压缩图片质量，同时修改图集参数Pixels Per Unit为16（具体根据图集的人物长度决定）。 最后用SpriteEditor中Grid By Cell Size的方法将图集分割成16 * 16像素大小的图片，阿比盖尔的素材就准备完成了：

![](3.png)

接下来制作人物的移动动画，星露谷中人物的移动有八个方向：上下左右，左上，左下，右上右下，实际上展示的动画只有上下左右四个方向，素材中已经包含了这一套动作，我们直接使用unity 的动画系统制作各个方向的移动动画即可：

![10s快速做动画！](4g.gif)

同时在代码中写个方向的枚举类做准备，这里看起来比较繁琐，但星露谷中几乎所有的操作都和方向有关，需要获得人物当前精准的方向，所以还是选择了这个比较笨重的办法来控制动画：

```csharp
public enum Direction
{
   Down,Up,Left,Right,DownLeft,DownRight,UpLeft,UpRight,Stand
}
```

然后开始制作动画状态机，星露谷中人物的日常状态为站立和行走，同时每种状态包含四个方向，所以我们可以选择混合树来管理这些方向的转换：

![新建两个混合树](5.png)

![每个混合树包含四个方向的动画](6.png)

星露谷人物移动没有加速的过程，直接匀速运动，所以我们直接设置类型为1D，通过检测按键按下+长按事件，控制动画转换即可：

![](7.png)

```csharp
    public void UniformMove()
    {
        Vector2 position = transform.position;

        switch (GetDirection())
        {
            case Direction.Down:
                position.y = position.y + -Speed * Time.deltaTime;
                break;
            case Direction.Up:
                position.y = position.y + Speed * Time.deltaTime;
                break;
            case Direction.Left:
                position.x = position.x + -Speed * Time.deltaTime;
                break;
            case Direction.Right:
                position.x = position.x + Speed * Time.deltaTime;
                break;
            case Direction.DownLeft:
                position.y = position.y + -Speed * Time.deltaTime * TurnSpeed;
                position.x = position.x + -Speed * Time.deltaTime * TurnSpeed;
                break;
            case Direction.DownRight:
                position.y = position.y + -Speed * Time.deltaTime * TurnSpeed;
                position.x = position.x + Speed * Time.deltaTime  * TurnSpeed;
                break;
            case Direction.UpLeft:
                position.y = position.y + Speed * Time.deltaTime * TurnSpeed;
                position.x = position.x + -Speed * Time.deltaTime * TurnSpeed;
                break;
            case Direction.UpRight:
                position.y = position.y + Speed * Time.deltaTime * TurnSpeed;
                position.x = position.x + Speed * Time.deltaTime * TurnSpeed;
                break;
            case Direction.Stand:
                break;
            default:
                break;
        }
        rigidbody2D.MovePosition(position);
    }
```

游戏中人物选中物品，会有一个举起物品一起走的动作，物品也会有一个一上一下的动画，在状态机里新建一层layer执行物品动画即可，最终效果如下：

![](8g.gif)

这样人物的移动暂时就完成了，后续要添加新的动画也比较方便，接下来完成人物的状态栏背包，首先同样从精灵图集中找到面板素材，面板需要增删物品时自动排列，所以我选择用Unity自带的UI控件Scrollview来完成，物品是在横向逐渐增加，所以我们直接删除滚动条，给Viewport增加一个Horizontal组件：

![](9g.gif)

然后再根据需求建立物品的模板预制体，并在Horizontal中测试出物品之间的合适的间距，当涉及到物品的增删操作，就不需要自己再调整布局了，效果如下：

![](10g.gif)

这里可能有朋友注意到不同物品的信息面板大小也有所不同，星露谷中物品繁多，对应的描述文字数量也不同，所以当我们生成不同的物品的时候，需要根据文字的数量来决定面板的宽度，具体代码如下：

```csharp
 //构造信息展示面板 ，传入参数的info中包含描述信息
    public void  SetDesPanel(ItemInfo info )
    {
        //载入字体，获取描述文字
        Font font = Resources.Load<Font>("Font/Arial");
        int fontsize = 18;
        string text = info.description;

        //获取这段文字的总字符长度
        font.RequestCharactersInTexture(text, fontsize, FontStyle.Normal);
        CharacterInfo characterInfo;
        float width = 0f;
        for (int i = 0; i < text.Length; i++)
        {
           // Debug.Log("字符长度");
            font.GetCharacterInfo(text[i], out characterInfo, fontsize);
            width += characterInfo.advance;
        }

        //设置信息面板长宽
        if (width <=253 && width > 173)
        {
            DisplayPanel.rectTransform.sizeDelta = new Vector2(width, 245);
        }
        else if (width < 173)
        {
            DisplayPanel.rectTransform.sizeDelta = new Vector2(173, 245);
        }
        else
        {
            int line =(int)(width /253);
            //Debug.Log("行数"+line );
            DisplayPanel.rectTransform.sizeDelta = new Vector2(300, 245+line*27);
        }
    }
```

同时要保证信息面板始终渲染在物品的后面，这样才不会被物品图标遮挡，所以将其父级设置为Inventory_Tabs，和Viewport同一层级：

```csharp
DisplayPanel.transform.SetParent(inventory_Tabs.transform,false );
```
最后一个关键点是选中物品并对其进行操作，这里要结合数据的读取来说，我把这部分内容放在下一篇，最后放一张目前实现的进度：包括从Excel表格和json中分别载入物品信息，数据的存储，基于Tilemap的单独每块地上植物的小动画（有点小麻烦），施工还没完成，就先不放工程了，敬请期待~

![](11g.gif)



