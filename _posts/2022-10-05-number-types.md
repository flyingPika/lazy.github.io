---
title: CGAL 5.5 数值类型
date: 2022-10-05 00:00:00 +0800
categories: [CG, GEOMETRY]
tags: [cgal]     # TAG names should always be lowercase
math: false
pin: false
---

<https://doc.cgal.org/latest/Number_types/index.html#Chapter_Number_Types>

## 1 介绍

本章节概述了 CGAL 支持的数值类型。数值类型必须满足某些语法和语义要求，这样才能被用于 CGAL 代码。总体上，它们是代数结构的模型。如果它们是实数子环的模型，它们也是 RealEmbeddable 的模型。

## 2 内置数值类型

内置数值类型 float、double 和 long double 具有所需的算术和比较运算符。它们缺少一些必需的例行程序，尽管这些例行程序被 CGAL 自动包含。

所有 C++ 内置数值类型只能表示有理数的离散（有界）子集。假设浮点运算遵循 IEEE 浮点标准。浮点文化具有比精确计算更多的基础结构支持（硬件、语言定义和编译器），因而非常高效。与所有具有有限精度表示的数值类型一样，内置数值类型本质上不精确。如果您决定使用高效的内置数值类型，请注意：您必须处理数值问题。例如，可以计算两条直线的交点，然后检查该点是否位于两条直线上。使用浮点运算，舍入误差可能导致结果为 false；使用内置整数类型，可能出现溢出。

## 3 CGAL 提供的数值类型

CGAL 提供可以用于精确计算的多种数值类型。

* Quotient

* MP_Float

* Lazy_exact_nt

* Interval_nt

* Sqrt_extension

* Number_type_checker

## 4 GMP 提供的数值类型

CGAL 为 GMP 算术库中定义的数值类型提供包装类。

* **Gmpz** CGAL/Gmpz.h：任意精度整数类型 mpz_t 的包装类。

* **Gmpq** CGAL/Gmpq.h：任意精度有理数类型 mpq_t 的包装类。

* **Gmpzf** CGAL/Gmpzf.h：精确任意精度浮点数类型，不支持 / 以保证操作的精确性，算术操作限制为 +、-、* 和 integral_division()。

* **Mpzf** CGAL/Mpzf.h：Gmpzf 的更快替代方案、不支持 integral_division()。

* **Gmpfr** CGAL/Gmpfr.h：固定精度浮点数类型。

此外，可以直接使用 GMP 提供的 C++ 数值类型：mpz_class 和 mpq_class（注意对 mpf_class 的支持不完全）。文件 CGAL/gmpxx.h 提供了必要函数，以使这些类兼容 CGAL 数值类型。

要使用这些类，必须安装 GMP 和 MPFR。

## 5 LEDA 提供的数值类型

LEDA 提供可用于笛卡尔和齐次表示的精确计算的数值类型。

## 6 CORE 提供的数值类型

总体上，Core 提供了与 LEDA 相同的数值类型集合。

## 7 区间算术

区间算术对于几何程序非常重要，它是过滤谓词的基本工具。对于许多问题，机器双精度数值区间是足够的，但并不总是足够的。

对于机器双精度数值区间，CGAL 提供 Interval_nt 类。对于浮点任意精度数值区间，CGAL 提供 Gmpfi 类。

Gmpfi 区间端点表示为 Gmpfr 数值。每个区间都有一个关联精度，即其端点的最大精度（用于表示尾数的位数）。操作结果始终被包含在返回的区间中。由于区间算术是在 Gmpfr 之上实现的，全局标志从 Gmpfr 接口继承。

要使用 Gmpfi 类，必须安装 MPFI。

## 8 用户提供的数值类型

0^0

## 9 设计和实现历史

略
