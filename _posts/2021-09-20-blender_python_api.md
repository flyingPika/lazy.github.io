---
title: Blender 2.93 Python API
date: 2021-09-20 00:01:00 +0800
categories: [CG, BLENDER]
tags: [blender]     # TAG names should always be lowercase
math: false
pin: false
---

<https://docs.blender.org/api/current/info_quickstart.html>

# Quickstart: New to Blender or scripting and want to get your feet wet?

Blender Python API 特性：

* 编辑用户界面可以编辑的任何数据（场景、网格、粒子等）。

* 修改用户首选项、键盘映射和主题。

* 使用个人设置运行工具。

* 创建用户界面元素。

* 创建新工具。

* 创建交互式工具。

* 创建与 Blender 集成的新渲染引擎。

* 订阅对数据及其属性的更改。

* 在现有 Blender 数据中定义新的设置。

* 使用 Python 在三维视口中绘制。

仍然缺失的特性：

* 创建新的空间类型。

* 为每种类型分配自定义属性。

## Before Starting

### Running Scripts

执行 Python 脚本的两种最常见方式是使用内置文本编辑器或在 Python 控制台中输入命令。文本编辑器和 Python 控制台都是可选择的空间类型。你可以使用顶部栏选项卡可访问的脚本工作区，而不用手动配置用于 Python 开发的空间。

## Key Concepts

### Data Access

#### Accessing Data-Blocks

从当前加载的 .blender 文件访问数据通过模块 bpy.data 完成。

```python
bpy.data.objects
# <bpy_collection[3], BlendDataObjects>
```

```python
bpy.data.scenes
# <bpy_collection[1], BlendDataScenes>
```

```python
bpy.data.materials
# <bpy_collection[1], BlendDataMaterials>
```

#### Accessing Collections

你会注意到，索引和字符串都可以用于访问集合的成员。与 Python 字典不同，这两种方法都可用；然而，在运行 Blender 时，成员的索引可能改变。

```python
list(bpy.data.objects)
# [bpy.data.objects["Cube"], bpy.data.objects["Plane"]]
```

```python
bpy.data.objects['Cube']
# bpy.data.objects["Cube"]
```

```python
bpy.data.objects[0]
# bpy.data.objects["Cube"]
```

#### Accessing Attributes

```python
bpy.data.objects[0].name
# 'Camera'
```

```python
bpy.data.scenes['Scene']
# bpy.data.scenes['Scene']
```

```python
bpy.data.materials.new("MyMaterial")
# bpy.data.materials['MyMaterial']
```

```python
bpy.data.scenes[0].render.resolution_percentage
# 100
bpy.data.scenes[0].objects["Torus"].data.vertices[0].co.x
# 1.0
```

#### Data Creation/Removal

Blender Python API 不能创建存在于主 Blender 数据库（通过 bpy.data 访问）之外的 Blender 数据，因为这些数据由 Blender 管理。

```python
mesh = bpy.data.meshes.new(name="MyMesh")
print(mesh)
# <bpy_struct, Mesh("MyMesh.001")>
```

```python
bpy.data.meshes.remove(mesh)
```

#### Custom Properties

Python 可以访问任何具有 ID 的数据块的属性。赋予属性时，你可以选择自己的名字，这些名字被创建或覆盖。

此数据与 .blend 文件一起保存，并与对象一起复制。

```python
bpy.context.object["MyOwnProperty"] = 42

if "SomeProp" in bpy.context.object:
    print("Property found")

# Use the get function like a Python dictionary
# which can have a fallback value.
value = bpy.data.scenes["Scene"].get("test_prop", "fallback value")

# dictionaries can be assigned as long as they only use basic types.
collection = bpy.data.collections.new("MyTestCollection")
collection["MySettings"] = {"foo": 10, "bar": "spam", "baz": {}}

del collection["MySettings"]
```

注意这些属性仅能被赋予基础 Python 类型。

### Context

虽然通过名字或列表直接访问数据非常有用，但更常见的是操作用户的选择。上下文总可以从 bpy.context 中获得，可以用于获取激活的对象、场景、工具设置和许多其他属性。

```python
bpy.context.object
bpy.context.selected_objects
bpy.context.visible_bones
```

注意上下文是只读的，这意味着这些值不能直接修改。但通过运行 API 函数或使用数据 API 可以改变它们。因此 `bpy.context.active_object = obj` 会报错，但 `bpy.context.view_layer.objects.active = obj` 像预期那样工作。

