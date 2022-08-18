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

类 Surface_mesh 可以通过其类成员函数、以及 CGAL and Boost Graph Library 所描述的 BGL API 使用，因为它是概念 MutableFaceGraph 和 FaceListGraph 的模型。因此，可以在表面网格上使用包 Triangulated Surface Mesh Simplification、Triangulated Surface Mesh Segmentation 和 Triangulated Surface Mesh Deformation 的算法。

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

半边存储关联面的索引。若半边 h 没有关联面，即 sm.face(h) == Surface_mesh::null_face()，则其位于边界上。若边的任一条半边位于边界上，则其位于边界上。若顶点的任一条关联半边位于边界上，则其位于边界上。

一个顶点仅仅有一个邻接半边。为了只检查邻接半边是否在边界上，调用函数 Surface_mesh::is_border(Vertex_index v, bool check_all_incident_halfedges = true) 时要令 check_all_incident_halfedges = false。

用户有责任正确设置顶点的邻接半边。

* Surface_mesh::set_vertex_halfedge_to_border_halfedge(Vertex_index v)

* Surface_mesh::set_vertex_halfedge_to_border_halfedge(Halfedge_index h)

* Surface_mesh::set_vertex_halfedge_to_border_halfedge()

## 7 表面网格和 BGL API

类 Surface_mesh 是概念 IncidenceGraph 的模型，于是可以直接使用 Dijkstra shortest path 和 Kruskal minimum spanning tree 等算法。

| BGL                                       | Surface_mesh |
| - | - |
| boost::graph_traits<G>::vertex_descriptor | Surface_mesh::Vertex_index |
| boost::graph_traits<G>::edge_descriptor   | Surface_mesh::Edge_index |
| vertices(const G& g)                      | sm.vertices() |
| edges(const G& g)                         | sm.edges() |
| vd = source(ed,g)                         | vd = sm.source(ed) |
| na                                        | n = sm.number_of_vertices() |
| n = num_vertices(g)                       | n = sm.number_of_vertices() + sm.number_of_removed_vertices() |

类 Surface_mesh 也是概念 MutableFaceGraph 的模型。

| BGL                                           | Surface_mesh |
| - | - |
| boost::graph_traits<G>::halfedge_descriptor   | Surface_mesh::Halfedge_index |
| boost::graph_traits<G>::face_descriptor       | Surface_mesh::Face_index |
| halfedges(const G& g)	                        | sm.halfedges() |
| faces(const G& g)	                            | sm.faces() |
| hd = next(hd, g)                              | hd = sm.next(hd) |
| hd = prev(hd, g)                              | hd = sm.prev(hd) |
| hd = opposite(hd,g)                           | hd = sm.opposite(hd) |
| hd = halfedge(vd,g)                           | hd = sm.halfedge(vd) |
| etc.                                          | |	

CGAL and the Boost Graph Library 中描述的 BGL API 使我们能够编写处理表面网格的几何算法，这些算法能够作用于任何 FaceGraph 或 MutableFaceGraph 的模型。

BGL 算法使用属性映射将信息与顶点和边相联系。一个重要的属性是索引，其为 0 到 num_vertices(g) 之间的整数。这允许算法创建适当大小的向量，以存储逐顶点信息。

### 7.1 例子

第一个例子展示了如何在表面网格上直接使用 Kruskal minimum spanning tree。

