---
title: MLS 方法概述
pubDate: '2026-04-08'
---

MLS (Moving Least Squares，移动最小二乘法) 是一种局部拟合方案，常用于诸如表面重建、粒子模拟之类的任务。简单地说，对于一系列离散数据点，传统的最小二乘法试图全局拟合，MLS 注重于在某个点周围拟合出一个局部结构。

有一组离散点 $\mathbf x_i$，其对应属性值 $u_i$。对于其中一点 $\mathbf z$，MLS 的核心方法如下：

1. 定义基函数组 $\mathrm P(\mathbf x)$。如三维线性基 $\mathrm P = \begin{pmatrix}1&x&y&z\end{pmatrix}^T$。
2. 为 $\mathbf z_i$ 支撑域内的点定义权重 $w_i$。
3. 求解局部系数 $\mathrm c(\mathbf z)$ 以最小化 L2 范数：
$$
\begin{array}{rll}
J&= \sum _i w_i \|\mathrm P^T(\mathbf x_i - \mathbf z)\mathrm c(\mathbf z) - u_i \|^2
\end{array}
$$  

接下来导出 $\mathrm c$ 的解析解。

首先 $J$ 可以进一步写成
$$
J = (\mathbf P\mathbf c - \mathbf u)^T \mathbf W (\mathbf P\mathbf c - \mathbf u)
$$

其中 $\mathbf P = \begin{pmatrix} \mathrm P(\mathbf x_1 - \mathbf z)^T \\ \mathrm P(\mathbf x_2 - \mathbf z)^T \\ \vdots \\ \mathrm P(\mathbf x_n - \mathbf z)^T \end{pmatrix}$，$\mathbf u = \begin{pmatrix} u_1 \\ u_2 \\ \vdots \\ u_n \end{pmatrix}$，$W = \mathrm{diag}(w_1, w_2, \dots, w_n)$。

$$
\begin{array}{rll}
& \text {minimize}\; J\\\\

\rightarrow & \frac {\partial J}{\partial \mathrm c} = 0 \\\\
\rightarrow & \frac {\partial}{\partial \mathrm c} \left( (\mathbf P\mathbf c - \mathbf u)^T \mathbf W (\mathbf P\mathbf c - \mathbf u) \right) = 0 \\
\\
\rightarrow & \frac {\partial}{\partial \mathrm c} \left( \mathbf c^T \mathbf P^T \mathbf W \mathbf P \mathbf c - 2 \mathbf c^T \mathbf P^T \mathbf W \mathbf u + \mathbf u^T \mathbf W \mathbf u \right) = 0 \\
\\
\rightarrow & 2 \mathbf P^T \mathbf W \mathbf P \mathbf c - 2 \mathbf P^T \mathbf W \mathbf u = 0 \\
\\
\rightarrow & (\mathbf P^T \mathbf W \mathbf P) \mathbf c = \mathbf P^T \mathbf W \mathbf u
\end{array}
$$

定义 **矩矩阵 (Moment Matrix)** 为 $\mathbf M(\mathbf z) = \mathbf P^T \mathbf W \mathbf P$，则系数的解析解为：
$$
\mathrm c(\mathbf z) = \mathbf M^{-1}(\mathbf z) \mathbf P^T \mathbf W \mathbf u
$$