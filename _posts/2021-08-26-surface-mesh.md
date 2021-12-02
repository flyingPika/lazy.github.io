---
title: CGAL 5.3 表面网格
date: 2021-08-26 00:02:00 +0800
categories: [CG, GEOMETRY]
tags: [cgal]     # TAG names should always be lowercase
math: false
pin: false
---

<https://doc.cgal.org/latest/Surface_mesh/index.html#Chapter_3D_Surface_mesh>

类 Surface_mesh 是一个半边数据结构的实现，用于表示多面体表面。它是包 Halfedge Data Structures 和包 3D Polyhedral Surface 的替代。主要区别在于它基于索引而不是指针。此外，向顶点、半边、边和面添加信息的机制更加简单，在运行时完成而不是编译时。

因为该数据结构使用整数索引作为顶点、半边、边和面的描述符，所以它比基于64位指针的版本具有更低的内存占用。由于索引是连续的，它们可以用作存储属性的 vector 的索引。

当删除元素时，它们只被标记为已删除，要真正删除它们必须调用垃圾收集函数。

类 Surface_mesh 可以通过其类成员函数、以及 CGAL 包和 Boost Graph Library 所描述的 BGL API 使用，因为它是概念 MutableFaceGraph 和 FaceListGraph 的模型。因此，可以在表面网格上使用包 Triangulated Surface Mesh Simplification、Triangulated Surface Mesh Segmentation 和 Triangulated Surface Mesh Deformation 的算法。

## 1 使用

类 Surface_mesh 提供四个表示半边数据结构基本元素的内部类：

* Surface_mesh::Vertex_index

* Surface_mesh::Halfedge_index

* Surface_mesh::Face_index

* Surface_mesh::Edge_index

这些类型仅仅是整数的包装，并且主要目的是保证类型安全。Surface_mesh 中新元素的添加和移除，通过一组不维护连通性的低级别函数实现。一个例外是 Surface_mesh::add_face()，其向网格添加（由顶点序列定义的）新面，如果操作不是拓扑有效的则执行失败。在这种情况下，返回的 Face_index 是 Surface_mesh::null_face()。

```cpp
typedef Surface_mesh<Point> Mesh;
Mesh m;
Mesh::Vertex_index u = m.add_vertex(Point(0,1,0));
Mesh::Vertex_index v = m.add_vertex(Point(0,0,0));
Mesh::Vertex_index w = m.add_vertex(Point(1,0,0));
m.add_face(u, v, w);
```

~~由于 Surface_mesh 是基于索引的 Vertex_index，Halfedge_index、Edge_index 和 Face_index 没有访问连通性或性质的成员函数~~。必须使用从中创建 Surface_mesh 实例的函数来获取此信息。

### 1.1 例子

下面的例子展示了如何通过添加两个面创建一个简单的 Surface_mesh，以及如何检查面被正确地添加到网格。

```cpp
// check_orientation.cpp
typedef CGAL::Simple_cartesian<double> K;
typedef CGAL::Surface_mesh<K::Point_3> Mesh;
typedef Mesh::Vertex_index vertex_descriptor;
typedef Mesh::Face_index face_descriptor;
int main()
{
    Mesh m;
    // Add the points as vertices
    vertex_descriptor u = m.add_vertex(K::Point_3(0,1,0));
    vertex_descriptor v = m.add_vertex(K::Point_3(0,0,0));
    vertex_descriptor w = m.add_vertex(K::Point_3(1,1,0));
    vertex_descriptor x = m.add_vertex(K::Point_3(1,0,0));
    m.add_face(u,v,w);  // u->v
    face_descriptor f = m.add_face(u,v,x);  // u->v
    if(f == Mesh::null_face())
    {
        std::cerr<<"The face could not be added because of an orientation error."<<std::endl;
        f = m.add_face(u,x,v);  // v->u
        assert(f != Mesh::null_face());
    }
    return 0;
}
```

## 2 连通性

表面网格是以边为中心的数据结构，能够维护顶点、边和面的关联信息。每条边使用两个方向相反的半边表示。每个半边存储关联面和关联顶点的索引，此外，它将存储与关联面相邻的上一条半边和下一条半边。每个面和每个顶点存储一条关联半边。半边不存储相反半边的索引，因为 Surface_mesh 在内存中连续存储相反的半边。

下图说明了允许在表面网格中导航的函数：Surface_mesh::opposite()，Surface_mesh::next()，Surface_mesh::prev()，Surface_mesh::target() 和 Surface_mesh::face()。另外，函数 Surface_mesh::halfedge() 允许获得与顶点和面相关联的半边。或者，可以使用定义在 CGAL and the Boost Graph Library 中的同名自由函数。

与面关联的半边构成环路。

连通性不允许用孔表示面。

## 3 范围和迭代器

