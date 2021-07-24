---
title: Taichi 0.7.22 高级编程
date: 2021-07-24 03:00:00 +0800
categories: [CG, SIMULATION]
tags: [taichi]     # TAG names should always be lowercase
math: true
pin: false
---

## Metaprogramming

太极提供元编程基础设施，元编程可以统一维度依赖的代码开发、提高运行时的性能、简化太极标准库的开发。太极内核被延迟实例化，大量计算发生在编译时。每一个太极内核都是模板内核，即使没有模板参数。

### 模板元编程

```python
@ti.kernel
def copy(x: ti.template(), y: ti.template()):
    for i in x:
        y[i] = x[i]

a = ti.field(ti.f32, 4)
b = ti.field(ti.f32, 4)
c = ti.field(ti.f32, 12)
d = ti.field(ti.f32, 12)
copy(a, b)
copy(c, d)
```

### 使用分组索引的维度无关编程

```python
@ti.kernel
def copy(x: ti.template(), y: ti.template()):
    for I in ti.grouped(y):
        # I is a vector with same dimensionality with x and data type i32
        # If y is 0D, then I = ti.Vector([]), which is equivalent to `None` when used in x[I]
        # If y is 1D, then I = ti.Vector([i])
        # If y is 2D, then I = ti.Vector([i, j])
        # If y is 3D, then I = ti.Vector([i, j, k])
        # ...
        x[I] = y[I]

@ti.kernel
def array_op(x: ti.template(), y: ti.template()):
    # if field x is 2D:
    for I in ti.grouped(x);
        y[I + ti.Vector([0, 1])] = I[0] + I[1]
    # then it is equivalent to:
    for i, j in x:
        y[i, j + 1] = i + j
```

### 场元数据

场的数据类型和大小可以在两类作用域中访问。

```python
@ti.func
def print_field_info(x: ti.template()):
    print('Field dimensionality is', len(x.shape))
    for i in ti.static(range(len(x.shape))):
        print('Size alone dimension', i, 'is', x.shape[i])
    ti.static_print('Field data type is', x.dtype)
```

### 矩阵（向量）元数据

```python
@ti.kernel
def foo():
  matrix = ti.Matrix([[1, 2], [3, 4], [5, 6]])
  print(matrix.n)  # 3
  print(matrix.m)  # 2
  vector = ti.Vector([7, 8, 9])
  print(vector.n)  # 3
  print(vector.m)  # 1
```

### 编译时计算

```python
enable_projection = True
@ti.kernel
def static():
  if ti.static(enable_projection):  # No runtime overhead
    x[0] = 1

@ti.kernel
def func():
  for i in ti.static(range(4)):  # 强制循环展开
      print(i)

@ti.kernel
def reset():
  for i in x:
    for j in ti.static(range(x.n)):
      # The inner loop must be unrolled since j is a vector index instead of a global field index.
      x[i][j] = 0
```

循环展开可以提高运行时性能。太极中矩阵索引必须是编译时常量，而场索引可以是运行时变量。

## Advanced dense layouts

太极从数据布局中解耦算法，自动优化在特定数据布局上的数据访问。这允许我们尝试多种不同的数据布局，以找出对于特定任务最高效的一种。

建议先使用默认布局规范，如果必要的话使用 ti.root.X 迁移到高级布局。

### 从 shape 到 ti.root.X

```python
x = ti.field(ti.f32)
ti.root.place(x)
# is equivalent to:
x = ti.field(ti.f32, shape=())
```

```python
x = ti.field(ti.f32)
ti.root.dense(ti.i, 3).place(x)
# is equivalent to:
x = ti.field(ti.f32, shape=3)
```

```python
x = ti.field(ti.f32)
ti.root.dense(ti.ij, (3, 4)).place(x)
# is equivalent to:
x = ti.field(ti.f32, shape=(3, 4))
```

### 行优先 vs. 列优先

```python
ti.root.dense(ti.i, 3).dense(ti.j, 2).place(x)    # row-major (default)
ti.root.dense(ti.j, 2).dense(ti.i, 3).place(y)    # column-major
```

```cpp
int x[3][2];  // row-major
int y[2][3];  // column-major
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 2; j++) {
        do_something(x[i][j]);
        do_something(y[j][i]);
    }
}
```

### 数组结构体（AoS）, 结构体数组（SoA）

大小相同的场可以在一起被放置。如果数据元素经常在一起访问，将其放置在相对近的存储位置有助于提高缓存命中率。

```python
# AoS
ti.root.dense(ti.i, 3).place(x, y)  
#  address low ............. address high
#  x[0]   y[0] | x[1]  y[1] | x[2]   y[2]
```

```python
# SoA
ti.root.dense(ti.i, 3).place(x)
ti.root.dense(ti.i, 3).place(y)
#  address low ............. address high
#  x[0]  x[1]   x[2] | y[0]   y[1]   y[2]
```

### 平面布局 vs. 层次布局

```python
val = ti.field(ti.f32, shape=(32, 64, 128))

# a better layout
val_ = ti.field(ti.f32)
ti.root.dense(ti.ijk, (8, 16, 32)).dense(ti.ijk, (4, 4, 4)).place(val_)
```

### 对于高级稠密数据布局的结构 for 循环

结构 for 循环会自动遵循嵌套稠密数据结构在内存中的顺序。

```python
# A为行优先的，则遍历顺序为行优先的
# A为列优先的，则遍历顺序为列优先的
# A为层次的，则一层一层遍历
for i, j in A:
  A[i, j] += 1
```

### 例子

