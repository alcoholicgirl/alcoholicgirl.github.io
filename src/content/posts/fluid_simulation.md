---
title: Navier-Stokes 方程推导
pubDate: '2026-01-19'
---

原文地址：https://andrew.gibiansky.com/blog/physics/fluid-dynamics-the-navier-stokes-equations/

为了模拟无源且不可压缩的流体需要用到 NS 方程，下面进行推导。

### 物质守恒

在一个闭合区域 $\Omega$ 内，物质总量变化率等于边界上流入/流出的量。

雷诺传输定理：$$\frac{\mathrm{d}}{\mathrm{d}t}\int_\Omega \rho \mathrm{d}V + \int_{\partial\Omega}\rho \mathbf{u}\cdot\mathbf{n}\mathrm{d}S=0$$

散度定理：$$\int_{\partial\Omega}\rho \mathbf{u}\cdot\mathbf{n}\mathrm{d}S=\int_\Omega \nabla \cdot(\rho\mathbf{u})\mathrm{d}V$$

莱布尼茨积分法则：$$\frac{\mathrm{d}}{\mathrm{d}t}\int_\Omega \rho \mathrm{d}V = \int_\Omega \frac{\mathrm{d}\rho}{\mathrm{d}t} dV $$

整理得：$$\int_\Omega \frac{\mathrm{d}\rho}{\mathrm{d}t}  +  \nabla \cdot(\rho\mathbf{u})\mathrm{d}V=0$$

该式对任意曲面 $\partial\Omega$ 成立，因此： $$\frac{\mathrm{d}\rho}{\mathrm{d}t}  +  \nabla \cdot(\rho\mathbf{u}) = 0$$

### 动量守恒

由 $\mathbf{F} = m\mathbf{a}$ 可得：

$$
\begin{array}{rll}
\mathbf{f}=
\frac{\Delta\mathbf{F}} {\Delta V}=&
\rho \frac{\mathrm{d}\mathbf{u}}{\mathrm{d}t}
\\
=&\rho \frac{\mathrm{d}\mathbf{u}(x,y,z,t)}{\mathrm{d}t}
\\
=&
\rho\left(
\frac{\partial \mathbf{u}}{\partial x}\frac{\mathrm{d}x}{\mathrm{d}t} +
\frac{\partial \mathbf{u}}{\partial y}\frac{\mathrm{d}y}{\mathrm{d}t} +
\frac{\partial \mathbf{u}}{\partial z}\frac{\mathrm{d}z}{\mathrm{d}t} +
\frac{\mathrm{d}\mathbf{u}}{\mathrm{d}t}
\right)\\
=&
\rho \left(\mathbf{u}\cdot\nabla\mathbf{u} + \frac{\mathrm{d}\mathbf{u}}{\mathrm{d}t}\right)
\end{array}
$$

### 计算受力

$\mathbf{f}
=\mathbf{f}_{外力}+\mathbf{f}_{应力}=\mathbf{g}+\nabla \cdot \sigma$

其中 $\sigma$ 为应力张量。将其分解为体积应力张量与偏应力张量：

$$
\begin{array}{rll}
\sigma=&\begin{pmatrix}
\sigma_{xx}&\tau_{xy}&\tau_{xz}\\
\tau_{yx}&\sigma_{yy}&\tau_{yz}\\
\tau_{zx}&\tau_{zy}&\sigma_{zz}\\
\end{pmatrix}\\
=& -\left( \begin{array}{ccc}
    p &  0 & 0 \\
0 &  p & 0 \\
0 &  0 & p
\end{array} \right) +\left( \begin{array}{ccc}
    \sigma_{xx} + p &  \tau_{xy} & \tau_{xz} \\
\tau_{yx} &  \sigma_{yy} + p & \tau_{yz} \\
\tau_{zx} &  \tau_{zy} & \sigma_{zz} + p
\end{array} \right)\\
=&-p\mathrm{I} + \mathrm{T}
\end{array}
$$

整理得

$$
\mathbf{g} - \nabla p + \nabla \cdot  \mathrm{T}=\rho \left(\mathbf{u}\cdot\nabla\mathbf{u} + \frac{\mathrm{d}\mathbf{u}}{\mathrm{d}t}\right)
$$

可以进一步写成物质导数的表达方法：
$$
\rho \frac{\mathrm{D}\mathbf{u}}{\mathrm{D}t}=\mathbf{g}-\nabla p + \nabla \cdot T
$$

其中 $\nabla \cdot T$ 由流体性质决定，如对于牛顿流体，应力与变形程度成比例：
$$
\tau_{ij} = \mu\left(\frac{\partial u_i}{\partial x_j} + \frac{\partial u_j}{\partial x_i}\right)
$$
经进一步推导可得
$$
\nabla\cdot T = \mu \nabla^2 v
$$