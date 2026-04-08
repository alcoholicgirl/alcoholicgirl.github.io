---
title: 线性方程组的数值求解
pubDate: '2026-04-08'
---

本文主要介绍 Jacobi 迭代法、高斯-赛德尔迭代法以及 Multigrid 方法。


为了迭代求解，我们将系数矩阵 $A$ 分解为：
$$A = D + L + U$$
其中：
- $D$：对角阵
- $L$：下三角阵
- $U$：上三角阵

---

### 1. Jacobi 迭代法
$$D\mathbf{x}^{(k+1)} = \mathbf{b} - (L + U)\mathbf{x}^{(k)}$$

TODO

---

### 2. Gauss-Seidel (G-S) 迭代法
$$(D + L)\mathbf{x}^{(k+1)} = \mathbf{b} - U\mathbf{x}^{(k)}$$

TODO

---

### 3. 从平滑到 Multigrid

TODO