```python
# 行优先
A = ti.field(ti.f32)
ti.root.dense(ti.ij, (256, 256)).place(A)

# 列优先
A = ti.field(ti.f32)
ti.root.dense(ti.ji, (256, 256)).place(A) # Note ti.ji instead of ti.ij

# 8x8 blocked 2D array of size 1024x1024
density = ti.field(ti.f32)
ti.root.dense(ti.ij, (128, 128)).dense(ti.ij, (8, 8)).place(density)

# AoS
pos = ti.Vector.field(3, dtype=ti.f32)
vel = ti.Vector.field(3, dtype=ti.f32)
ti.root.dense(ti.i, 1024).place(pos, vel)
# equivalent to
ti.root.dense(ti.i, 1024).place(pos(0), pos(1), pos(2), vel(0), vel(1), vel(2))

# SoA
pos = ti.Vector.field(3, dtype=ti.f32)
vel = ti.Vector.field(3, dtype=ti.f32)
for i in range(3):
  ti.root.dense(ti.i, 1024).place(pos(i))
for i in range(3):
  ti.root.dense(ti.i, 1024).place(vel(i))
```

## Sparse computation

参考论文：[Taichi: A Language for High-Performance Computation on Spatially Sparse Data Structures](https://yuanming.taichi.graphics/publication/2019-taichi/taichi-lang.pdf)

## Differentiable programming

### ti.Tape()

ti.Tape(U) 的参数 U 必须是0维场，在开始时自动设置 U[None] 为0。

1. needs_grad=True

2. with ti.Tape(y):

3. x.grad[None]

```python
x = ti.field(float, (), needs_grad=True)
y = ti.field(float, (), needs_grad=True)

@ti.kernel
def compute_y():
    y[None] = ti.sin(x[None])

with ti.Tape(y):
    compute_y()

print('dy/dx =', x.grad[None])
print('at x =', x[None])
```

```python
x = ti.field(float, ())
y = ti.field(float, ())
dy_dx = ti.field(float, ())

@ti.kernel
def compute_dy_dx():
    dy_dx[None] = ti.cos(x[None])

compute_dy_dx()

print('dy/dx =', dy_dx[None])
print('at x =', x[None])
```

### 内核简化规则

全局数据访问规则：若全局场元素被写入多次，从第二次开始必须使用原子累加。直到累加完成，才能进行读操作。

内核简化规则：内核本体由多个简单嵌套 for 循环组成。

```python
@ti.kernel
def differentiable_task():
  for i in x:
    x[i] = y[i]

  for i in range(10):
    for j in range(20):
      for k in range(300):
        ... do whatever you want, as long as there are no loops

  # Not allowed. The outer for loop contains two for loops
  for i in range(10):
    for j in range(20):
      ...
    for j in range(20):
      ...
```

静态 for 循环被 python 前端预处理器展开，因此不计入循环级别。

### DiffTaichi

参考论文：[Differentiable Programming for Physical Simulation](https://arxiv.org/pdf/1910.00935.pdf)

## Performance tuning

### for 循环装饰器

### GPU 线程层次

迭代 < 线程 < 线程块 < 网格

线程是并行的最小单元。线程里的所有迭代串行执行。我们通常在每个线程里使用一个迭代以最大化并行性能。

线程块中的所有线程并行执行，共享块局部存储。

网格是被主机启动的最小单元。网格中的所有块并行执行。太极中并行 for 循环是一个网格。

### API 参考

`ti.block_dim(n)` 指定下一个并行 for 循环中每个线程块的线程数（必须为2的幂）

```python
@ti.kernel
def func():
    for i in range(8192):  # no decorator, use default settings
        ...

    ti.block_dim(128)      # change the property of next for-loop:
    for i in range(8192):  # will be parallelized with block_dim=128
        ...

    for i in range(8192):  # no decorator, use default settings
```

## Objective data-oriented programming

略

## Syntax sugars

在太极中，可以对内核和函数的局部变量使用 ti.static() 来创建别名。

场和函数可以通过 ti.static() 指定别名。

```python
@ti.kernel
def my_kernel():
  a, b, fun = ti.static(field_a, field_b, some_function)
  for i,j in a:
    b[i,j] = fun(a[i,j])
```

别名也可以为类成员和方法创建，避免杂乱的 self。

```python
@ti.kernel
def compute_laplacian(self):
  a,b,dx,dy = ti.static(self.a,self.b,self.dx,self.dy)
  for i,j in a:
    b[i,j] = (a[i+1, j] - 2.0*a[i, j] + a[i-1, j])/(dx**2) \
           + (a[i, j+1] - 2.0*a[i, j] + a[i, j-1])/(dy**2)
```

## Coordinate offsets

场可以使用坐标偏移量定义。偏移量移动场的边界，场的原点不再是零向量。场的维度应和偏移量一致。一个典型的用例是在物理模拟中支持具有负坐标的体素。

```python
a = ti.Matrix.field(2, 2, dtype=ti.f32, shape=(32, 64), offset=(-16, 8)
a[-16, 8]  # lower left corner
a[16, 8]   # lower right corner
a[-16, 72]  # upper left corner
a[16, 72]   # upper right corner
```

```python
a = ti.Matrix.field(2, 3, dtype=ti.f32, shape=(32,), offset=(-16, ))          # Works!
b = ti.Vector.field(3, dtype=ti.f32, shape=(16, 32, 64), offset=(7, 3, -4))   # Works!
c = ti.Matrix.field(2, 1, dtype=ti.f32, shape=None, offset=(32,))             # AssertionError
d = ti.Matrix.field(3, 2, dtype=ti.f32, shape=(32, 32), offset=(-16, ))       # AssertionError
e = ti.field(dtype=ti.i32, shape=16, offset=-16)                          # Works!
f = ti.field(dtype=ti.i32, shape=None, offset=-16)                        # AssertionError
g = ti.field(dtype=ti.i32, shape=(16, 32), offset=-16)                    # AssertionError
```
