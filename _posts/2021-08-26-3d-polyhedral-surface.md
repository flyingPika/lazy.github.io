---
title: 三维多面体表面
date: 2021-08-26 00:01:00 +0800
categories: [CG, GEOMETRY]
tags: [cgal]     # TAG names should always be lowercase
math: false
pin: false
---

# 三维多面体表面

## 1 介绍

三维多面体表面由顶点、边、面和它们之间的邻接关系组成。下面的组织是半边数据结构，其限制可表示的表面类别为可定向二维流形（有边界和无边界）。如果表面是闭合的，我们称之为多面体。

多面体表面被视为容器类，它管理顶点、半边、面以及关联，并且维护它们的组合完整性。它基于半边数据结构的高度灵活设计。然而，多面体表面可以在不知道底层设计的情况下使用和理解。本章的一些例子也会逐步介绍这种灵活性的首次应用。

## 2 定义

三维多面体表面 Polyhedron_3<PolyhedronTraits_3> 包含顶点 V、边 E、面 F 和它们的邻接关系。每个边使用两个相反方向的半边表示。用半边存储的关联如下图所示。

顶点表示空间中的点，边为两个端点间的直线段，面为无孔的平面多边形。面由沿边界的半边循环序列定义。多面体表面本身可以有孔。沿着孔的边界的半边称为边界半边，并且没有邻接面。一条边为边界边，若其某一半边为边界半边。表面不包含边界半边，则为闭合的。闭合表面是三维多面体的边界表示。依照惯例，从多面体外部看，半边沿面的逆时针方向。**这意味着半边沿顶点顺时针方向。**尽管没有定义闭合的物体，但由半边方向定义的面的实体边的概念，扩展到了有边界的多面体表面。考虑面的法向量，其指向外（遵循右手定则）。

严格的定义见参考文献。该定义的一个含义是，多面体表面总是可定向的，并且是具有边界的定向二维流形。另一含义是，避免自交的最小可表示表面为三角形（有边界的多面体表面）或四面体（多面体）。定向二维流形的边界表示在欧拉操作下封闭。它们通过在表面上创建或闭合孔的操作扩展。

除关联关系外，其他相交不允许出现。然而，由于自相交难以有效检查，这不能在操作下自动验证。Polyhedron_3<PolyhedronTraits_3> 只维护多面体表面的组合完整性，并且不考虑点的坐标或其它几何信息。

Polyhedron_3<PolyhedronTraits_3> 既可以表示多面体表面，也可以表示多面体。接口的设计方式使得容易忽略边界边、只作用于多面体。

## 3 示例程序

多面体表面基于半边数据结构的高度灵活设计。灵活性示例见第5节。这种设计不是理解下面示例的前提条件。高级示例见第6节。

### 3.1 First Example Using Defaults

第一个例子使用内核作为 Traits 类初始化多面体。它创建了一个四面体并且在 Halfedge_handle 中存储其中一个半边的引用。句柄用于保存对半边、顶点和面的引用，以备将来使用。又有 Halfedge_iterator 用于枚举半边。这种迭代器可以在任何需要句柄的地方使用。常量多面体也分别提供了 Halfedge_const_handle 和 Halfedge_const_iterator，以及类似的、以 Vertex_ 和 Facet_ 为前缀的句柄和迭代器。

该实例继续测试半边是否引用四面体。测试检查半边 h 所指的连通分量，而不是整个多面体表面。这个例子只适用于多面体表面的组合级别。下一个例子添加几何元素。

```cpp
#include <CGAL/Simple_cartesian.h>
#include <CGAL/Polyhedron_3.h>
typedef CGAL::Simple_cartesian<double>     Kernel;
typedef CGAL::Polyhedron_3<Kernel>         Polyhedron;
typedef Polyhedron::Halfedge_handle        Halfedge_handle;
int main() {
    Polyhedron P;
    Halfedge_handle h = P.make_tetrahedron();
    if ( P.is_tetrahedron(h))
        return 0;
    return 1;
}
```

### 3.2 Example with Geometry in Vertices

