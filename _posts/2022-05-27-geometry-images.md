---
title: Geometry Images
date: 2022-05-31 00:00:00 +0800
categories: [CG, GEOMETRY]
tags: [paper]     # TAG names should always be lowercase
math: true
pin: false
---

## 摘要

曲面几何通常使用不规则网格建模。重网格化过程指，使用半规则连通性近似这样的几何，这有利于许多图形应用。然而，现在的重网格化技术只能创建半规则网格。原始网格通常被分解为圆盘状的图表，在图表上对几何进行参数化和采样。

在这篇文章中，我们将任意曲面重网格化为完全规则结构，并称之为几何图像。它将几何捕获为二维数组，其中存储了量化的点。曲面信号，例如法线和颜色，使用相同的隐式曲面参数化（纹理坐标缺失）存储在二维数组中。为了创建几何图像，我们沿着边连成的网络对网格进行切割，将生成的图表参数化为正方形。几何图像可以使用传统的图像压缩算法编码，例如基于小波的编码器。

**关键词：** remeshing, surface parametrization.

## 1 介绍

将曲面表示为几何图像存在的挑战：

* 寻找割线，使网格与圆盘拓扑同胚，并允许对曲面进行良好的参数化。

* 图像边界必须被参数化，使得重建曲面在割线处精确匹配，避免裂缝。

* 参数化在曲面上均匀分布图像采样。

* 将割线拓扑编码为简单数据边带，以实现割线融合。

几何图像的限制：

* 不能表示非流形几何。

* 展开高亏格曲面会造成极大失真。

在本文中，我们描述了一个将任意网格转换为几何图像和相关属性映射的自动系统。我们证明了几何图像对于各种图形学模型来说是实用而优雅的表示。

## 2 先前工作

我们贡献的关键在于，通过切割曲面并使用完全规则的四边形网格采样，将整个曲面表示为单个几何图像。我们优化了割线的创建，以实现良好的参数化。

## 3 几何图像的创建

M: 二维流形三角网格

M': 割开的 M

ρ: 割线

ρ': 开放式割线

割点: 割线上价不为 2 的顶点

割径：开放式割线上、两个有序割点之间线段和顶点的集合

D: 单位正方形域

φ: 从 D 到 M' 的分段线性映射

几何图像采样用于重建 M 的近似。在这项工作中，我们用线性基函数（三角形）来定义几何重建插值。我们的目标是寻找良好的 ρ 和 φ，使得对于适度的采样率，重建效果良好。

### 方法概述

首先寻找拓扑上充分的割线，使用该割线进行初始参数化。然后基于参数化结果改善割线，使用新割线进行重参数化。切割和重参数化迭代进行，直到参数化结果不再改善。

### 3.1 参数化

#### 边界参数化

约束：

* 不存在三个顶点映射到正方形域同一条边的三角形（细分）

* 打断映射到正方形域角点处的边（细分）

* 价为 1 的割点不被映射到正方形域角点（旋转边界参数化）

#### 内部参数化

> [Texture Mapping Progressive Meshes]

在我们的应用中，一个理想的度量是采样和重建后的曲面精度。$L_2$ 几何拉伸度量是理想测度的近似。

首先，M' 的内部被简化以构成渐进网格表示。生成的基网格中的少量内部点在 D 中使用蛮力优化。然后，在渐进网格中使用顶点分割，以连续细化网格。对于每个添加的顶点，使用局部、非线性的优化算法，优化其邻居的参数化以最小化拉伸。

### 3.2 切割

#### 初始切割

若网格有边界，则固定边界，作为 ρ 的子集。从网格中删除一个种子三角形后，我们应用两个阶段。

在第一阶段中，重复识别网格中不属于固定边界、且仅与一个三角形相邻的边，删除这条边以及相邻三角形。为获得“最小半径”，需要根据三角形与种子三角形间的测地距离，对三角形进行排序。当第一阶段结束时，我们删除了包含网格中所有三角形的拓扑圆盘，所有剩余的顶点和边构成 ρ。

在第二阶段中，重复识别 ρ 中仅与一条边相邻的顶点，删除这个顶点和这条边。第二阶段结束时，只剩下连通的环路，之后通过计算受约束最短路径“拉直”割径。

`对于亏格为 0 的闭合网格，生成的 ρ 仅包含一个顶点，需要添加两条相邻的边。`

#### 迭代切割增强

> [Parametrization and Smooth Approximation of Surface Triangulations]

为得到有效的几何图像，经过网格中的各种“极值”对于 ρ 非常重要。

使用 Floater 方法将 M' 参数化到单位圆的内部，寻找具有最大几何拉伸的三角形，选取其某一顶点作为极值点。然后计算最短路径将极值点添加到 ρ。每次计算新的 ρ 之后，使用 3.1 节的算法进行重参数化。

`对于亏格为 0 的闭合网格，当我们第一次寻找极值点时，用该极值点代替 ρ，再添加两条相邻的边。`

#### 总结

### 3.3 拓扑边带

使用边带信号记录拓扑割线。

## 4 应用

## 5 结果

## 6 总结和讨论
