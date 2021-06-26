---
title: 用Unity还原星露谷物语（2）：种地篇（数据读写+TileMap）
date: 2021-06-26 17:52:13
tags: 
- Unity
- TileMap
- litjson
categories:
- Unity 
keyword: 'Stardew Valley'
description: Unity复刻星露谷数据篇
cover : https://z3.ax1x.com/2021/06/26/RGEHkq.gif
---
大家好，我是想对🐎🏇使用炎拳的炎拳，上一篇星露谷中我完成了人物的移动并搭建了一个简单的界面，这篇就来瞅瞅星露谷的地是怎么种的。

首先看看游戏中的种地的过程：

![](1g.gif)

大致上完成一次耕种，需要这几个步骤：选中锄头—开垦一块地—选择种子种下—浇水；同时种子会随时间的流逝成长，长到有叶片后人物走过，会触发一个庄稼动一下的动画。

所以这里我要做的功能就清晰了，先整个庄稼的预制体，其中包含所需的图片文字素材，然后点击一下土地就在那位置生成一个，大功告成啦！

才怪~

![](2.jpeg)

考虑到星露谷的物品繁多，如果用预制体来制作每个单独的物品，就太笨拙了；同时每个物品有自己的描述和各种属性，对应的Sprites也可以通过其地址实时读取，所以我希望能通过表格来填写这些物品的固有属性，这些数据属于静态数据。

同时种子，鱼，这些物品使用时会有数量的增减，而斧头，锄头这些道具则可以进行升级，数量，等级这些数据属于动态数据，经常改变，游戏结束也需要保存下来。

所以最后我的方案是：物品的名字，描述，Sprites地址等静态数据通过表格获得，表格的读取使用EPPlus插件实现；物品的数量，等级等动态数据，以json文件的方式保存，这里我使用了LitJson来实现。

首先写个物品的基础类，再在表格中填上对应的静态数据：

```csharp
public class ItemInfo
{
    //动态数据,从json中获取
    public int Id;
    public int amount;
    public ItemQuality itemQuality;
    public bool isSelected;

    //静态数据
    public string name;
    public string description;
    public string []icons;//表格中图片名字的集合
    public ItemType itemType;
    public Tool tool;
    public int growTime;

    /// 构造函数仅构造静态数据，通过表格id获取
    public ItemInfo(int BaseDataid )
    {
        this.name = GetBaseData.instance.baseInfo[BaseDataid].name ;
        this.description = GetBaseData.instance.baseInfo[BaseDataid].description;
        this.icons = GetBaseData.instance.baseInfo[BaseDataid].icons;
        this.itemType = GetBaseData.instance.baseInfo[BaseDataid].itemType;
        this.tool = GetBaseData.instance.baseInfo[BaseDataid].tool;
        this.growTime = GetBaseData.instance.baseInfo[BaseDataid].growTime ;
    }
}
```
![](3.png)

表格的读取我使用EPPlus插件，使用方法很简单，网上找到对应dll文件到工程，调用这两个指令集：

```csharp 
using OfficeOpenXml;
using System.IO;
```
静态数据读取：

