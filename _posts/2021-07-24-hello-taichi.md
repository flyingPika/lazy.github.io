---
title: Taichi 0.7.22 HELLO TAICHI
date: 2021-07-24 00:00:00 +0800
categories: [CG, SIMULATION]
tags: [taichi]     # TAG names should always be lowercase
math: true
pin: false
---

## OVERVIEW

### Hello,world!

```python
import taichi as ti
# 初始化
ti.init(arch=ti.gpu)  # 在GPU上运行，自动选择后端（CUDA->OpenGL->CPU）
ti.init(arch=cuda)
ti.init(arch=opengl)
ti.init(arch=cpu)
```

当在 Windows 或 ARM 设备上使用 CUDA 后端时，太极默认为字段存储分配1 GB显存。

```python
# 指定太极作用域
@ti.kernel  # 被python代码调用，指定参数类型
@ti.func  # 被太极内核或函数调用
```

支持嵌套**函数**，不支持递归**函数**，不支持嵌套**内核**。

内核最外层作用域的 for 循环被自动并行，不能使用 break 结束并行循环。

* 范围 for 循环
* 结构 for 循环：必须在内核的最外层作用域

```python
# field与numpy data进行转换
pixels = ti.field(ti.f32, (1024, 512))
arr = np.random.rand(1024, 512)
pixels.from_numpy(arr)  # 使用numpy data初始化field
arr = pixels.to_numpy()  # 将field转换为numpy data
```

## BASIC CONCEPTS

### Kernels and functions

内核被 Python 作用域调用。内核至多有八个参数，指定参数类型，仅支持标量。返回值为空，或者为指定类型的标量值。返回值会自动转换为指定类型。

```python
@ti.kernel
def my_kernel(x: ti.f32) -> ti.i32:
    return x

print(my_kernel(1.1))  # 1
```

函数被太极作用域调用。函数可以有多个参数或返回值，参数不必指定类型、通过值传递。参数和返回值支持向量和矩阵。

函数不支持多个 return 关键字。

```python
@ti.func
def my_func(x):
    x = x + 1  # won't change the original value of x

@ti.kernel
def my_kernel():
    ...
    x = 233
    my_func(x)
    print(x)  # 233
    ...
```

### Type system

i32 和 f32 最常用，32为默认精度。可以使用 int 和 float 作为默认精度的别名。布尔类型用 i32 表示。

`i32 + f32 = f32`

`i32 + i64 = i64`

```python
# 修改默认精度
ti.init(default_ip=ti.i64, default_fp=ti.f32)
```

浮点数转换成整数时小数截断。

* 隐式类型转换

```python
a = 1.0
a = 2
print(a)  # 2.0
```

```python
# warning
a = 1
a = 2.0
print(a)  # 2
```

* 显式类型转换

```python
a = 1.0
b = ti.cast(a, ti.i32)  # 1
c = ti.cast(b, ti.f32)  # 1.0
```

```python
a = 1.0
b = int(a)  # 1
c = float(b)  # 1.0
```

```python
u = ti.Vector([2.3, 4.7])
v = int(u)  # ti.Vector([2, 4])
# If you are using ti.i32 as default_ip, this is equivalent to:
v = ti.cast(u, ti.i32)  # ti.Vector([2, 4])
```

### Fields and matrices

场是由太极提供的全局变量。场可以是稀疏的或者稠密的。场的元素可以是标量或矩阵（向量）。

* 标量场

场通过索引访问，访问0维场使用 None 作为索引值。场的值被初始化为0。

* 矩阵场

ti.Vector 仅仅是 ti.Matrix 的别名。尽量使用较小尺寸的矩阵。

```python
# A为128*64的场，其中的元素为3*2的矩阵
A = ti.Matrix.field(3, 2, dtype=ti.f32, shape=(128, 64))
mat = A[0, 0]
print(mat[2, 1])
print(A[0, 0][2, 1])  # 前者是场的索引，后者是矩阵的索引
```

### Interacting with external arrays

* 标量场

```python
field = ti.field(ti.i32, shape=(233, 666))
field.shape  # (233, 666)

array = field.to_numpy()
array.shape  # (233, 666)

field.from_numpy(array)  # the input array must be of shape (233, 666)
```

* 向量场

```python
field = ti.Vector.field(3, ti.i32, shape=(233, 666))
field.shape  # (233, 666)
field.n      # 3

array = field.to_numpy()
array.shape  # (233, 666, 3)

field.from_numpy(array)  # the input array must be of shape (233, 666, 3)
```

* 矩阵场

```python
field = ti.Matrix.field(3, 4, ti.i32, shape=(233, 666))
field.shape  # (233, 666)
field.n      # 3
field.m      # 4

array = field.to_numpy()
array.shape  # (233, 666, 3, 4)

field.from_numpy(array)  # the input array must be of shape (233, 666, 3, 4)
```

```python
# 传递外部数组作为内核参数
@ti.kernel
def test_numpy(arr: ti.ext_arr()):
    ...
    
a = np.empty(shape=(n, m), dtype=np.int32)
test_numpy(a)
```