Surface_mesh 提供迭代器范围以枚举所有顶点、半边、边和面。它提供与 Boost.Range 库兼容的、返回元素范围的成员函数。

### 3.1 例子

下面的例子展示了如何从范围中获取迭代器类型、获取开始迭代器和结束迭代器的备选方案以及基于范围的循环的备选方案。

```cpp
// sm_iterators.cpp
#include <vector>
#include <CGAL/Simple_cartesian.h>
#include <CGAL/Surface_mesh.h>
typedef CGAL::Simple_cartesian<double> K;
typedef CGAL::Surface_mesh<K::Point_3> Mesh;
typedef Mesh::Vertex_index vertex_descriptor;
typedef Mesh::Face_index face_descriptor;
int main()
{
    Mesh m;
    // u            x
    // +------------+
    // |            |
    // |            |
    // |      f     |
    // |            |
    // |            |
    // +------------+
    // v            w
    // Add the points as vertices
    vertex_descriptor u = m.add_vertex(K::Point_3(0,1,0));
    vertex_descriptor v = m.add_vertex(K::Point_3(0,0,0));
    vertex_descriptor w = m.add_vertex(K::Point_3(1,0,0));
    vertex_descriptor x = m.add_vertex(K::Point_3(1,1,0));
    /* face_descriptor f = */ m.add_face(u,v,w,x);
    {
        std::cout << "all vertices " << std::endl;
        // The vertex iterator type is a nested type of the Vertex_range
        Mesh::Vertex_range::iterator  vb, ve;
        Mesh::Vertex_range r = m.vertices();
        // The iterators can be accessed through the C++ range API
        vb = r.begin();
        ve = r.end();
        // or the boost Range API
        vb = boost::begin(r);
        ve = boost::end(r);
        // or with boost::tie, as the CGAL range derives from std::pair
        for(boost::tie(vb, ve) = m.vertices(); vb != ve; ++vb){
            std::cout << *vb << std::endl;
        }
        // Instead of the classical for loop one can use
        // the boost macro for a range
        // ?????
        for(vertex_descriptor vd : m.vertices()){
            std::cout << vd << std::endl;
        }
        // or the C++11 for loop. Note that there is a ':' and not a ',' as in BOOST_FOREACH
        for(vertex_descriptor vd : m.vertices()){
            std::cout << vd << std::endl;
        }
    }
    return 0;
}
```

## 4 循环器

在包 CGAL and the Boost Graph Library 中，围绕面和顶点的循环器以类模板的形式提供。

围绕面的循环器本质上调用 Surface_mesh::next()，逆时针绕面从一个半边到另一个半边。~~当取消引用时，返回半边或者关联顶点或者相对的面。~~

* CGAL::Halfedge_around_face_circulator<Mesh>

* CGAL::Vertex_around_face_circulator<Mesh>

* CGAL::Face_around_face_circulator<Mesh>

围绕目标顶点的循环器本质上调用 Surface_mesh::opposite(Surface_mesh::next())，顺时针绕目标顶点从一个半边到另一个半边。

* CGAL::Halfedge_around_target_circulator<Mesh>

* CGAL::Vertex_around_target_circulator<Mesh>

* CGAL::Face_around_target_circulator<Mesh>

所有循环器均为双向循环器。除此之外，它们还支持到布尔类型的转换，以更方便地检查空值。

### 4.1 例子

下面的例子展示了如何枚举围绕给定目标半边的顶点。第二个循环表明，这些循环器类型，都有用以创建迭代范围的等价迭代器和自由函数。

```cpp
// sm_circulators.cpp
#include <CGAL/Simple_cartesian.h>
#include <CGAL/Surface_mesh.h>
#include <vector>
typedef CGAL::Simple_cartesian<double> K;
typedef CGAL::Surface_mesh<K::Point_3> Mesh;
typedef Mesh::Vertex_index vertex_descriptor;
typedef Mesh::Face_index face_descriptor;
int main()
{
    Mesh m;
    // u            x
    // +------------+
    // |            |
    // |            |
    // |      f     |
    // |            |
    // |            |
    // +------------+
    // v            w
    // Add the points as vertices
    vertex_descriptor u = m.add_vertex(K::Point_3(0,1,0));
    vertex_descriptor v = m.add_vertex(K::Point_3(0,0,0));
    vertex_descriptor w = m.add_vertex(K::Point_3(1,0,0));
    vertex_descriptor x = m.add_vertex(K::Point_3(1,1,0));
    face_descriptor f = m.add_face(u,v,w,x);
    {
        std::cout << "vertices around vertex " << v << std::endl;
        CGAL::Vertex_around_target_circulator<Mesh> vbegin(m.halfedge(v),m);  // 顶点转半边
        CGAL::Vertex_around_target_circulator<Mesh> done(vbegin);
        do {
            std::cout << *vbegin++ << std::endl;  // 解引用优先级更低
        } while(vbegin != done);
    }
    {
        std::cout << "vertices around face " << f << std::endl;
        CGAL::Vertex_around_face_iterator<Mesh> vbegin, vend;
        for(boost::tie(vbegin, vend) = vertices_around_face(m.halfedge(f), m);
             vbegin != vend;
             ++vbegin){
            std::cout << *vbegin << std::endl;
        }
    }
    // or the same again, but directly with a range based loop
    for(vertex_descriptor vd : vertices_around_face(m.halfedge(f), m)){
        std::cout << vd << std::endl;
    }
    return 0;
}
```

