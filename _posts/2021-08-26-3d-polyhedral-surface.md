---
title: CGAL 5.3 三维多面体表面
date: 2021-08-26 00:01:00 +0800
categories: [CG, GEOMETRY]
tags: [cgal]     # TAG names should always be lowercase
math: false
pin: false
---

<https://doc.cgal.org/latest/Polyhedron/index.html#Chapter_3D_Polyhedral_Surfaces>

## 1 介绍

三维多面体表面由顶点、边、面和它们之间的邻接关系组成。下面的组织是半边数据结构，其限制可表示的表面类别为可定向二维流形（有边界和无边界）。如果表面是闭合的，我们称之为多面体。

多面体表面被视为容器类，它管理顶点、半边、面以及关联，并且维护它们的组合完整性。它基于半边数据结构的高度灵活设计。然而，多面体表面可以在不知道底层设计的情况下使用和理解。本章的一些例子也会逐步介绍这种灵活性的首次应用。

## 2 定义

三维多面体表面 Polyhedron_3<PolyhedronTraits_3> 包含顶点 V、边 E、面 F 和它们的邻接关系。每个边使用两个相反方向的半边表示。用半边存储的关联如下图所示。

顶点表示空间中的点，边为两个端点间的直线段，面为无孔的平面多边形。面由沿边界的半边循环序列定义。多面体表面本身可以有孔。沿着孔的边界的半边称为边界半边，并且没有邻接面。一条边为边界边，若其某一半边为边界半边。表面不包含边界半边，则为闭合的。闭合表面是三维多面体的边界表示。依照惯例，从多面体外部看，半边沿面的逆时针方向。~~这意味着半边沿顶点顺时针方向~~。尽管没有定义闭合的物体，但由半边方向定义的面的实体边的概念，扩展到了有边界的多面体表面。考虑面的法向量，其指向外（遵循右手定则）。

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
std::copy( P.points_begin(), P.points_end(), std::ostream_iterator<Point_3>(std::cout, "\n"));
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
    std::transform( P.facets_begin(), P.facets_end(), P.planes_begin(), Plane_equation());  // ???
    CGAL::IO::set_pretty_mode( std::cout);  // ??????????????????????????
    std::copy( P.planes_begin(), P.planes_end(), std::ostream_iterator<Plane_3>( std::cout, "\n"));
    return 0;
}
```

### 3.5 Example with a Vector Instead of a List Representation

多面体类模板实际上有四个参数，其中三个有默认值。在上面的例子中，显式地使用三个参数的默认值（忽略第四个参数，它是容器类的标准分配器），多面体的定义如下：

```cpp
typedef CGAL::Polyhedron_3< Traits, CGAL::Polyhedron_items_3, CGAL::HalfedgeDS_default> Polyhedron;
```

Polyhedron_items_3 包含用于顶点、边和面的类型。HalfedgeDS_default 定义了所用的半边数据结构，默认基于列表，另一种选择是基于向量。

当剩余空间不足时，基于向量的表示会自动改变大小。

注意，触发调整大小操作的是多面体，而不是底层的半边数据结构。因为调整大小操作需要满足只有多面体能保证的前提条件，例如有效索引。

```cpp
#include <CGAL/Simple_cartesian.h>
#include <CGAL/HalfedgeDS_vector.h>
#include <CGAL/Polyhedron_3.h>
#include <iostream>
typedef CGAL::Simple_cartesian<double> Kernel;
typedef Kernel::Point_3 Point_3;
typedef CGAL::Polyhedron_3<Kernel, CGAL::Polyhedron_items_3, CGAL::HalfedgeDS_vector> Polyhedron;
int main()
{
    Point_3 p(1.0, 0.0, 0.0);
    Point_3 q(0.0, 1.0, 0.0);
    Point_3 r(0.0, 0.0, 1.0);
    Point_3 s(0.0, 0.0, 0.0);

    Polyhedron P;
    P.make_tetrahedron(p, q, r, s);
    CGAL::IO::set_ascii_mode(std::cout);
    std::copy(P.points_begin(), P.points_end(), std::ostream_iterator<Point_3>(std::cout, "\n"));
    return 0;
}
```

### 3.6 Example with Circulator Writing Object File Format (OFF)

对于环绕面的半边的逆时针循环序列，以及环绕顶点的半边的顺时针循环序列，多面体曲面提供了方便的循环器。

```cpp
#include <CGAL/Simple_cartesian.h>
#include <CGAL/Polyhedron_3.h>
#include <iostream>
typedef CGAL::Simple_cartesian<double>               Kernel;
typedef Kernel::Point_3                              Point_3;
typedef CGAL::Polyhedron_3<Kernel>                   Polyhedron;
typedef Polyhedron::Facet_iterator                   Facet_iterator;
typedef Polyhedron::Halfedge_around_facet_circulator Halfedge_facet_circulator;
int main()
{
    Point_3 p( 0.0, 0.0, 0.0);
    Point_3 q( 1.0, 0.0, 0.0);
    Point_3 r( 0.0, 1.0, 0.0);
    Point_3 s( 0.0, 0.0, 1.0);
    Polyhedron P;
    P.make_tetrahedron( p, q, r, s);

    // ++++++++++++++++++++write++++++++++++++++++++
    CGAL::IO::set_ascii_mode(std::cout);
    std::cout << "OFF" << std::endl << P.size_of_vertices() << ' ' << P.size_of_facets() << " 0" << std::endl;
    std::copy( P.points_begin(), P.points_end(), std::ostream_iterator<Point_3>( std::cout, "\n"));
    for (  Facet_iterator i = P.facets_begin(); i != P.facets_end(); ++i)
    {
        Halfedge_facet_circulator j = i->facet_begin();
        // Facets in polyhedral surfaces are at least triangles.
        CGAL_assertion( CGAL::circulator_size(j) >= 3);
        std::cout << CGAL::circulator_size(j) << ' ';
        do
        {
            std::cout << ' ' << std::distance(P.vertices_begin(), j->vertex());  // 非随机访问
        } while ( ++j != i->facet_begin());
        std::cout << std::endl;
    }
    return 0;
}
```

### 3.7 Example Using Euler Operators to Build a Cube

欧拉操作是修改多面体表面的自然方式。我们提供了一组多边形操作：split_face()、join_facet()、split_vertex()、join_vertex()、split_loop() 和 join_loop()。我们添加了进一步方便的函数，例如 split_edge()。然而，它们可以通过以上的六个操作实现。此外，我们提供更多的操作符来处理有边界边的多面体表面，例如创建和删除孔洞。我们参考了参考手册中对于欧拉操作的定义和插图。

下面的示例实现了将单元立方体附加到多面体表面的函数。为了跟踪创建立方体期间的不同步骤，一系列草图可能有助于为程序代码中出现的不同句柄添加标签。下图展示了从创建过程中选取的六个步骤，这些步骤在程序里被标注。

![图1](/assets/img/posts/2021-08-26/make_cube.png)
_图1_

```cpp
template <class Poly>
typename Poly::Halfedge_handle make_cube_3(Poly& P)
{
    // appends a cube of size [0,1]^3 to the polyhedron P.
    CGAL_precondition( P.is_valid());
    typedef typename Poly::Point_3         Point;
    typedef typename Poly::Halfedge_handle Halfedge_handle;
    Halfedge_handle h = P.make_tetrahedron( Point( 1, 0, 0),
                                           Point( 0, 0, 1),
                                           Point( 0, 0, 0),
                                           Point( 0, 1, 0));
    Halfedge_handle g = h->next()->opposite()->next();             // Fig. (a)
    P.split_edge( h->next());
    P.split_edge( g->next());
    P.split_edge( g);                                              // Fig. (b)
    h->next()->vertex()->point()     = Point( 1, 0, 1);
    g->next()->vertex()->point()     = Point( 0, 1, 1);
    g->opposite()->vertex()->point() = Point( 1, 1, 0);            // Fig. (c)
    Halfedge_handle f = P.split_facet( g->next(),
                                      g->next()->next()->next());  // Fig. (d)
    Halfedge_handle e = P.split_edge( f);
    e->vertex()->point() = Point( 1, 1, 1);                        // Fig. (e)
    P.split_facet( e, f->next()->next());                          // Fig. (f)
    CGAL_postcondition( P.is_valid());
    return h;
}
typedef CGAL::Simple_cartesian<double>     Kernel;
typedef CGAL::Polyhedron_3<Kernel>         Polyhedron;
typedef Polyhedron::Halfedge_handle        Halfedge_handle;
int main()
{
    Polyhedron P;
    Halfedge_handle h = make_cube_3(P);
    if ( P.is_tetrahedron(h))
    {
        std::cout << "R U OK" << std::endl;
        return 0;
    }
    return 1;
}
```

### 3.8 Draw a Polyhedron

通过调用 CGAL::draw<POLY>() 函数，多面体可以被可视化。调用该函数将会阻塞，当用户关闭窗口时程序继续运行。

```cpp
#include <CGAL/Exact_predicates_inexact_constructions_kernel.h>
#include <CGAL/Polyhedron_3.h>
#include <CGAL/IO/Polyhedron_iostream.h>
#include <CGAL/draw_polyhedron.h>
#include <fstream>
typedef CGAL::Exact_predicates_inexact_constructions_kernel  Kernel;
typedef CGAL::Polyhedron_3<Kernel>                       Polyhedron;
int main(int argc, char* argv[])
{
  Polyhedron P;
  std::ifstream in1((argc>1)?argv[1]:"data/cross.off");
  in1 >> P;
  CGAL::draw(P);
  return EXIT_SUCCESS;
}
```

该函数需要 CGAL_Qt5。若 CGAL_USE_BASIC_VIEWER 在编译时定义，则该函数可用。

## 4 文件 I/O

文件 I/O 当前只考虑曲面的拓扑和点的坐标。它忽略了可能的平面方程或任何用户添加的属性。

CGAL 默认支持的文件格式是 OFF，文件扩展名为 .off。修改器 set_pretty_mode() 允许输出结构化注解。否则，输出将没有注解。缺省情况下写操作不带注解。由于 OFF 是默认格式，因此提供了 iostream 操作符。

```cpp
#include <CGAL/IO/Polyhedron_iostream.h>
template <class PolyhedronTraits_3>
ostream& operator<<( ostream& out, const CGAL::Polyhedron_3<PolyhedronTraits_3>& P);
template <class PolyhedronTraits_3>
istream& operator>>( istream& in, CGAL::Polyhedron_3<PolyhedronTraits_3>& P);
```

支持写操作的其他格式有.iv、.wrl 和.obj。方便的输出函数将多面体表面写入有 CGAL 程序生成的 Geomview 过程。这些输出函数作为流操作符提供，现在作用于相应格式的流类型。

```cpp
#include <CGAL/IO/Polyhedron_inventor_ostream.h>
#include <CGAL/IO/Polyhedron_VRML_1_ostream.h>
#include <CGAL/IO/Polyhedron_VRML_2_ostream.h>
#include <CGAL/IO/Polyhedron_geomview_ostream.h>
template <class PolyhedronTraits_3>
Inventor_ostream& operator<<( Inventor_ostream& out, const CGAL::Polyhedron_3<PolyhedronTraits_3>& P);
template <class PolyhedronTraits_3>
VRML_1_ostream& operator<<( VRML_1_ostream& out, const CGAL::Polyhedron_3<PolyhedronTraits_3>& P);
template <class PolyhedronTraits_3>
VRML_2_ostream& operator<<( VRML_2_ostream& out, const CGAL::Polyhedron_3<PolyhedronTraits_3>& P);
template <class PolyhedronTraits_3>
Geomview_stream& operator<<( Geomview_stream& out, const CGAL::Polyhedron_3<PolyhedronTraits_3>& P);
```

所有这些文件格式都有一个共同点，即它们将曲面表示为一组面。每个面都是指向一组顶点的索引列表。顶点用坐标三元组表示。Polyhedron_3 的文件 I/O 对这些格式施加了某些限制。它们必须表示一个有效的多面体表面，例如一个没有孤立点的二维流型。

对于有关广义多面体网格 I/O 的更多信息，参见 Polygon Mesh IO。

## 5 扩展顶点、半边和面

在3.5节我们知道了，如何将底层半边数据结构的默认列表表示修改为向量表示。

```cpp
typedef CGAL::Polyhedron_3< Traits, CGAL::Polyhedron_items_3, CGAL::HalfedgeDS_default> Polyhedron;
```

现在我们想仔细看一下第二个模板参数 Polyhedron_items_3，它指定了使用哪种顶点、半边和面。注意 face 是用于半边数据结构的项，而 facet 仅仅在多面体表面的最顶层作为 face 的别名。

```cpp
class Polyhedron_items_3 {
    public:
    // 顶点
    template < class Refs, class Traits>
    struct Vertex_wrapper {
        typedef typename Traits::Point_3 Point;
        typedef CGAL::HalfedgeDS_vertex_base<Refs, CGAL::Tag_true, Point> Vertex;
    };
    // 半边
    template < class Refs, class Traits>
    struct Halfedge_wrapper {
        typedef CGAL::HalfedgeDS_halfedge_base<Refs> Halfedge;
    };
    // 面
    template < class Refs, class Traits>
    struct Face_wrapper {
        typedef typename Traits::Plane_3 Plane;
        typedef CGAL::HalfedgeDS_face_base<Refs, CGAL::Tag_true, Plane> Face;
    };
};
```

默认多面体使用了所有受支持的关联，顶点类中的点，以及面类中的平面方程。注意包装器类如何提供两个模板参数：Refs（稍后讨论）和 Traits（多面体曲面使用的几何特征类，这里为我们提供了点和平面方程的类型）。

使用示例代码，我们可以写出自己的 Item。相反，如果只想交换一个类，我们介绍一个更简单的方法。我们使用更简单的面，它有颜色属性而没有平面方程。为了简化顶点、半边和面的类的创建，推荐从给定基类派生。即使基类不包含任何数据，它也会提供方便的类型定义。

```cpp
template <class Refs>
struct My_face : public CGAL::HalfedgeDS_face_base<Refs> 
{
    CGAL::IO::Color color;
};
```

新的 Item 从旧的 Item 派生而来，并且包装器包含重载的面类型定义。注意包装器的名字和模板参数是固定的。如本例所示，即使不使用模板参数，也不能改变。

```cpp
struct My_items : public CGAL::Polyhedron_items_3 
{
    template <class Refs, class Traits>
    struct Face_wrapper {
    typedef My_face<Refs> Face;
    };
};
```

把所有部分放在一起。完整的示例程序说明了一旦定义了颜色属性，访问它是多么容易。

```cpp
#include <CGAL/Simple_cartesian.h>
#include <CGAL/IO/Color.h>
#include <CGAL/Polyhedron_3.h>
// A face type with a color member variable.
template <class Refs>
struct My_face : public CGAL::HalfedgeDS_face_base<Refs> 
{
    CGAL::IO::Color color;
};
// An items type using my face.
struct My_items : public CGAL::Polyhedron_items_3 
{
    template <class Refs, class Traits>
    struct Face_wrapper {
        typedef My_face<Refs> Face;
    };
};
typedef CGAL::Simple_cartesian<double>        Kernel;
typedef CGAL::Polyhedron_3<Kernel, My_items>  Polyhedron;
typedef Polyhedron::Halfedge_handle           Halfedge_handle;
int main() {
    Polyhedron P;
    Halfedge_handle h = P.make_tetrahedron();
    h->facet()->color = CGAL::IO::red();
    return 0;
}
```

现在我们回到包装器类的第一个模板参数 Refs。该参数为我们提供局部类型，使我们能够在顶点、半边和面之间进行进一步引用。这些局部类型是 Polyhedron_3::Vertex_handle、Polyhedron_3::Halfedge_handle 和 Polyhedron_3::Facet_handle，以及对应的 .._const_handle。现在，我们向面添加新的顶点引用。为了更全面的设计，可以添加封装和访问函数，但简单起见我们在这里省略。

```cpp
template <class Refs>
struct My_face : public CGAL::HalfedgeDS_face_base<Refs> 
{
    typedef typename Refs::Vertex_handle Vertex_handle;
    Vertex_handle vertex_ref;
};
```

更多高级示例可以在 halfedge data structures 找到，其进一步说明了半边数据结构的设计。

## 6 高级示例程序

### 6.1 创建细分曲面

程序从标准输入读取一个多面体表面，并将一个精细的多面体表面写入标准输出。输入和输出的格式为 OFF，文件扩展名为.off。

细化是使用$\sqrt{3}$-机制创建细分曲面的一个步骤。

该示例程序只能处理闭合曲面，但扩展示例 polyhedron_prog_subdiv_with_boundary.cpp 可以处理有边界的曲面。

```cpp
#include <CGAL/Simple_cartesian.h>
#include <CGAL/Polyhedron_3.h>
#include <iostream>
#include <algorithm>
#include <vector>
#include <cmath>
#include <fstream>
typedef CGAL::Simple_cartesian<double>                       Kernel;
typedef Kernel::Vector_3                                     Vector;
typedef Kernel::Point_3                                      Point;
typedef CGAL::Polyhedron_3<Kernel>                           Polyhedron;
typedef Polyhedron::Vertex                                   Vertex;
typedef Polyhedron::Vertex_iterator                          Vertex_iterator;
typedef Polyhedron::Halfedge_handle                          Halfedge_handle;
typedef Polyhedron::Edge_iterator                            Edge_iterator;
typedef Polyhedron::Facet_iterator                           Facet_iterator;
typedef Polyhedron::Halfedge_around_vertex_const_circulator  HV_circulator;
typedef Polyhedron::Halfedge_around_facet_circulator         HF_circulator;
void create_center_vertex( Polyhedron& P, Facet_iterator f)
{
    Vector vec( 0.0, 0.0, 0.0);
    std::size_t order = 0;
    HF_circulator h = f->facet_begin();
    do {
        vec = vec + ( h->vertex()->point() - CGAL::ORIGIN);  // Point -> Vector
        ++ order;
    } while ( ++h != f->facet_begin());
    CGAL_assertion( order >= 3);  // guaranteed by definition of polyhedron
    Point center =  CGAL::ORIGIN + (vec / static_cast<double>(order));
    Halfedge_handle new_center = P.create_center_vertex( f->halfedge());  // 为面创建中心点
    new_center->vertex()->point() = center;
}
struct Smooth_old_vertex
{
    Point operator()( const Vertex& v) const {
        CGAL_precondition((CGAL::circulator_size( v.vertex_begin()) & 1) == 0);
        std::size_t degree = CGAL::circulator_size( v.vertex_begin()) / 2;
        double alpha = ( 4.0 - 2.0 * std::cos( 2.0 * CGAL_PI / degree)) / 9.0;
        Vector vec = (v.point() - CGAL::ORIGIN) * ( 1.0 - alpha);  // 新顶点坐标
        HV_circulator h = v.vertex_begin();  // 环绕顶点 v 的半边循环器，第一条半边指向顶点 v
        do {
            vec = vec + ( h->opposite()->vertex()->point() - CGAL::ORIGIN) * alpha / static_cast<double>(degree);
            ++ h;
            CGAL_assertion( h != v.vertex_begin()); // even degree guaranteed
            ++ h;
        } while ( h != v.vertex_begin());
        return (CGAL::ORIGIN + vec);
    }
};
void flip_edge( Polyhedron& P, Halfedge_handle e)
{
    Halfedge_handle h = e->next();
    P.join_facet( e);
    P.split_facet( h, h->next()->next());
}
void subdiv( Polyhedron& P)
{
    if ( P.size_of_facets() == 0)
        return;
    // We use that new vertices/halfedges/facets are appended at the end.
    std::size_t nv = P.size_of_vertices();
    Vertex_iterator last_v = P.vertices_end();
    -- last_v;  // the last of the old vertices  用于遍历点
    Edge_iterator last_e = P.edges_end();
    -- last_e;  // the last of the old edges  用于遍历边
    Facet_iterator last_f = P.facets_end();
    -- last_f;  // the last of the old facets  用于遍历面
    Facet_iterator f = P.facets_begin();    // create new center vertices
    do {
        create_center_vertex( P, f);
    } while ( f++ != last_f);
    std::vector<Point> pts;  // 用于存储处理后的旧顶点
    pts.reserve( nv);  // get intermediate space for the new points
    ++ last_v;  // make it the past-the-end position again
    std::transform( P.vertices_begin(), last_v, std::back_inserter( pts), Smooth_old_vertex());  // 处理旧顶点
    std::copy( pts.begin(), pts.end(), P.points_begin());  // 用处理后的旧顶点覆盖旧顶点
    Edge_iterator e = P.edges_begin();              // flip the old edges
    ++ last_e;  // make it the past-the-end position again
    while ( e != last_e) {
        Halfedge_handle h = e;
        ++e;  // careful, incr. before flip since flip destroys current edge
        flip_edge( P, h);
    };
    CGAL_postcondition( P.is_valid());
}
int main(int argc, char* argv[])
{
    Polyhedron P;
    std::ifstream in1((argc>1)?argv[1]:"./data/cube.off");
    in1 >> P;
    P.normalize_border();  // 排序半边，使得非边界在边界之前
    if ( P.size_of_border_edges() != 0)
    {
        std::cerr << "The input object has border edges. Cannot subdivide." << std::endl;
        std::exit(1);
    }
    subdiv( P);
    std::cout << "R U OK" << std::endl;
    std::cout << std::setprecision(17) << P;  // 设置浮点精度
    return 0;
}
```

> reserve(n)：分配内存空间，不会减少已被元素占用的内存空间。
> std::transform()：在指定的范围内应用于给定操作，并将结果存储在另一个范围内。
> std::copy()：拷贝，注意分配足够内存空间。
> std::setprecision()：设置浮点精度。

### 6.2 使用增量生成器和修改器机制

实用程序类 Polyhedron_incremental_builder_3 可以从点列表和面列表创建多面体曲面。这对于实现通用格式读取器非常有用。这里用来生成一个三角形。

修改器机制允许以受控的方式访问多面体曲面的内部表示。修改器本质上是使用函数对象的回调机制。在成员函数 Polyhedron_3::delegate() 的最后，有效性检查（默认情况下不会调用）以一种昂贵的后置条件实现。

在这个例子中，Build_triangle 是从 Modifier_base<HalfedgeDS> 派生的函数对象。多面体的成员函数 Polyhedron_3::delegate() 接收该函数对象，并且调用它的 Modifier_base::operator()()。因此，Build_triangle 的成员函数能够在半边数据结构中创建三角形。

```cpp
#include <CGAL/Simple_cartesian.h>
#include <CGAL/Polyhedron_incremental_builder_3.h>
#include <CGAL/Polyhedron_3.h>
template <class HDS>
class Build_triangle : public CGAL::Modifier_base<HDS> {
public:
    Build_triangle() {}
    void operator()( HDS& hds) {
        // Postcondition: hds is a valid polyhedral surface.
        CGAL::Polyhedron_incremental_builder_3<HDS> B( hds, true);
        B.begin_surface( 3, 1, 6);  // V F E
        typedef typename HDS::Vertex   Vertex;
        typedef typename Vertex::Point Point;
        B.add_vertex( Point( 0, 0, 0));
        B.add_vertex( Point( 1, 0, 0));
        B.add_vertex( Point( 0, 1, 0));
        B.begin_facet();
        B.add_vertex_to_facet( 0);
        B.add_vertex_to_facet( 1);
        B.add_vertex_to_facet( 2);
        B.end_facet();
        B.end_surface();
    }
};
typedef CGAL::Simple_cartesian<double>     Kernel;
typedef CGAL::Polyhedron_3<Kernel>         Polyhedron;
typedef Polyhedron::HalfedgeDS             HalfedgeDS;
int main()
{
    Polyhedron P;
    Build_triangle<HalfedgeDS> triangle;
    P.delegate( triangle);
    CGAL_assertion( P.is_triangle( P.halfedges_begin()));
    return 0;
}
```
