---
title: Taichi 0.7.22 API 参考
date: 2021-07-24 02:00:00 +0800
categories: [CG, SIMULATION]
tags: [taichi]     # TAG names should always be lowercase
math: false
pin: false
---

## Scalar fields

标量场最多有8个维度。

* 0维标量场即一个标量。

* 1维标量场为一维线性数组。

* 2维标量场用于二维的值。

* 3维标量场用于体数据。

标量场可以是稀疏或稠密的，这里仅考虑稠密标量场。

### 声明

`ti.field(dtype, shape = None, offset = None)`

```python
x = ti.field(ti.i32, shape=4)

x = ti.field(ti.f32, shape=(4, 3))

x = ti.field(ti.f32, shape=())  # 使用None作为索引

# equivalent to: x = ti.field(ti.f32, shape=(4, 3))
x = ti.field(ti.f32)
ti.root.dense(ti.ij, (4, 3)).place(x)
```

在调用内核或者访问任何场之前，所有场都必须被声明定义。

### 访问

`a[p, q, ...]`

### 元数据

`a.shape`

`a.dtype`

`a.parent(n=1)`

## Vectors

### 声明

* 全局向量场

`ti.Vector.field(n, dtype, shape = None, offset = None)`

* 临时局部变量

`ti.Vector([x, y, ...])`

### 访问

* 全局向量场

`a[p, q, ...][i]`

* 临时局部变量

`a[i] a.x a.y a.z a.w`

### 方法

向量场对象调用以下方法，不会改变对象本身。

`a.norm(eps = 0)`

```python
a = ti.Vector([3, 4])
a.norm()  # ti.sqrt(a.dot(a) + eps) = 5
```

`a.norm_sqr()`

```python
a = ti.Vector([3, 4])
a.norm_sqr()  # 3*3 + 4*4 = 25
```

`a.normalized()`

```python
a = ti.Vector([3, 4])
a.normalized()  # [3 / 5, 4 / 5]
```

`a.dot(b)`

```python
a = ti.Vector([1, 3])
b = ti.Vector([2, 4])
a.dot(b) # 1*2 + 3*4 = 14
```

`a.cross(b)`

```python
a = ti.Vector([1, 2, 3])
b = ti.Vector([4, 5, 6])
c = ti.cross(a, b)  # c = [2*6 - 5*3, 4*3 - 1*6, 1*5 - 4*2] = [-3, 6, -3]

p = ti.Vector([1, 2])
q = ti.Vector([4, 5])
r = ti.cross(a, b)  # r = 1*5 - 4*2 = -3
```

`a.outer_product(b)`

```python
# 张量积
a = ti.Vector([1, 2])
b = ti.Vector([4, 5, 6])
c = ti.outer_product(a, b) # NOTE: c[i, j] = a[i] * b[j]
# c = [[1*4, 1*5, 1*6], [2*4, 2*5, 2*6]]
```

`a.cast(dt)`

```python
# 太极作用域
a = ti.Vector([1.6, 2.3])
a.cast(ti.i32) # [2, 3]
```

### 元数据

`a.n`

`a.shape`

`a.dtype`

## Matrices

* 逐元素乘法 `*`，矩阵乘法 `@`。

* `A.transpose()`

* `R, S = ti.polar_decompose(A, ti.f32)`

* `U, sigma, V = ti.svd(A, ti.f32)` sigma 为3*3对角矩阵

* `eigenvalues, eigenvectors = ti.eig(A, ti.f32)` 以负数形式存储

* `eigenvalues, eigenvectors = ti.sym_eig(A, ti.f32)` 对于对称矩阵，以实数形式存储

### 声明

* 全局矩阵场

`ti.Matrix.field(n, m, dtype, shape = None, offset = None)`

```python
# Python作用域
a = ti.Matrix.field(3, 4, dtype=ti.f32, shape=(5, 4))  # 3*4矩阵
```

* 临时局部变量

`ti.Matrix([[x, y, ...][, z, w, ...], ...])`

```python
# 太极作用域
a = ti.Matrix([[2, 3, 4], [5, 6, 7]])  # 2*3矩阵
```

`ti.Matrix.rows([v0, v1, v2, ...])`

`ti.Matrix.cols([v0, v1, v2, ...])`

```python
# 太极作用域
v0 = ti.Vector([1.0, 2.0, 3.0])
v1 = ti.Vector([4.0, 5.0, 6.0])
v2 = ti.Vector([7.0, 8.0, 9.0])

# to specify data in rows
a = ti.Matrix.rows([v0, v1, v2])

# to specify data in columns instead
a = ti.Matrix.cols([v0, v1, v2])

# lists can be used instead of vectors
a = ti.Matrix.rows([[1.0, 2.0, 3.0], [4.0, 5.0, 6.0], [7.0, 8.0, 9.0]])
```

### 访问

* 全局矩阵场

`a[p, q, ...][i, j]`

* 临时局部变量

`a[i, j]`

### 方法

矩阵场对象调用以下方法，不会改变对象本身。

`a.transpose()`

`a.trace()`

`a.determinant()` 太极作用域，最大为4*4矩阵

