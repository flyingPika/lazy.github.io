---
title: CGAL 5.3 半边数据结构
date: 2021-08-26 00:00:00 +0800
categories: [CG, GEOMETRY]
tags: [cgal]     # TAG names should always be lowercase
math: false
pin: false
---

<https://doc.cgal.org/latest/HalfedgeDS/index.html#Chapter_Halfedge_Data_Structures>

## 1 介绍

半边数据结构是以边为中心的数据结构，能够维护顶点、边和面的关联信息。每个边被分解为两个相反方向的半边。每个半边存储一个关联面和关联顶点。对于每个面和每个顶点，存储一个关联半边。半边数据结构的简化变体可能省略一些信息。

![图1](/assets/img/posts/2021-08-26/halfedge_small.png)
_图1_

半边数据结构是一种组合数据结构，几何解释由构建在半边数据结构之上的类添加。这些类可能比直接使用半边数据结构更方便，因为半边数据结构意味着作为一个实现层。参见多面体表面一章中的 Polyhedron_3 类。

## 2 软件设计

![图2](/assets/img/posts/2021-08-26/hds_design_col.png)
_图2_

上图说明了软件设计的三层职责，以 Polyhedron_3 作为顶层的例子。Item 提供信息实际存储的空间。半边需要提供下一个半边和相反半边的引用，对于前一个半边、关联顶点和关联面的引用是可选的。顶点和面可以为空，可选择提供关联半边的索引。HalfedgeDataStructure 和 Polyhedron 支持前面提到的选项。此外，Item 类可以使用任意属性和成员函数进行扩展，这将通过继承提升为用于 Polyhedron 的实际的类。

顶点、半边和面以 Item 类局部类型的形式传递给 HalfedgeDataStructure 和 Polyhedron。顶点、半边和面的实现满足需求的强制性部分。它们可以被用户用作扩展的基类。更丰富的实现作为默认实现被提供，对于 Polyhedron，它们提供所有可选的关联、顶点类型的三维点和面类型的平面方程。

概念 HalfedgeDS 负责 Item 的组织存储，目前提供了双向链表和 vector 的实现。HalfedgeDS 定义了属于 Item 的句柄和迭代器，这些类型被提升为 Item 本身的声明，并且被用于提供关联 Item 的引用。类型提升通过 Item 的模板参数 Refs 完成。HalfedgeDataStructure 提供成员函数来插入删除 Item、遍历所有 Item 并且提供对 Item 的访问。

概念 HalfedgeDS 有两种不同模型：HalfedgeDS_list 和 HalfedgeDS_vector。我们保证接口简单，并且分解共同功能到单独的助手类：HalfedgeDS_decorator、HalfedgeDS_const_decorator 和 HalfedgeDS_items_decorator。这些助手类包含有助于实现下一层（例如 Polyhedron）操作的操作。此外，助手类包含自适应功能。例如，如果半边不提供 HalfedgeDSHalfedge::prev() 成员函数，助手类的 HalfedgeDS_items_decorator::find_prev() 成员函数会沿面的正向搜索前一半边。但如果提供 HalfedgeDSHalfedge::prev() 成员函数，HalfedgeDS_items_decorator::find_prev() 将简单地调用它。这种区别在编译时通过编译时标签的技术解决。

Polyhedron_3 类作为第三层示例添加了几何解释，提供了易于使用的高级功能接口，并且统一了对底层灵活性的访问。它将 face 重命名为 facet。接口旨在保护内部表示的完整性，存储在 Item 中的句柄不再由用户直接编写。Polyhedron 添加了方便有效的循环器，用于访问围绕顶点或面的边的循环序列。为了实现这一点，Polyhedron_3 得到 Item 提供的顶点、半边和面。这些新的 Item 实际在 HalfedgeDS 中使用，这给了我们在设计中连贯的类型结构，特别是与之前的设计相比。

## 3 示例程序

