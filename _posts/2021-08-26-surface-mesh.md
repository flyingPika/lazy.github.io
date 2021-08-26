---
title: 表面网格
date: 2021-08-26 00:02:00 +0800
categories: [CG, GEOMETRY]
tags: [cgal]     # TAG names should always be lowercase
math: false
pin: false
---

类 Surface_mesh 是一个半边数据结构的实现，用于表示多面体表面。它是包 Halfedge Data Structures 和包 3D Polyhedral Surface 的替代。主要区别在于它基于索引而不是指针。此外，向顶点、半边、边和面添加信息的机制更加简单，在运行时完成而不是编译时。

因为该数据结构使用整数索引作为顶点、半边、边和面的描述符，所以它比基于64位指针的版本具有更低的内存占用。由于索引是连续的，它们可以用作存储属性的 vector 的索引。

当删除元素时，它们只被标记为已删除，要真正删除它们必须调用垃圾收集函数。

类 Surface_mesh 可以通过其类成员函数、以及 CGAL 包和 Boost Graph Library 所描述的 BGL API 使用，因为它是概念 MutableFaceGraph 和 FaceListGraph 的模型。因此，可以在表面网格上使用包 Triangulated Surface Mesh Simplification、Triangulated Surface Mesh Segmentation 和 Triangulated Surface Mesh Deformation 的算法。

## 1 使用