`a.inverse()` 太极作用域，最大为4*4矩阵

## Scalar operations

### 操作符

`-a`

`a + b`

`a - b`

`a * b`

`a / b`

`a // b`

`a % b`

```python
# python作用域和太极作用域
print(2 % 3)   # 2
print(-2 % 3)  # 1
# c风格求余
print(ti.raw_mod(2, 3))   # 2
print(ti.raw_mod(-2, 3))  # -2
```

`a ** b`

`~a`

`a == b`

`a != b`

`a > b`

`a < b`

`a >= b`

`a <= b`

`not a`

`a or b`

`a and b`

`a if cond else b`

`a & b`

`a ^ b`

`a | b`

### 函数

`ti.sin(x)`

`ti.cos(x)`

`ti.tan(x)`

`ti.asin(x)`

`ti.acos(x)`

`ti.atan2(y, x)`

`ti.tanh(x)`

`ti.sqrt(x)`

`ti.rsqrt(x)` A fast version for 1 / ti.sqrt(x).

`ti.exp(x)`

`ti.log(x)`

`ti.floor(x)`

`ti.ceil(x)`

`ti.cast(x, dtype)`

`int(x)` ti.cast(x, int)

`float(x)` ti.cast(x, float)

`abs(x)`

`max(x, y, ...)`

`min(x, y, ...)`

`pow(x, y)  # x ** y`

对于 CPU 和 CUDA 后端，在函数 ti.init() 中为参数 random_seed 赋值整型随机数种子。默认的随机数种子为0。

`ti.random(dtype = float)` 均匀随机浮点数或整数

`ti.randn(dtype = None)` 服从标准正态分布的随机浮点数

### 向量和矩阵的逐元素计算

当在向量和矩阵上应用标量操作时，操作会作用在每个元素上。

```python
B = ti.Matrix([[1.0, 2.0, 3.0], [4.0, 5.0, 6.0]])
C = ti.Matrix([[3.0, 4.0, 5.0], [6.0, 7.0, 8.0]])

A = ti.sin(B)
A = B ** 2
A = B ** C
A += 2
A += B
```

## Atomic operations

增量运算符（x[i] += 1）是原子的。并行修改全局变量时，应确保执行原子操作。当原子操作应用于局部变量时，太极编译器会将这些操作分解为非原子变体。

显式原子操作符返回第一个参数的原始值，在第一个参数中存储结果。

`ti.atomic_add(x, y)`

```python
x[i] = 3
y[i] = 4
z[i] = ti.atomic_add(x[i], y[i])  # now x[i] = 7, y[i] = 4, z[i] = 3
```

`ti.atomic_sub(x, y)`

`ti.atomic_and(x, y)`

`ti.atomic_or(x, y)`

`ti.atomic_xor(x, y)`

## Structural nodes

太极提供结构节点（SNodes）以组成层次结构和特定属性。ti.root 是数据结构的根节点。

`snode.place(x, ...)`

```python
x = ti.field(dtype=ti.i32)
y = ti.field(dtype=ti.f32)
ti.root.place(x, y)
```

`field.shape  # field.snode.shape`

```python
x = ti.field(dtype=ti.i32)
ti.root.dense(ti.ijk, (3, 5, 4)).place(x)
x.shape  # returns (3, 5, 4)
```

`field.snode`

```python
# something wrong
x = ti.field(dtype=ti.i32)
y = ti.field(dtype=ti.f32)
blk1 = ti.root.dense(ti.i, 4)
blk1.place(x, y)
assert x.snode == blk1
```

`snode.shape`
```python
# something wrong
blk1 = ti.root
blk2 = blk1.dense(ti.i,  3)
blk3 = blk2.dense(ti.jk, (5, 2))
blk4 = blk3.dense(ti.k,  2)
blk1.shape  # ()
blk2.shape  # (3, )
blk3.shape  # (3, 5, 2)
blk4.shape  # (3, 5, 4)
```

`snode.parent(n = 1)`

```python
# something wrong
blk1 = ti.root.dense(ti.i, 8)
blk2 = blk1.dense(ti.j, 4)
blk3 = blk2.bitmasked(ti.k, 6)
blk1.parent()  # ti.root
blk2.parent()  # blk1
blk3.parent()  # blk2
blk3.parent(1) # blk2
blk3.parent(2) # blk1
blk3.parent(3) # ti.root
blk3.parent(4) # None
```

### Node types

`snode.dense(indices, shape)` 定长连续数组

```python
x = ti.field(dtype=ti.i32)
ti.root.dense(ti.i, 3).place(x)
y = ti.field(dtype=ti.i32)
ti.root.dense(ti.ij, (3, 4)).place(y)
```

`snode.dynamic(index, size, chunk_size = None)`

`snode.bitmasked()`

`snode.pointer()`

`snode.hash()`

### Working with dynamic SNodes

`ti.length(snode, indices)`

`ti.append(snode, indices, val)`

### Taichi fields like powers of two

非整二次幂的维度会被扩充为整二次幂。

### Indices

`ti.ijkl`

`ti.indices(a, b, ...)`