### 3.1 The Default Halfedge Data Structure

下面的例子使用了默认的 HalfedgeDS 和 Decorator。默认的 HalfedgeDS 使用基于列表的表示。Item 的所有关联和一种顶点的点类型被定义。简单的 Traits 类提供用于点的类型。程序创建一个由两个半边、一个顶点和两个面组成的环，并检查其有效性。

![图3](/assets/img/posts/2021-08-26/loop.png)
_图3_

```cpp
#include <CGAL/HalfedgeDS_default.h>
#include <CGAL/HalfedgeDS_decorator.h>
struct Traits { typedef int Point_2; };
typedef CGAL::HalfedgeDS_default<Traits> HDS;
typedef CGAL::HalfedgeDS_decorator<HDS> Decorator;
int main() {
    HDS hds;
    Decorator decorator(hds);
    decorator.create_loop();
    CGAL_assertion( decorator.is_valid());
    return 0;
}
```

### 3.2 A Minimal Halfedge Data Structure

下面的例子使用了 HalfedgeDS_min_items 和默认的 HalfedgeDS。结果是一个数据结构，其只维护包含 next 和 opposite 指针的半边，不存储顶点和面。这种数据结构表示无向图。

```cpp
#include <CGAL/HalfedgeDS_min_items.h>
#include <CGAL/HalfedgeDS_default.h>
#include <CGAL/HalfedgeDS_decorator.h>
// no traits needed, argument can be arbitrary dummy.
typedef CGAL::HalfedgeDS_default<int, CGAL::HalfedgeDS_min_items> HDS;
typedef CGAL::HalfedgeDS_decorator<HDS>  Decorator;
int main() {
    HDS hds;
    Decorator decorator(hds);
    decorator.create_loop();
    CGAL_assertion( decorator.is_valid());
    return 0;
}
```

### 3.3 The Default with a Vector Instead of a List

默认 HalfedgeDS 使用链表和最大基类。这里我们将列表改为 vector 表示。同样，简单的 Traits 类提供用于点的类型。注意，对于 vector 存储，HalfedgeDS 的大小应该预先保留，可以使用示例中的构造函数，也可以使用成员函数 reserve()。以后可以改变 HalfedgeDS 的大小通过进一步调用成员函数 reserve()，但前提是 HalfedgeDS 处于一致（即有效）状态。

```cpp
#include <CGAL/HalfedgeDS_items_2.h>
#include <CGAL/HalfedgeDS_vector.h>
#include <CGAL/HalfedgeDS_decorator.h>
struct Traits { typedef int Point_2; };
typedef CGAL::HalfedgeDS_vector< Traits, CGAL::HalfedgeDS_items_2> HDS;
typedef CGAL::HalfedgeDS_decorator<HDS>  Decorator;
int main() {
    HDS hds(1,2,2);
    Decorator decorator(hds);
    decorator.create_loop();
    CGAL_assertion( decorator.is_valid());
    return 0;
}
```

### 3.4 Example Adding Color to Faces

这个例子重用了可用于面的基类，并且添加了成员变量 color。

```cpp
#include <CGAL/HalfedgeDS_items_2.h>
#include <CGAL/HalfedgeDS_default.h>
#include <CGAL/IO/Color.h>
// A face type with a color member variable.
template <class Refs>
struct My_face : public CGAL::HalfedgeDS_face_base<Refs> {
    CGAL::IO::Color color;
    My_face() {}
    My_face( CGAL::IO::Color c) : color(c) {}
};
// An items type using my face.
struct My_items : public CGAL::HalfedgeDS_items_2 {
    template <class Refs, class Traits>
    struct Face_wrapper {
        typedef My_face<Refs> Face;
    };
};
struct My_traits { // arbitrary point type, not used here.
    typedef int  Point_2;
};
typedef CGAL::HalfedgeDS_default<My_traits, My_items> HDS;
typedef HDS::Face                                     Face;
typedef HDS::Face_handle                              Face_handle;
int main() {
    HDS hds;
    Face_handle f = hds.faces_push_back( Face( CGAL::IO::red()));
    f->color = CGAL::IO::blue();
    CGAL_assertion( f->color == CGAL::IO::blue());
    return 0;
}
```