上下文属性根据访问位置变化。3D 视口具有与 Python 控制台不同的上下文属性，所以在访问用户状态已知的上下文属性时要小心。

### Operators (Tools)

操作符是一种工具，用户通常通过按钮、菜单项或快捷键来访问。从用户角度看它们是工具，但 Python 可以通过 bpy.ops 模块使用自己的设置运行它们。

```python
bpy.ops.mesh.flip_normals()
# {'FINISHED'}
bpy.ops.mesh.hide(unselected = False)
# {'FINISHED'}
bpy.ops.object.transform_apply()
# {'FINISHED'}
```

#### Operator Poll()

许多操作符都有一个 poll 函数，用于检查光标是否在有效区域或对象是否处于正确模式。当 Python 中操作符的 poll 函数运行失败时，会触发异常。

为避免在调用操作符的任何地方使用 try-except 语句，你可以调用操作符自己的 poll 函数，来检查是否可以在当前上下文中运行操作符。

```python
if bpy.ops.view3d.render_border.poll():
    bpy.ops.view3d.render_border()
```

## Integration

Python 脚本可以使用以下方式与 Blender 集成：

* 定义渲染引擎

* 定义操作符

* 定义菜单、标题和面板

* 向已存在的菜单、标题和面板插入新按钮

在 Python 中，这是通过定义一个类来完成的，该类是现有类型的子类。

### Example Operator

### Example Panel

## Types

Blender 定义了一系列 Python 类型，但也使用 Python 原生类型。Blender Python API 可以分为三种类型。

### Native Types

在简单情况下，将数字或字符串作为自定义类型会很麻烦，因此可以将它们作为普通 Python 类型访问。

* Blender float, int, boolean -> float, int, boolean

* Blender 枚举 -> string

```python
C.object.rotation_mode = 'AXIS_ANGLE'
```

* Blender 多个枚举 -> set of strings

```python
# setting multiple camera overlay guides
bpy.context.scene.camera.data.show_guide = {'GOLDEN', 'CENTER'}

# passing as an operator argument for report types
self.report({'WARNING', 'INFO'}, "Some message!")
```

### Internal Types

bpy.types.bpy_struct 用于 Blender 数据块和集合，也适用于包含自身属性的数据：集合、网格、骨骼和场景等。

有两种包装 Blender 数据的主要类型，一种用于数据块（内部称为 bpy_struct），另一种用于属性。

```python
bpy.context.object
# bpy.data.objects['Cube']
```

```python
C.scene.objects
# bpy.data.scenes['Scene'].objects
```

注意这些类型引用了 Blender 的数据，因此修改它们立即可见。

### Mathutils Types

可以从 mathutils 访问向量、四元数、欧拉角、矩阵和颜色类型。一些属性，例如 bpy.types.Object.location、bpy.types.PoseBone.rotation_euler 和 bpy.types.Scene.cursor_location，可以作为特殊数学类型访问。 

```python
# 矩阵、向量乘法
bpy.context.object.matrix_world @ bpy.context.object.data.verts[0].co
```

mathutils 类型保留对 Blender 内部数据的引用，以便可以应用更改。

```python
# modifies the Z axis in place.
bpy.context.object.location.z += 2.0

# location variable holds a reference to the object too.
location = bpy.context.object.location
location *= 2.0

# Copying the value drops the reference so the value can be passed to
# functions and modified without unwanted side effects.
location = bpy.context.object.location.copy()
```

## Animation

有两种方式通过 Python 添加关键帧。

第一种是直接通过关键属性，这就像用户从按钮插入关键帧。你也可以手动创建曲线和关键帧数据，然后设置属性的路径。下面是两种方法的例子，都在活动对象的 Z 轴上插入关键帧。

```python
obj = bpy.context.object
obj.location[2] = 0.0
obj.keyframe_insert(data_path = "location", frame = 10.0, index = 2)
obj.location[2] = 1.0
obj.keyframe_insert(data_path = "location", frame = 20.0, index = 2)
```

```python
obj = bpy.context.object
obj.animation_data_create()
obj.animation_data.action = bpy.data.actions.new(name = "MyAction")
fcu_z = obj.animation_data.action.fcurves.new(data_path = "location", index = 2)
fcu_z.keyframe_points.add(2)
fcu_z.keyframe_points[0].co = 10.0, 0.0
fcu_z.keyframe_points[1].co = 20.0, 1.0
```
