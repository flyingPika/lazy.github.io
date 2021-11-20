---
title: 三角网格惯性张量
date: 2021-11-20 00:00:00 +0800
categories: [CG, SIMULATION]
tags: [moment of inertia]     # TAG names should always be lowercase
math: true
pin: false
---

## 1 前言

### 1.1 惯性张量

&emsp;&emsp;在刚体动力学中，惯性张量描述质量如何在物体中分布。对于一个对象集合，其惯性张量为各对象惯性张量的和。角动量 $L$ 为惯性张量 $I$ 和角速度 $\omega$ 的乘积。

$$L=I\omega$$

&emsp;&emsp;对于由质点构成的物体，记 $r_i$ 为第 $i$ 个质点的位置向量，记 $m_i$ 为第 $i$ 个质点的质量，且用 $E$ 表示单位矩阵。于是有

$$I=\sum_{}{m_i(r_i^2E-{r_i}\otimes{r_i})}$$

式中的 $\otimes$ 表示张量积。

### 1.2 质心

&emsp;&emsp;通过对离散的质点进行加权平均，可以求得质心 $c$。

$$c=\frac{\sum_{}m_ix_i}{\sum_{}m_i}$$

### 1.3 平行轴定理

&emsp;&emsp;使用平行轴定理可以将惯性张量转换到具有不同原点的坐标系。在原坐标系下，令物体的质心位于原点处，惯性张量为 $I$，且记新坐标系的原点位置为 $r_0$。于是有

$$I'=I+(r_0^2E-{r_0}\otimes{r_0})\sum_{}{m_i}$$

式中 $I'$ 为新坐标系下的惯性张量。为进一步简化，对于向量 $v$，我们记

$$T(v)=v^2E-{v}\otimes{v}$$

取标量 $a$，易证

$$T(av)=a^2T(v)$$

于是有

$$I'=I+T(r_0)\sum_{}{m_i}$$

### 1.4 三角网格惯性张量

&emsp;&emsp;直接计算三角网格惯性张量的计算公式已有研究人员[1, 2]提出，下文基于三角形分割和相似的思想，对该公式进行了推导。

## 2 三角形的惯性张量

![图1](/assets/img/posts/2021-11-20/p1.png)
_图1_

&emsp;&emsp;对于图1所示的 $△ABC$，其由三个顶点连接而成，密度均匀，质量为 $M$。$G$ 为 $△ABC$ 与 $△DEF$ 的质心，且为坐标系的原点 $O$。$G_1$、$G_2$ 和 $G_3$ 分别为 $△ADF$、$△DBE$ 和 $△FEC$ 的质心。记 $△ABC$ 的惯性张量为 $I_{O-△ABC}$ ，其他三角形的惯性张量同理。于是有

$$I_{O-△ABC}=I_{O-△ADF}+I_{O-△DBE}+I_{O-△FEC}+I_{O-△DEF}$$

又有

$$I_{O-△ADF}=\frac{1}{16}I_{O-△ABC}+\frac{1}{4}MT(GG_1)=\frac{1}{16}I_{O-△ABC}+\frac{1}{16}MT(GA)$$

$$I_{O-△DBE}=\frac{1}{16}I_{O-△ABC}+\frac{1}{4}MT(GG_2)=\frac{1}{16}I_{O-△ABC}+\frac{1}{16}MT(GB)$$

$$I_{O-△FEC}=\frac{1}{16}I_{O-△ABC}+\frac{1}{4}MT(GG_3)=\frac{1}{16}I_{O-△ABC}+\frac{1}{16}MT(GC)$$

$$I_{O-△DEF}=\frac{1}{16}I_{O-△ABC}+\frac{1}{4}MT(GG)=\frac{1}{16}I_{O-△ABC}$$

于是有

$$\begin{split}
I_{O-△ABC}&=\frac{1}{4}I_{O-△ABC}+\frac{1}{16}M(T(GA)+T(GB)+T(GC))\\
&=\frac{1}{12}M(T(GA)+T(GB)+T(GC))
\end{split}$$

![图2](/assets/img/posts/2021-11-20/p2.png)
_图2_

&emsp;&emsp;对于图2所示的 $△ABC$ ，坐标原点 $O$ 与质心 $G$ 不重叠，使用平行轴定理可得

$$I_{O-△ABC}=\frac{1}{12}M(T(GA)+T(GB)+T(GC))+MT(GO)\tag{1}$$

## 3 四面体的惯性张量

![图3](/assets/img/posts/2021-11-20/p3.png)
_图3_

&emsp;&emsp;对于图3所示的四面体，其由四个顶点连接而成，密度均匀并设为1，质量为 $M$。$G$ 为四面体的质心，$G_1$ 为 $△ABC$ 的质心，$O$ 为坐标系的原点。记四面体 $ABCD$ 的惯性张量为 $I_{O-ABCD}$ ，其他四面体的惯性张量同理。于是有

$$I_{O-ABCD}=I_{O-ABCG}+I_{O-ABDG}+I_{O-ACDG}+I_{O-BCDG}$$

设 $△ABC$ 的面积为 $S_{△ABC}$，$G$ 到 $△ABC$ 的距离为 $h_1$。积分可得四面体 $ABCG$ 的惯性张量 $I_{O-ABCG}$ 。

