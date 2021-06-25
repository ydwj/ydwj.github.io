---
title: Shader Learning Notes
date: 2021-06-25 16:10:45
tags: 
- Shader
- Cg
categories:
- Shader
keyword: 'Shader'
description: Shader 笔记
cover : https://z3.ax1x.com/2021/06/25/R1JQPJ.jpg
---

来自蛮牛教育的刘仕斌老师的课程，写这个系列单纯是为了巩固课程内容。
如果有理解错误和不到位的地方希望大佬们多多指正~

---

 - **什么是shader**
 - **什么是渲染管线**
 - **Shader和材质，贴图的关系**

---

## 什么是Shader
 
 Shader，中文翻译即为着色器，是一种较为短小的程序片段，用于**告诉图形硬件如何计算和输出图像**，过去由汇编语言编写，现在也可以使用高级语言编写，一句话概括：**Shader是可编程图形管线的算法片段**。

它主要分两类，Vertex Shader（顶点着色器）和Fragment Shader（片段着色器），两种Shader也分别对应着Gpu上的两个组件：可编程顶点处理器，和可编程片段处理器。

## 可编程顶点处理器和可编程片段处理器

顶点和片段着色器拥有强大的并行计算能力，擅长不高于4阶的矩阵运算，片段着色器还支持高速的纹理查询能力。

## 顶点着色器

输入：GPU前端模块提取图元信息（顶点位置，法线向量，纹理坐标（uv））等
操作：顶点坐标空间转换，法向量空间转换，光照计算等（现在光照计算一般在片段着色器，比较细），然后将计算好的数据传入指定寄存器中

## 片段着色器

输入：顶点着色器传入的数据
操作：光照计算，uv扰动，纹理采样等，最后输出当前片段的颜色给光栅化阶段（片段着色器是对每个独立的颜色进行操作的）

深度理解链接：[the book of shader](https://thebookofshaders.com/01/?lan=ch)

---
## 什么是渲染管线

渲染管线也称之为渲染流水线，是**显示芯片内部处理图形信号，相互独立的并行处理单元**。一个流水线是一序列可以并行和按照固定顺序进行的阶段。每个阶段都从它的前一阶段接受输入，然后把输出发给随后的阶段，就像在同一时间内，不同阶段的不同的汽车装配线，传统的图形硬件流水线以流水的方式处理大量的顶点，几何图元和片段。
 
 ---
### 渲染管线工作流程

![应用程序阶段，几何阶段，光栅阶段](https://img-blog.csdnimg.cn/20200428232934376.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70)

图形渲染管线分为三个阶段：应用程序阶段，几何阶段，光栅阶段，如上图所示；

参考文章链接：

[Shader学习基础之一（图形流水线）](https://blog.csdn.net/xiongwen_li/article/details/72417529)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200428003147818.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM5MzY5MDQ0,size_16,color_FFFFFF,t_70)

**总结**：我们可编程的部分：①顶点变换，光照等（Vertex Shader ）。②如何采样，计算颜色，雾化处理等(Fragment Shader)。

PS：优化一般是减少Drawcall，Drawcall是应用程序在运行过程中调用图形硬件进行渲染的这一调度过程。


参考文章链接：

+ [ShaderLab: Culling & Depth Testing](https://blog.csdn.net/maozi_bsz/article/details/77387855#fnref:grazingangle)
+ [Unity shader深度测试](https://blog.csdn.net/v_xchen_v/article/details/79380222)
[Unity Shaders】Alpha Test和Alpha Blending](https://blog.csdn.net/candycat1992/article/details/41599167)
+ [一口气解决RenderQueue、Ztest、Zwrite、AlphaTest、AlphaBlend和Stencil](https://zhuanlan.zhihu.com/p/28557283)

---
## Shader和材质，贴图的关系

Shader(着色器)实际上就是一小段程序，它负责将输入的顶点数据以指定的方式和输入的贴图或者颜色等组合起来，然后输出。绘图单元可以依据将图像绘制在屏幕上。其中，输入的贴图，颜色等加上Shader，以及Shader特定的参数设置，得到的就是一个Material（材质）。之后，我们便可以将材质赋予给三维物体进行渲染（输出）了。

材质好比最终使用的商品，Shader好比生产这种商品的加工方法，而贴图就是原材料。

---
## 课时总结

+ Shader 是图形可编程方案的程序片段
+ 渲染管线是一种计算机从数据到最终图形成像的形象描述
+ 材质是商品，Shader是方法，贴图是材料