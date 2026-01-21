---
title: NeRF、高斯渲染
pubDate: '2025-12-31'
---

早在2001年，Matthias Zwicker 等研究者在《EWA Volume Splatting》中就提出了基于椭圆高斯核的体积 splatting 渲染框架，奠定了现代 3D Gaussian Splatting 渲染管道的核心基础。

3DGS 拟合的是一个 Radience Field，其核心是将场景中的每个 3D Gaussian 原语高效投影并混合至 2D 图像平面。每个高斯原语定义为一个多变量高斯分布：

$$
G(\mathbf{x}) = \exp(-\frac{1}{2}\mathbf{x}^T\Sigma^{-1}\mathbf{x})
$$