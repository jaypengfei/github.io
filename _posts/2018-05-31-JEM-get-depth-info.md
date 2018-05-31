---
layout: post
title:  JEM中的视频编码块深度信息获取
date:   2018-05-31 20:59:00 +0800
categories: VVC
tag: video coding
---

* content
{:toc}


### 如何获取深度信息

之前JVET刚刚公布了新一代视频编码标准的名字，叫VVC。按照之前一贯的叫法，也可以被称为H.266。而VVC所使用的参考软件也从HEVC的HM变为了现在JEM（最新版本为JEM7.0）。

大家如果在做相关的H.266编码标准的优化工作，那么势必会用到JEM这一参考软件，因为需要在其中来实现相关的算法，并评测其性能。本文主要介绍一下在JEM中是如何获取编码块的最终编码的深度信息。

以分辨率为3328x1664的视频序列为例，JEM在编码时，默认使用的CTU大小为128x128。所以在编码该视频时，共有338个CTU。下面的代码说明如何获取每个CTU内以4*4为单位的块的深度信息。

```c++
/*
*声明一个存储深度信息的数组，为什么是1024大小呢？
*因为CTU大小为128*128，而深度信息是以4*4的块大小为单位存储的
*所以对于128*128的CTU共有1024个深度信息。
*下面的代码，是在TEncCu::compressCtu()中使用的，放于xCompressCU()调用之后
*/
int LCUDepth[1024]; 
for(int i = 0;i < 1024; i++)
{
    LCUDepth[i] = m_pppcBestCU[uiWidthIdx][uiHeightIdx]->getDepth(g_auiRasterToZscan[i]);
}
/*
*获取CTU编号
*/
int ctuid = (int)pCtu->getCtuRsAddr();
```













<hr>
​最后的最后，老婆我爱你。