```csharp
    public void GetExcelData()
    {
        string FilePath = Application.dataPath + "/Resources/ItemData/BaseData.xlsx";
        Debug.Log("路径" + FilePath);
        FileInfo fileInfo = new FileInfo(FilePath);
        if (fileInfo.Length == 0)
        {
            Debug.Log("Excel文件不存在");
            return;
        }

        using (ExcelPackage excel = new ExcelPackage(fileInfo))
        {
            ExcelWorksheet worksheet = excel.Workbook.Worksheets[1];

            int maxRow_Data = worksheet.Dimension.End.Row; //行
            int maxColumn_Data = worksheet.Dimension.End.Column; //列
            int Id = 0;

            //从第二行开始读取数据
            for (int i = 2; i <= maxRow_Data; i++)
            {
                BaseMessage info  = new BaseMessage();
                string name=null;
                for (int j = 1; j <= maxColumn_Data; j++)
                {
                    switch (j)
                    {
                        case 1:
                            info.name = worksheet.Cells[i, j].Value.ToString();
                            Debug.Log("名字" + name);
                            break;
                        case 2:
                            info.description = worksheet.Cells[i, j].Value.ToString();
                            break;
                        case 3:
                            string temp = worksheet.Cells[i, j].Value.ToString();
                            info.icons = temp.Split('，');
                            break;
                        case 4:
                            string temp2 = worksheet.Cells[i, j].Value.ToString();
                            info.itemType = (ItemType)int.Parse(temp2);
                            break;
                        case 5:
                            string temp5 = worksheet.Cells[i, j].Value.ToString();
                            info.tool = (Tool)int.Parse(temp5);
                            break;
                        case 6:
                            string temp6 = worksheet.Cells[i, j].Value.ToString();
                            info.growTime= int.Parse(temp6);
                            break;
                        default:
                            break;
                    }
                }
                baseInfo.Add(Id , info);
                Id++;
            }
        }//关闭表格
    }
```
物品的Sprite单独写了个方法，方便直接通过名字读取：

```csharp

    public string SpritesPath = "Sprites";
    private Sprite[] sprites;
    private Dictionary<string, object> spritesDictionary = new Dictionary<string, object>();
    // 加载所有精灵图
    public void LoadAllSprites()
    {
        sprites = Resources.LoadAll<Sprite>(SpritesPath);
        if (sprites.Length == 0)
        {
            Debug.Log("精灵图没读到");
        }
        for (int i = 0; i < sprites.Length; i++)
        {
            spritesDictionary.Add(sprites[i].name, sprites[i]);
        }
    }

    // 直接通过名字获得sprite
    public Sprite ReadSpritesByString (string name)
    {
        if (name == null)
            Debug.Log("名字为空不存在");
        Sprite a = null;
        foreach (KeyValuePair<string, object> pair in spritesDictionary)
        {
            // Debug.Log(pair.Key + " " + pair.Value);
            if (pair.Key.ToString() == name)
            {
              a = pair.Value as Sprite;
            }
        }
        return a;
    }
```

然后新建一个动态数据类和专门存放List<动态数据>的类，方便json读写：

```csharp
public class DynamicData
{
    public int jsonId;//动态数据ID
    public int baseId;//对应的静态数据ID
    public int amount;
    public bool isSelected;
}

public class DynamicList
{
    public List<DynamicData> dyDatas=new List<DynamicData>();
}
```
 Json数据读写:

 ```csharp
using LitJson;
using System.IO;


    //存储背包动态数据
    public void SaveBagData()
    {
        string json = JsonMapper.ToJson(Bagmanager.instance.dynamicList);
        
        //路径下的文件不存在就新建个
        if (!File.Exists(path))
            File.Create(path ).Close();
        
        //选择覆盖模式，每次写入数据会清除上次数据
        using (StreamWriter sw = new StreamWriter(new FileStream(path , FileMode.Truncate)))
        {
            sw.Write(json);
        }
    }

    //读取背包数据
    public void ReadBagData()
    {
        //检查文件是否存在
        if (!File.Exists(path))
            return;

        using (StreamReader sr = new StreamReader(new FileStream(path, FileMode.Open)))
        {
            string json = sr.ReadToEnd();
            DynamicList tempList = new DynamicList();
            tempList = JsonMapper.ToObject<DynamicList>(json);
            Bagmanager.instance.dynamicList = tempList;
        }
        Bagmanager.instance.RefreshBagData();
    }

 ```

 完成了背包物品的动静态数据读取后 ,还要让数据到背包的UI面板上展示，这里就可以新建一个模板预制体了，同时这里再新建一个脚本负责在游戏开始时，将相应的数据填入到预制体，并在面板上生成，这里就不详细说了：

 ![](4.png)

 ![](5.png)


 ---

 物品系统和背包有个雏形了，接下来看看土地咋整，首先需要对每一块土地单独进行操作，所以需要一个规律的网格状的土地，这里当然要使用Tilemap（瓦片地图）系统。