$$\begin{split}
I_{O-ABCG}&=\int_{0}^{1}{t^2S_{△ABC}h_1(\frac{1}{12}(T(G_1A)+T(G_1B)+T(G_1C))+T(OG_1))t^2dt}\\
&=\frac{1}{12}S_{△ABC}h_1((T(G_1A)+T(G_1B)+T(G_1C))+12T(OG_1))\int_{0}^{1}t^4dt\\
&=\frac{1}{60}S_{△ABC}h_1((T(G_1A)+T(G_1B)+T(G_1C))+12T(OG_1))
\end{split}$$

又有

$$\frac{1}{4}M=\frac{1}{3}S_{△ABC}h_1$$

$$T(OG_1)=T(GG_1)=\frac{1}{9}T(DG)=\frac{1}{9}T(GD)$$

于是有

$$I_{O-ABCG}=\frac{1}{80}M((T(G_1A)+T(G_1B)+T(G_1C))+\frac{4}{3}T(GD))$$

下面计算 $T(G_1A)$ ，以此类推 $T(G_1B)$ 和 $T(G_1C)$ 。

$$\begin{split}
T(G_1A)&=T(G_1G+GA)\\
&=T(\frac{1}{3}GD+GA)\\
&=(\frac{1}{3}GD+GA)^2E-(\frac{1}{3}GD+GA)\otimes(\frac{1}{3}GD+GA)\\
&=(\frac{1}{9}(GD)^2+(GA)^2+\frac{2}{3}{GD}\cdot{GA})E-(\frac{1}{9}GD\otimes{GD}+\frac{1}{3}GD\otimes{GA}+\frac{1}{3}GA\otimes{GD}+GA\otimes{GA})\\
&=\frac{1}{9}T(GD)+T(GA)+\frac{2}{3}(GD\cdot{GA})E-\frac{1}{3}(GD\otimes{GA}+GA\otimes{GD})
\end{split}$$

于是有

$$T(G_1B)=\frac{1}{9}T(GD)+T(GB)+\frac{2}{3}(GD\cdot{GB})E-\frac{1}{3}(GD\otimes{GB}+GB\otimes{GD})$$

$$T(G_1C)=\frac{1}{9}T(GD)+T(GC)+\frac{2}{3}(GD\cdot{GC})E-\frac{1}{3}(GD\otimes{GC}+GC\otimes{GD})$$

所以

$$\begin{split}
I_{O-ABCG}&=\frac{1}{80}M(\frac{1}{3}T(GD)+T(GA)+T(GB)+T(GC)+\frac{2}{3}GD\cdot(GA+GB+GC)E-\frac{1}{3}GD\otimes(GA+GB+GC)-\frac{1}{3}(GA+GB+GC)\otimes{GD}+\frac{4}{3}T(GD))\\
&=\frac{1}{80}M(\frac{1}{3}T(GD)+T(GA)+T(GB)+T(GC)-\frac{2}{3}(GD)^2E+\frac{1}{3}GD\otimes{GD}+\frac{1}{3}GD\otimes{GD}+\frac{4}{3}T(GD))\\
&=\frac{1}{80}M(T(GA)+T(GB)+T(GC)+\frac{1}{3}T(GD)-\frac{2}{3}T(GD)+\frac{4}{3}T(GD))\\
&=\frac{1}{80}M(T(GA)+T(GB)+T(GC)+T(GD))
\end{split}$$

同理易得

$$I_{O-ABCG}=I_{O-ABDG}=I_{O-ACDG}=I_{O-BCDG}$$

于是有

$$\begin{split}
I_{O-ABCD}&=4I_{O-ABCG}\\
&=\frac{1}{20}M(T(GA)+T(GB)+T(GC)+T(GD))
\end{split}$$

若坐标原点 $O$ 与质心 $G$ 不重叠，使用平行轴定理可得

$$I_{O-ABCD}=\frac{1}{20}M(T(GA)+T(GB)+T(GC)+T(GD))+MT(GO)\tag{2}$$

## 4 三角网格惯性张量

### 4.1 薄壳惯性张量

为计算网格的薄壳惯性张量，我们将网格视为密度均匀且为1的薄壳。

首先计算网格的面质心。对三角面片的三个顶点加权平均，可以得到面片的质心。对于网格各面片，使用面积对质心进行加权，可以得到网格的面质心。

我们将网格的面质心作为网格的旋转中心，即网格各面片的旋转中心。由公式(1)可得各面片的惯性张量，累加可得网格的薄壳惯性张量。

### 4.2 体惯性张量

为计算网格的体惯性张量，我们将网格视为密度均匀且为1的实体。

首先计算网格的体质心。连接三角面片的三个顶点和全局坐标系原点，构建四面体。对四面体的四个顶点加权平均，可以得到四面体的质心。对于由网格各面片定义的四面体，使用体积对质心进行加权，可以得到网格的体质心。注意，这里使用的体积为有向体积，若全局坐标系原点位于网格内，则体积为正数；若全局坐标系原点位于网格外，则体积为负数。

我们将网格的体质心作为网格的旋转中心，即网格各面片定义的四面体的旋转中心。由公式(2)可得各四面体的惯性张量，累加可得网格的体惯性张量。

## 5 应用

惯性张量的特征值为主惯性矩，特征向量为主惯性轴。

主惯性轴可用于计算网格模型的 OBB（有向包围体）[3]，相比 PCA（主成分分析）求出的主轴，主惯性轴的计算利用了顶点间的邻接信息。

## 6 参考文献

[1] Computing the moment of inertia of a solid defined by a triangle mesh

[2] https://github.com/erich666/jgt-code/blob/master/Volume_11/Number_2/Kallay2006/Moment_of_Inertia.cpp

[3] Real-Time Rendering