### 3.5 Example Defining a More Compact Halfedge

这里展示的 HalfedgeDS 空间效率稍低，例如翼边数据结构、DCEL 或四边数据结构的变体。另一方面，它在遍历期间不需要任何搜索操作。

下面的例子使用传统 C 技术将遍历时间交换为紧凑的存储表示。思想如下：HalfedgeDS 两两分配半边。关于基于 vector 的数据结构，这意味着在 C 指针算法中，半边和其相对半边的差的绝对值总为1。我们可以使用一个编码差符号的位代替 opposite 指针。我们将这个位作为最低有效位存储在下一个半边句柄中。此外，我们我们不实现前一个半边的指针。剩下的是每个半边三个指针。

我们使用静态成员函数 HalfedgeDS::halfedge_handle() 从指针转换到半边句柄。相同的解决方案可以用于基于链表的 HalfedgeDS。这里是基于 vector 的 HalfedgeDS 的例子。

```cpp
#include <CGAL/HalfedgeDS_items_2.h>
#include <CGAL/HalfedgeDS_vector.h>
#include <CGAL/HalfedgeDS_decorator.h>
#include <cstddef>
// Define a new halfedge class. We assume that the Halfedge_handle can
// be created from a pointer (e.g. the HalfedgeDS is based here on the
// In_place_list or a std::vector with such property) and that halfedges
// are allocated in pairs. We encode the opposite pointer in a single bit,
// which is stored in the lower bit of the next-pointer. We use the
// static member function HDS::halfedge_handle to translate pointer to
// handles.
template <class Refs>
class My_halfedge {
public:
    typedef Refs                                 HDS;
    typedef My_halfedge<Refs>                    Base_base;
    typedef My_halfedge<Refs>                    Base;
    typedef My_halfedge<Refs>                    Self;
    typedef CGAL::Tag_false                      Supports_halfedge_prev;
    typedef CGAL::Tag_true                       Supports_halfedge_vertex;
    typedef CGAL::Tag_true                       Supports_halfedge_face;
    typedef typename Refs::Vertex_handle         Vertex_handle;
    typedef typename Refs::Vertex_const_handle   Vertex_const_handle;
    typedef typename Refs::Halfedge              Halfedge;
    typedef typename Refs::Halfedge_handle       Halfedge_handle;
    typedef typename Refs::Halfedge_const_handle Halfedge_const_handle;
    typedef typename Refs::Face_handle           Face_handle;
    typedef typename Refs::Face_const_handle     Face_const_handle;
private:
    std::ptrdiff_t  nxt;
public:
    My_halfedge() : nxt(0), f( Face_handle()) {}
    Halfedge_handle opposite() {
        // Halfedge could be different from My_halfedge (e.g. pointer for
        // linked list). Get proper handle from 'this' pointer first, do
        // pointer arithmetic, then convert pointer back to handle again.
        Halfedge_handle h = HDS::halfedge_handle(this); // proper handle
        if ( nxt & 1)
            return HDS::halfedge_handle( &* h + 1);
        return HDS::halfedge_handle( &* h - 1);
    }
    Halfedge_const_handle opposite() const { // same as above
        Halfedge_const_handle h = HDS::halfedge_handle(this); // proper handle
        if ( nxt & 1)
            return HDS::halfedge_handle( &* h + 1);
        return HDS::halfedge_handle( &* h - 1);
    }
    Halfedge_handle next() {
        return HDS::halfedge_handle((Halfedge*)(nxt & (~ std::ptrdiff_t(1))));
    }
    Halfedge_const_handle next() const {
        return HDS::halfedge_handle((const Halfedge*)
                                    (nxt & (~ std::ptrdiff_t(1))));
    }
    void  set_opposite( Halfedge_handle h) {
        CGAL_precondition(( &* h - 1 == &* HDS::halfedge_handle(this)) ||
                          ( &* h + 1 == &* HDS::halfedge_handle(this)));
        if ( &* h - 1 == &* HDS::halfedge_handle(this))
            nxt |= 1;
        else
            nxt &= (~ std::ptrdiff_t(1));
    }
    void  set_next( Halfedge_handle h) {
        CGAL_precondition( ((std::ptrdiff_t)(&*h) & 1) == 0);
        nxt = ((std::ptrdiff_t)(&*h)) | (nxt & 1);
    }
private:    // Support for the Vertex_handle.
    Vertex_handle    v;
public:
    // the incident vertex.
    Vertex_handle         vertex()                     { return v; }
    Vertex_const_handle   vertex() const               { return v; }
    void                  set_vertex( Vertex_handle w) { v = w; }
private:
    Face_handle      f;
public:
    Face_handle           face()                       { return f; }
    Face_const_handle     face() const                 { return f; }
    void                  set_face( Face_handle g)     { f = g; }
    bool                  is_border() const { return f == Face_handle(); }
};
// Replace halfedge in the default items type.
struct My_items : public CGAL::HalfedgeDS_items_2 {
    template <class Refs, class Traits>
    struct Halfedge_wrapper {
        typedef My_halfedge<Refs> Halfedge;
    };
};
struct Traits { typedef int Point_2; };
typedef CGAL::HalfedgeDS_vector<Traits, My_items> HDS;
typedef CGAL::HalfedgeDS_decorator<HDS>  Decorator;
int main() {
    HDS hds(1,2,2);
    Decorator decorator(hds);
    decorator.create_loop();
    CGAL_assertion( decorator.is_valid());
    return 0;
}
```