我们在四面体的构造中添加几何元素。四个点作为参数被传递给构造函数。该示例还演示了顶点迭代器的使用和对顶点中点的访问。注意访问符号 v->point()。类似地，所有存储在顶点、半边和面中的信息可以通过句柄或迭代器的成员函数访问。

```cpp
#include <CGAL/Simple_cartesian.h>
#include <CGAL/Polyhedron_3.h>
#include <iostream>
typedef CGAL::Simple_cartesian<double>     Kernel;
typedef Kernel::Point_3                    Point_3;
typedef CGAL::Polyhedron_3<Kernel>         Polyhedron;
typedef Polyhedron::Vertex_iterator        Vertex_iterator;
int main() {
    Point_3 p( 1.0, 0.0, 0.0);
    Point_3 q( 0.0, 1.0, 0.0);
    Point_3 r( 0.0, 0.0, 1.0);
    Point_3 s( 0.0, 0.0, 0.0);
    Polyhedron P;
    P.make_tetrahedron( p, q, r, s);
    CGAL::IO::set_ascii_mode( std::cout);
    for ( Vertex_iterator v = P.vertices_begin(); v != P.vertices_end(); ++v)
        std::cout << v->point() << std::endl;
    return 0;
}
```

为方便起见，多面体提供了点迭代器。通过使用 std::copy 和输出流迭代适配器，上面的 for 循环可以简化为单个语句。

```cpp
std::copy( P.points_begin(), P.points_end(), std::ostream_iterator<Point_3>(std::cout,"\n"));
```

### 3.3 Example for Affine Transformation

仿射变换 A 可以作为变换点的函子，并且可以为多面体表面定义方便的点迭代器。所以，假设我们只想变换多面体 P 的点的坐标，std::transform 可以在一行中完成。

```cpp
std::transform( P.points_begin(), P.points_end(), P.points_begin(), A);
```

### 3.4 Example Computing Plane Equations

多面体表面已经具备为每个面存储平面方程的条件，但它不提供计算平面方程的函数。

这个例子计算多面体表面的平面方程。实际的计算由 Plane_equtation 函子实现。根据算术（精确/不精确）和面的形状（凸/非凸），不同的方法是有用的。我们这里假设严格的凸面和精确的算术。在我们的例子中，使用 int 坐标的齐次表示是充分的。程序的输出是四面体的四个平面方程。

```cpp
#include <CGAL/Simple_cartesian.h>
#include <CGAL/Polyhedron_3.h>
#include <iostream>
#include <algorithm>
struct Plane_equation {
    template <class Facet>
    typename Facet::Plane_3 operator()( Facet& f) {
        typename Facet::Halfedge_handle h = f.halfedge();
        typedef typename Facet::Plane_3  Plane;
        return Plane( h->vertex()->point(),
                      h->next()->vertex()->point(),
                      h->next()->next()->vertex()->point());
    }
};
typedef CGAL::Simple_cartesian<double> Kernel;
typedef Kernel::Point_3                Point_3;
typedef Kernel::Plane_3                Plane_3;
typedef CGAL::Polyhedron_3<Kernel>     Polyhedron;
int main() {
    Point_3 p( 1, 0, 0);
    Point_3 q( 0, 1, 0);
    Point_3 r( 0, 0, 1);
    Point_3 s( 0, 0, 0);
    Polyhedron P;
    P.make_tetrahedron( p, q, r, s);
    std::transform( P.facets_begin(), P.facets_end(), P.planes_begin(),
                    Plane_equation());
    CGAL::IO::set_pretty_mode( std::cout);
    std::copy( P.planes_begin(), P.planes_end(),
               std::ostream_iterator<Plane_3>( std::cout, "\n"));
    return 0;
}
```

### 3.5 Example with a Vector Instead of a List Representation

### 3.6 Example with Circulator Writing Object File Format (OFF)

### 3.7 Example Using Euler Operators to Build a Cube

### 3.8 Draw a Polyhedron

## 4 文件 I/O

## 5 扩展顶点、半边和面

## 6 高级示例程序