```cpp
#include <CGAL/Simple_cartesian.h>
#include <CGAL/Surface_mesh.h>
#include <boost/graph/kruskal_min_spanning_tree.hpp>
#include <iostream>
#include <fstream>
#include <list>
typedef CGAL::Simple_cartesian<double>                       Kernel;
typedef Kernel::Point_3                                      Point;
typedef CGAL::Surface_mesh<Point>                            Mesh;
typedef boost::graph_traits<Mesh>::vertex_descriptor vertex_descriptor;
typedef boost::graph_traits<Mesh>::vertex_iterator   vertex_iterator;
typedef boost::graph_traits<Mesh>::edge_descriptor   edge_descriptor;
void kruskal(const Mesh& sm)
{
   // We use the default edge weight which is the squared length of the edge
  std::list<edge_descriptor> mst;
  boost::kruskal_minimum_spanning_tree(sm, std::back_inserter(mst));
  std::cout << "#VRML V2.0 utf8\n"
    "Shape {\n"
    "  appearance Appearance {\n"
    "    material Material { emissiveColor 1 0 0}}\n"
    "    geometry\n"
    "    IndexedLineSet {\n"
    "      coord Coordinate {\n"
    "        point [ \n";
  vertex_iterator vb,ve;
  for(boost::tie(vb, ve) = vertices(sm); vb!=ve; ++vb){
    std::cout <<  "        " << sm.point(*vb) << "\n";
  }
  std::cout << "        ]\n"
               "     }\n"
    "      coordIndex [\n";
  for(std::list<edge_descriptor>::iterator it = mst.begin(); it != mst.end(); ++it)
  {
    edge_descriptor e = *it ;
    vertex_descriptor s = source(e,sm);
    vertex_descriptor t = target(e,sm);
    std::cout << "      " << s << ", " << t <<  ", -1\n";
  }
  std::cout << "]\n"
    "  }#IndexedLineSet\n"
    "}# Shape\n";
}
int main(int argc, char** argv)
{
  Mesh sm;
  std::string fname = argc==1?CGAL::data_file_path("meshes/knot1.off"):argv[1];
  if(!CGAL::IO::read_polygon_mesh(fname, sm))
  {
    std::cerr << "Invalid input file." << std::endl;
    return EXIT_FAILURE;
  }
  kruskal(sm);
  return 0;
}
```

第二个例子展示了如何将属性映射用于 Prim's minimum spanning tree 等算法。算法内部也使用顶点索引属性映射。

```cpp
#include <CGAL/Simple_cartesian.h>
#include <CGAL/Surface_mesh.h>
#include <iostream>
#include <fstream>
// workaround a bug in Boost-1.54
#include <CGAL/boost/graph/dijkstra_shortest_paths.h>
#include <boost/graph/prim_minimum_spanning_tree.hpp>
typedef CGAL::Simple_cartesian<double>                       Kernel;
typedef Kernel::Point_3                                      Point;
typedef CGAL::Surface_mesh<Point>                            Mesh;
typedef boost::graph_traits<Mesh>::vertex_descriptor vertex_descriptor;
int main(int argc, char* argv[])
{
  Mesh sm;
  std::string fname = argc==1?CGAL::data_file_path("meshes/knot1.off"):argv[1];
  if(!CGAL::IO::read_polygon_mesh(fname, sm))
  {
    std::cerr << "Invalid input file." << std::endl;
    return EXIT_FAILURE;
  }
  Mesh::Property_map<vertex_descriptor,vertex_descriptor> predecessor;
  predecessor = sm.add_property_map<vertex_descriptor,vertex_descriptor>("v:predecessor").first;
  boost::prim_minimum_spanning_tree(sm, predecessor, boost::root_vertex(*vertices(sm).first));
  std::cout << "#VRML V2.0 utf8\n"
    "DirectionalLight {\n"
    "direction 0 -1 0\n"
    "}\n"
    "Shape {\n"
    "  appearance Appearance {\n"
    "    material Material { emissiveColor 1 0 0}}\n"
    "    geometry\n"
    "    IndexedLineSet {\n"
    "      coord Coordinate {\n"
    "        point [ \n";
  for(vertex_descriptor vd : vertices(sm)){
    std::cout <<  "        " << sm.point(vd) << "\n";
  }
  std::cout << "        ]\n"
    "     }\n"
    "      coordIndex [\n";
  for(vertex_descriptor vd : vertices(sm)){
    if(predecessor[vd]!=vd){
      std::cout << "      " << std::size_t(vd) << ", " << std::size_t(predecessor[vd]) <<  ", -1\n";
    }
  }
  std::cout << "]\n"
    "  }#IndexedLineSet\n"
    "}# Shape\n";
  sm.remove_property_map(predecessor);
  return 0;
}
```

## 8 表面网格 I/O

作为概念 FaceGraph 的模型，类 Surface_mesh 可以使用多种不同的文件类型读取或写入。

## 9 内存管理

内存管理是半自动的。当更多的元素添加到结构中时，内存会增长，但当移除元素时，内存不会缩减。

当添加元素并且底层向量的容量耗尽时，该向量重新分配内存。由于描述符本质上是索引，它们在重新分配后引用相同的元素。

移除元素时，该元素仅标记为已移除，并被放入自由链表。当向表面网格添加元素时，如果自由链表不是空的，将从自由链表中获取内存。