### 3.6 Example Using the Halfedge Iterator

在默认的 HalfedgeDS 中创建了两条边。使用半边迭代器计数半边。

```cpp
#include <CGAL/HalfedgeDS_default.h>
#include <CGAL/HalfedgeDS_decorator.h>
struct Traits { typedef int Point_2; };
typedef CGAL::HalfedgeDS_default<Traits> HDS;
typedef CGAL::HalfedgeDS_decorator<HDS>  Decorator;
typedef HDS::Halfedge_iterator           Iterator;
int main() {
    HDS hds;
    Decorator decorator(hds);
    decorator.create_loop();
    decorator.create_segment();
    CGAL_assertion( decorator.is_valid());
    int n = 0;
    for ( Iterator i = hds.halfedges_begin(); i != hds.halfedges_end(); ++i )
        ++n;
    CGAL_assertion( n == 4);  // == 2 edges
    return 0;
}
```

### 3.7 Example for an Adapter to Build an Edge Iterator

在默认的 HalfedgeDS 中创建了两条边。适配器 N_step_adaptor 用于声明计数边的边迭代器。

```cpp
#include <CGAL/HalfedgeDS_default.h>
#include <CGAL/HalfedgeDS_decorator.h>
#include <CGAL/N_step_adaptor.h>
struct Traits { typedef int Point_2; };
typedef CGAL::HalfedgeDS_default<Traits>            HDS;
typedef CGAL::HalfedgeDS_decorator<HDS>             Decorator;
typedef HDS::Halfedge_iterator                      Halfedge_iterator;
typedef CGAL::N_step_adaptor< Halfedge_iterator, 2> Iterator;
int main() {
    HDS hds;
    Decorator decorator(hds);
    decorator.create_loop();
    decorator.create_segment();
    CGAL_assertion( decorator.is_valid());
    int n = 0;
    for ( Iterator e = hds.halfedges_begin(); e != hds.halfedges_end(); ++e)
        ++n;
    CGAL_assertion( n == 2);  // == 2 edges
    return 0;
}
```
