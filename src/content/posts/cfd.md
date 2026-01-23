---
title: NS 方程的计算方法
pubDate: '2026-01-23'
---

怎么在计算机里模拟流体呢？Navier-Stokes 方程告诉我们：

$$
\frac{\partial \mathbf{u}}{\partial t}
=-\underbrace{(\mathbf{u}\cdot\nabla)\mathbf{u}}_{\text{平流项}} +\underbrace{\nu\nabla^2\mathbf{u}}_\text{扩散项}+\underbrace{\mathbf{f}}_\text{外力项}-\underbrace{\frac{1}{\rho}\nabla p}_{\text{压力项}}
$$

并且 $\mathbf{u}$ 无源：

$$
\nabla \cdot \mathbf{u}=0
$$

接下来逐项求解。

### 平流项

平流项：$\frac{\partial \mathbf{u}}{\partial t}=(\mathbf{u}\cdot\nabla)\mathbf{u}$

平流项源于对速度场 $\mathbf{u}(x, y, z, t)$ 的全导数：

$$
\begin{array}{rll}
\frac{\mathrm{d}\mathbf{u}}{\mathrm{d}t}
=&
\frac{\partial\mathbf{u}}{\partial x}
\frac{\mathrm{d}x}{\mathrm{d} t} +
\frac{\partial\mathbf{u}}{\partial y}
\frac{\mathrm{d}y}{\mathrm{d} t} +
\frac{\partial\mathbf{u}}{\partial z}
\frac{\mathrm{d}z}{\mathrm{d} t} +
\frac{\partial\mathbf{u}}{\partial t}\\
=&
\frac{\partial\mathbf{u}}{\partial x}
\mathbf{u}_x+
\frac{\partial\mathbf{u}}{\partial y}
\mathbf{u}_y+
\frac{\partial\mathbf{u}}{\partial z}
\mathbf{u}_z+
\frac{\partial\mathbf{u}}{\partial t}\\
=&
(\mathbf{u}\cdot\nabla) \mathbf{u}+\frac{\partial\mathbf{u}}{\partial t}\\
\end{array}
$$