开始我以为Tile类就是组成Tilemap的每个图块，但查看了定义才发现Tile本质还是一个继承了ScriptableObject的，类似 unity 材质或纹理资源的文件。（使用过Tilemap的朋友知道，使用前的第一步就是在Tile palette中导入Sprite生成Tile文件）所以直接对单个Tile文件操作，进而改变游戏中的图块是不可取的，本末倒置了。

好在Unity还是很贴心的提供了方法来对每个单元格进行操作，你可以获取Tilemap的每个图块的坐标，然后进行Tile的更换：

![](7.png)

接下来开始实现土地，这里我创建了两层Tilemap，第一层展示土地状态（比如浇水土地会湿一块），第二层展示物品，每次使用对应的Tile都要手动将sprite拖到Tile palette中生成还是挺麻烦的，这里同样写了个根据Sprite名生成Tile 的方法：

![](8.png)

```csharp
  //判定在Tilemap文件夹下否存在该名称的tile文件，不存在就创建新的并保存下来

    public Tile CheckTileExits(string name)
    {
        string tempPath = string.Format("{0}{1}", "Tilemap/", name);
        string Path = string.Format("{0}{1}{2}", "/Resources/Tilemap/", name, ".asset");
        string Lastpath = Application.dataPath + Path;
        if (!File.Exists(Lastpath))
        {
            CreateExampleAsset(name);
            Tile land1 = (Tile)Resources.Load(tempPath);
            return land1;
        }
        Tile land2 = (Tile)Resources.Load(tempPath);
        return land2;
    }

    // 根据精灵图名字创建tile
    public void CreateExampleAsset(string name)
    {
        string Path = "Assets/Resources/Tilemap/";
        Tile exampleAsset = Tile.CreateInstance<Tile>();
        exampleAsset.sprite = Bagmanager.instance.ReadSpritesByString(name);
        string temp = string.Format("{0}{1}{2}", Path, name, ".asset");
        AssetDatabase.CreateAsset(exampleAsset, temp);
        AssetDatabase.Refresh();
    }
```
然后新建一个土地类：

```csharp
public enum LandType
{
    Unkown, normal, reclaimed, seeded, watered, planted
}

public class Land
{
    public int Id;
   
    //当前土地状态
    public LandType landType;

    //每一块土地的坐标（对应Tilemap中的图块坐标）
    public Vector3Int LandPos;

    //需要展示的图片集合
    public string [] icons ;

    //种子所需的成长时间
    public int growTime;

    //种子一开始种下的时间
    public int startTime;

    //种子实际成长时间
    public float actualTime;
}
```

土地数据的读写操作和物品的差不多，这里也不过多赘述，接下来解决操作问题，这里先不管动画，解决主要问题（其实就是懒），星露谷中允许对人物周围的8个图块进行各种操作，所以我们需要人物自身的坐标，土地已经网格化了，所以人物自身的坐标也需要转换土地对应的坐标（这里要以人物的脚为中心点）：

```csharp

 public Vector3Int[] GetHumanAround()
    {
        Vector3Int temp = GetCurPos();
        Vector3Int[] temps = new Vector3Int[8];
        temps[0] = new Vector3Int(temp.x-1, temp.y+1,0);
        temps[1] = new Vector3Int(temp.x, temp.y+1,0);
        temps[2] = new Vector3Int(temp.x+1, temp.y+1,0);
        temps[3] = new Vector3Int(temp.x-1, temp.y,0);
        
        temps[4] = new Vector3Int(temp.x+1, temp.y,0);
        temps[5] = new Vector3Int(temp.x-1, temp.y-1,0);
        temps[6] = new Vector3Int(temp.x, temp.y-1,0);
        temps[7] = new Vector3Int(temp.x + 1, temp.y - 1, 0);
        return temps;
       
    }
```
同时鼠标坐标也转换为Int型，在指定图块进行操作，满足条件即可执行.