## 5 属性

Surface_mesh 提供了一种机制，可以在运行时为顶点、半边、边和面指定新属性。每个属性由字符串和键类型标识。给定属性的值存储为连续的内存块。每当向数据结构添加键类型的元素时，或者当 Surface_mesh::collect_garbage() 执行时，对属性的引用无效。删除元素后，元素的属性将继续存在。试图通过无效元素访问属性会导致未定义的行为。

> 属性由元素标识。

默认只维护一个属性，即“v:point”。当通过 Surface_mesh::add_vertex() 添加新点到数据结构时，必须提供属性的值。使用 Surface_mesh::points() 或 Surface_mesh::point(Surface_mesh::Vertex_index v) 可以直接访问属性。

移除元素时，仅将元素标记为移除，当 Surface_mesh::collect_garbage() 执行时才真正移除元素。垃圾回收也会真正移除这些元素的属性。

连通性也存储在属性中，即属性“v:connectivity”、“h:connectivity”和“f:connectivity”。

可以方便地移除用户添加的属性映射，通过索引类型（Surface_mesh::remove_property_maps\<T>()）或者移除全部（Surface_mesh::remove_all_property_maps()）。

要清除网格，你可能获得所有属性映射都被移除的网格（Surface_mesh::clear()），或者保留属性映射的网格（Surface_mesh::clear_without_removing_property_maps()）。注意在这两种情况下，“v::point”属性映射将被保留，并且保留对它的引用是安全的。

### 5.1 例子

这个例子展示了如何使用属性系统的最常用特性。

```cpp
// sm_properties.cpp
typedef CGAL::Simple_cartesian<double> K;
typedef CGAL::Surface_mesh<K::Point_3> Mesh;
typedef Mesh::Vertex_index vertex_descriptor;
typedef Mesh::Face_index face_descriptor;
int main()
{
    Mesh m;
    vertex_descriptor v0 = m.add_vertex(K::Point_3(0,2,0));
    vertex_descriptor v1 = m.add_vertex(K::Point_3(2,2,0));
    vertex_descriptor v2 = m.add_vertex(K::Point_3(0,0,0));
    vertex_descriptor v3 = m.add_vertex(K::Point_3(2,0,0));
    vertex_descriptor v4 = m.add_vertex(K::Point_3(1,1,0));
    m.add_face(v3, v1, v4);
    m.add_face(v0, v4, v1);
    m.add_face(v0, v2, v4);
    m.add_face(v2, v3, v4);
    // give each vertex a name, the default is empty
    Mesh::Property_map<vertex_descriptor,std::string> name;
    bool created;
    boost::tie(name, created) = m.add_property_map<vertex_descriptor,std::string>("v:name","");
    assert(created);
    // add some names to the vertices
    name[v0] = "hello";
    name[v2] = "world";
    {
        // You get an existing property, and created will be false
        Mesh::Property_map<vertex_descriptor,std::string> name;
        bool created;
        boost::tie(name, created) = m.add_property_map<vertex_descriptor,std::string>("v:name", "");
        assert(! created);
    }
    //  You can't get a property that does not exist
    Mesh::Property_map<face_descriptor,std::string> gnus;
    bool found;
    boost::tie(gnus, found) = m.property_map<face_descriptor,std::string>("v:gnus");
    assert(! found);
    // retrieve the point property for which exists a convenience function
    Mesh::Property_map<vertex_descriptor, K::Point_3> location = m.points();
    for(vertex_descriptor vd : m.vertices()) {
        std::cout << name[vd] << " @ " << location[vd] << std::endl;
    }
    std::vector<std::string> props = m.properties<vertex_descriptor>();  // 顶点属性
    for(std::string p : props){
        std::cout << p << std::endl;
    }
    // delete the string property again
    m.remove_property_map(name);
    return 0;
}
```

```cpp
// 输出
hello @ 0 2 0
 @ 2 2 0
world @ 0 0 0
 @ 2 0 0
 @ 1 1 0
v:connectivity
v:point
v:removed
v:name
```

## 6 边界

## 7 表面网格和 BGL API

### 7.1 例子

## 8 表面网格 I/O

## 9 内存管理

### 9.1 例子

## 10 画表面网格

## 11 实现细节

## 12 实现历史