> 自由链表？

对于所有元素，我们提供函数以获得已使用元素的数量、已使用且已移除元素的数量。对于顶点，这样的函数分别为 Surface_mesh::number_of_vertices() 和 Surface_mesh::number_of_removed_vertices()。

迭代器（如 Surface_mesh::Vertex_iterator）仅枚举未标记为已移除的元素。

要真正缩减使用的内存，必须调用 Surface_mesh::collect_garbage()。垃圾回收还会压缩与表面网格有关的属性。

注意，通过垃圾收集，元素会获得新的索引。如果保留顶点描述符，它们很可能不再引用正确的顶点。

### 9.1 例子

```cpp
#include <iostream>
#include <CGAL/Simple_cartesian.h>
#include <CGAL/Surface_mesh.h>
typedef CGAL::Simple_cartesian<double> K;
typedef CGAL::Surface_mesh<K::Point_3> Mesh;
typedef Mesh::Vertex_index vertex_descriptor;
int main()
{
  Mesh m;
  Mesh::Vertex_index u;
  for(unsigned int i=0; i < 5; ++i){
    Mesh::Vertex_index v = m.add_vertex(K::Point_3(0,0,i+1));
    if(i==2) u=v;
  }
  m.remove_vertex(u);
  std::cout << "After insertion of 5 vertices and removal of the 3. vertex\n"
            << "# vertices  / # vertices + # removed vertices = "
            << m.number_of_vertices()
            << " / " << m.number_of_vertices() + m.number_of_removed_vertices() << std::endl;
  std::cout << "Iterate over vertices\n";
  {
    for(vertex_descriptor vd : m.vertices()){
      std::cout << m.point(vd) << std::endl;
    }
  }
  // The status of being used or removed is stored in a property map
  Mesh::Property_map<Mesh::Vertex_index,bool> removed
    = m.property_map<Mesh::Vertex_index,bool>("v:removed").first;
  std::cout << "\nIterate over vertices and deleted vertices\n"
            << "# vertices / # vertices + # removed vertices = "
            << m.number_of_vertices()
            << " / " << m.number_of_vertices() + m.number_of_removed_vertices() << std::endl;
  {
    unsigned int i = 0, end = m.number_of_vertices() + m.number_of_removed_vertices();
    for( ; i < end; ++i) {
      vertex_descriptor vh(i);
      assert(m.is_removed(vh) == removed[vh]);
      std::cout << m.point(vh) << ((m.is_removed(vh)) ? "  R\n" : "\n");
    }
  }
  m.collect_garbage();
  std::cout << "\nAfter garbage collection\n"
            << "# vertices / # vertices + # removed vertices = "
            << m.number_of_vertices()
            << " / " << m.number_of_vertices() + m.number_of_removed_vertices() << std::endl;
  {
   unsigned int i = 0, end = m.number_of_vertices() + m.number_of_removed_vertices();
    for( ; i < end; ++i) {
      vertex_descriptor vh(i);
      std::cout << m.point(vh) << ((m.is_removed(vh)) ? "  R\n" : "\n");
    }
  }
  return 0;
}
```

```cpp
// 输出
After insertion of 5 vertices and removal of the 3. vertex
# vertices  / # vertices + # removed vertices = 4 / 5
Iterate over vertices
0 0 1
0 0 2
0 0 4
0 0 5

Iterate over vertices and deleted vertices
# vertices / # vertices + # removed vertices = 4 / 5
0 0 1
0 0 2
0 0 3  R
0 0 4
0 0 5

After garbage collection
# vertices / # vertices + # removed vertices = 4 / 4
0 0 1
0 0 2
0 0 5
0 0 4
```

## 10 绘制表面网格

略

## 11 实现细节

我们使用 boost::uint32_t 作为索引整数类型，在 64 位操作系统上大小仅为指针的一半，支持两亿个元素的网格。

我们使用 std::vector 存储属性。因此，通过访问属性映射的首个元素的地址，可以访问底层原始数组。这可能很有用，例如，将顶点数组传递给 OpenGL。

我们使用自由链表存储已移除的元素。这意味着，当移除顶点并随后调用 add_vertex 时，将重用已移除元素的内存。因此第 n 个插入的元素索引不一定为 n-1，并且迭代元素不一定以插入顺序枚举。

## 12 实现历史

略