最后值得一提的是时间和每个图块被人物碰到的小动画，这两个部分我都用协程完成，虽然用协程记录时间会有误差，但星露谷并不是对时间要求很精确的游戏：

时间：

```csharp

// 10s记录一次时间，也顺便刷新右上角的时间面板（10s相当于星露谷的10分钟）
  public IEnumerator TimeIncrease()
    {
        while (true)
        {
            while (gameTime.time <= 1200)
            {
                gameTime.time += 10f;
                rotate.z = rotate.z + 0.15f;

                yield return new WaitForSeconds(10f);
                RotatePoint.Rotate(rotate);
                dayText.text = string.Format("{0}{1}", gameTime.day.ToString(), "日");
                hour = (int)(gameTime.time / 60);
                minute = (int)(gameTime.time - hour * 60);
                string tempTime = string.Format("{0}{1}{2}", hour.ToString(), ":", minute.ToString());
                minuteText.text = tempTime;
             }
            DayEnd();
        }
 }

```

每个图块的动画这里有点麻烦，Unity将每个图块的transform数据封装到了一个4x4的矩阵中吗，所以需要额外做一次矩阵转换，再调用这个方法完成这个小动画：

![](9.png)

```csharp
    //判断是否碰到了种子
    public void TouchCrop()
    {
        Vector3Int tempPos1 = GetCurPos();
        Matrix4x4 curTrans=new Matrix4x4();
        foreach (var key in LandManager.instance.seededPos.Keys)
        {
            if (LandManager.instance.seededPos[key] == tempPos1)
            {
                curTrans = LandPanel.level2.GetTransformMatrix(LandManager.instance.lands.land2[key].LandPos);
               if (direction == Direction.Left || direction == Direction.DownLeft || direction == Direction.UpLeft)
                {
                   StartCoroutine(LeftShake(0.5f, curTrans, key));
                }
                else
                {
                   StartCoroutine(RightShake(0.5f, curTrans, key));
                }
                break;
            }
        }
    }

    // 庄稼左抖动
    public IEnumerator  LeftShake(float duration,Matrix4x4 curTrans,int key)
    {
        Vector3 temp = new Vector3(0, 0, 0);
        float elapsed = 0f;

        while (elapsed < duration)
            {
                if (elapsed < duration / 2)
                {
                    temp = temp + (leftAngel / duration) * Time.deltaTime * 2;
                    Quaternion temp_rotation = Quaternion.Euler(temp);
                    curTrans.SetTRS(curTrans.GetPostion(), temp_rotation, new Vector3(1, 1, 1));
                    LandPanel.level2.SetTransformMatrix(LandManager.instance.lands.land2[key].LandPos, curTrans);
                }
                else if(elapsed < duration)
                {
                    temp= temp- (leftAngel / duration) * Time.deltaTime * 2;
                    Quaternion temp_rotation = Quaternion.Euler(temp);
                    curTrans.SetTRS(curTrans.GetPostion(), temp_rotation, new Vector3(1, 1, 1));
                    LandPanel.level2.SetTransformMatrix(LandManager.instance.lands.land2[key].LandPos, curTrans);
                }
            elapsed += Time.deltaTime;
            yield return 0;
        }
    }
```

最后展示下目前的施工进度，星露谷还未完结，暂时先不放这个还未圆满的工程了:


![](10g.gif)

最后感谢知乎 @絮酱酱 的EPPlus的小教程，永远滴神：马三小伙儿写的Litjson扩展（Litjson原版不支持Vector3Int类型数据）和_阿松先生的矩阵转换工具类，感谢大佬们的工作，让俺节省了很多的功夫~
贴上马三小伙儿和阿松先生的原帖地址：

[魔改Json](https://cloud.tencent.com/developer/article/1608178)

[Matrix4x4矩阵变换详细实例](https://blog.csdn.net/aaa583004321/article/details/81948780?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)