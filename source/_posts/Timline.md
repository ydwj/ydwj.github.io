---
title: Unity-Timeline基础功能介绍
date: 2021-06-21 14:49:24
tags: 
- Unity
- Timeline
categories:
- Unity 
keyword: 'Timeline'
description: Unity Timeline 学习笔记
cover : https://z3.ax1x.com/2021/06/25/Rlz7tA.jpg
---

**大智老师的课程笔记**

![](https://img-blog.csdnimg.cn/20200909102846317.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

---

## 使用插件版本：

+ TimeLine:1.2.16
+ Cinmachine :2.6.2
+ Unity版本：2019.3.2.f1

![](https://img-blog.csdnimg.cn/2020090911330387.png#pic_center)

TimeLine支持多个系统的协作，而Animation 仅支持动画，所以Timeline在制作一段过场动画有天然的优越性。

**窗口介绍：**

区域播放按钮，选中可以自由选择播放区域
![](https://img-blog.csdnimg.cn/20200909114040197.png#pic_center)

**based on clips**: 根据动画的最后一帧设置TimeLine长度

**Fixed Length**: 手动设置长度

默认时间轴显示帧数（frames），也可以选择秒数（Seconds）查看

可调整动画帧数（Frame Rate），TimeLine总持续时间不变，但帧数改变

![](https://img-blog.csdnimg.cn/2020090911490379.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

![](https://img-blog.csdnimg.cn/20200909115821495.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

---

## Actication Track

功能：控制场景中物体的激活

选中一段clip按A键可以直接显示所有完整的Clip
可以添加多个 Activation clip
Post-playback state:决定此TimeLine播放完后，物体的状态（激活或非激活）

![](https://img-blog.csdnimg.cn/20200909142817949.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

同时单独的clip片段也可以在检视窗口做精确调整：

![](https://img-blog.csdnimg.cn/20200909143441865.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

----

## AnimationTrack

功能：对场景中的对象进行动画编辑

默认录制的Animation无法改变长度，可以直接右键--ConvertToClipTrack，转换为一个Clip，即可在 animation窗口中进行操作（支持在窗口中再此进行录制完成更改）

默认创建的Animation Track是无限时间长度


---

## 使用导入的动画

将物体直接拖入场景实例化，再拖入轨道，同类型的动画文件拖入即可，需要做位移可以单独再建一条轨道进行编辑。


**clip的编辑模式**

![](https://img-blog.csdnimg.cn/2020090915345635.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)
快捷键：非永久的选中某种模式，可以长按数字键：

+ "1"--Mix Mode 
+ "2"---Ripple （波纹）Mode ：等距离移动，移动一个clip前后距离都会发生改变
+ "3"---Replace Mode:不发生混合，直接覆盖

---

## 动画导入的使用全流程

1：素材导入，带位移的动画如何处理

动画素材下载地址：www.mixamocom

材质文件嵌入在这里，需要单独导出

![](https://img-blog.csdnimg.cn/20200909163545732.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

**如何使带有位移的动画持续循环：**

无论是对原动画选择loop模式，还是在timeline 中使用loop模式，都会遇到动画循环播放完，角色回归原点的问题。

解决这个问题需要去查阅动画系统中的Root Motion相关资料，这里不过多赘述，直接给出操作方法：

1：点击进入模型检视窗口，Animation Type改为Humanoid：

![](https://img-blog.csdnimg.cn/20200909170601486.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

2：Avatar 选择Create From This Model 

3：点击进入动画文件视窗，Animation Type同样选择Humanoid，但下方Avatar选择Create From Other Avatar，然后使用刚才模型中创建出来的Avatar

4：重新给场景中的模型的Animator组件赋上Avatar：

![](https://img-blog.csdnimg.cn/20200909172142540.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

5：将动画调为循环状态

![](https://img-blog.csdnimg.cn/20200909172709310.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)
搞定！

**如何让对象在TImeLIne中在自己想要的位置开始运动，同时两段带运动的动画无缝衔接**

1：Track Offsets改为Apply Scene Offsets：

![](https://img-blog.csdnimg.cn/2020090917331517.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

2：两段同样带位移的动画，后面一段记得选Match Offsets To Previous Clip 

![](https://img-blog.csdnimg.cn/20200909174240843.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

---

## 动画覆盖和Avatar Mask

![](https://img-blog.csdnimg.cn/20200909180131500.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

功能：两段动画融合的问题

1:在一段预定要编辑的Animation Track 上右键 "Add Override Track"

![](https://img-blog.csdnimg.cn/20200909180703939.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

2：选择一段动画放入Overide Track，在指定文件夹Create - Avatar Mask

![](https://img-blog.csdnimg.cn/20200909180911530.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

3：此时我们只想人物上半身执行动画，所以将下半身屏蔽，直接点击绿色区域即可

![](https://img-blog.csdnimg.cn/20200909183630955.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

3:将Avater  添加给Override即可，发现Override上出现了一个小人，点击一下即可禁用

![](https://img-blog.csdnimg.cn/20200909185938837.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

---

## Animation Track和Clip 的设置

![](https://img-blog.csdnimg.cn/20200911182120458.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

+ Clip In：决定动画本身素材的播放时间
+ Speed Multiplier：动画播放速度
+ Animation Extrapolation （动画推断）：决定动画播放完后如何表现
+ Blend Curves：动画混入混出的曲线


![](https://img-blog.csdnimg.cn/20200914102403638.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

**Remove Start Offset:整个动画根据第一帧作为起点**

Foot IK：人物行走的时候应用反向动力学，自动调整行走的角度
Loop:是否选择循环（可以选择是否应用动画中的循环）

---

## Aduio Track

1. 直接拖动音频文件到轨道（无立体感）
2. 拖动音频到轨道，选中一个物体拖到音频上即可

![](https://img-blog.csdnimg.cn/20200914104036635.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

![](https://img-blog.csdnimg.cn/20200914105408270.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

Sterep Pan ：声道选项

Spatial Blend：绑定状态下激活，2D &3D 的时间

导入Timeline没声音，记得检查game视窗中是否"MUte Audio"，这样会禁用除了project文件夹中的所有的声音

---

## Control Track

主要功能：控制嵌套的TimeLine和Particle system，也能控制场景内物体的激活（和activation Track）功能类似：

![](https://img-blog.csdnimg.cn/20200914143730905.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

1. 控制TimeLIne物体（Control Playable Directors），将TimeLine直接拖到轨道操作

2. Control Particle System：控制粒子系统，需要将粒子系统提前在Scene 中固定好位置

3. Control ITimeControl ：接口，需要代码实现

4. Control Children：是否允许控制子物体

---

## ITime Control实现自定义控制

功能：可轻松实现一些随Timeline播放时间进行

+ 新建脚本继承ITimeControl接口，并在场景中实例化，同时拖拽到Control Track轨道中

![](https://img-blog.csdnimg.cn/2020091415475567.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

---

## Signal Track（2019.1后新加入的功能 ）

功能：极大的扩展Timeline与场景中物体的交互

运行机制：

![](https://img-blog.csdnimg.cn/20200914161145218.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

> 在TimeLine 中创建Signal轨道，选中一帧添加信号发射器（signal Emitter）

![](https://img-blog.csdnimg.cn/20200914161445205.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

> 创建一个Signal Emitter 的资产:

![](https://img-blog.csdnimg.cn/20200914161744125.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

> 之后将物体拖到轨道对应的signal 标记，之后操作和UI的操作类似，不过多赘述

**signal是支持复用的！很方便！**

> 可以通过Markers在全局的轨道上添加标记:

![](https://img-blog.csdnimg.cn/20200914163106891.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70#pic_center)

> 可以绑定物体的Track都可以添加Signal Emitter



