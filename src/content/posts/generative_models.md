---
title: 生成式模型杂谈
pubDate: '2025-12-19'
---

笼统地讲，生成式模型的任务是生成一个样本 $\hat{\mathbf{x}}$，使其近似地服从某个分布 $p(\mathbf{x})$。

在各种生成式的任务中，$p(\mathbf{x})$ 往往是未知且难以建模的，对应的样本空间 $\Omega$ 是巨大且复杂的，通常情况下我们仅可以管中窥豹地从中采样一系列样本 $\mathbf{x}_1, \mathbf{x}_2, \dots, \mathbf{x}_n$。

一个自然的想法是用 $f(\mathbf{x}; \theta)$ 直接拟合 $p(\mathbf{x})$，两者的 KL 散度为：$$\begin{array}{rl}\\ 
&D_{KL}(p\;||\;f)\\
=&\mathbb{E}_{p(\mathbf{x})}\left[\log\left(\frac{p(\mathbf{x})}{f(\mathbf{x}; \theta)}\right)\right]\\
=&\mathbb{E}_{p(\mathbf{x})}\log p(\mathbf{x})-\mathbb{E}_{p(\mathbf{x})}\log f(\mathbf{x}; \theta)
\end{array}$$

其中 $\mathbb{E}_{p(\mathbf{x})}\log p(\mathbf{x})$ 与参数 $\theta$ 无关，**因此主要的优化目标是 $-\mathbb{E}_{p(\mathbf{x})}\log f(\mathbf{x}; \theta)$，即等价于最大化对数似然的期望**。这样我们就绕开了 $p(\mathbf{x})$，使得优化的过程仅与样本集 $\{\mathbf{x}_i\}$ 与参数有关。

## 建模分布函数
在对 $f(\mathbf{x};\theta)$ 建模时会遇到一些问题。

首先，一个合法的概率密度函数应当是归一化的、非负的：
- $\int_\Omega f(\mathbf{x})\;d\mathbf{x}=1$
- $f(\mathbf{x})\geq 0$

其次，就算 $q(\mathbf{x};\theta)$ 是合法的概率密度函数，从中采样也是一个非常困难的任务。

以一个一维分布为例。要在一维的随机分布中采样，通常的做法是计算其累计分布函数 $F(x) = \int_{-\infty}^{x}f(t)dt$，这样就可以用一个均匀随机变量 $a \sim U(0, 1)$ 采样，可以知道 $F^{-1}(a)$ 服从原先的分布。

对于更高维度的情况，这种朴素的方法可以依法炮制吗？不能。累积分布函数的计算需要在整个积分域上积分，而这在高于一维的情况中几乎是不可能的。

初露端倪的是，计算机可以不费吹灰之力地从简单的分布中采样。例如均匀分布、正态分布（利用 Box-Muller 方法将均匀分布映射到正态分布）。若能把这种简单的分布用某种方式变换成复杂的分布，从前者中采样就相当于在后者中采样。

例如，归一化流的思路就是把标准正态分布 $\mathcal{N}(0, 1)$ 变换为这样的复杂分布。这个变换是可逆的，从而能通过计算雅可比行列式保证变换后的分布同样是归一化的。

## VAE
早期的生成式模型，如 Disco Diffusion，是直接在像素空间进行的。倘若要生成一张 512x512 的单色图片，它的维度达到了惊人的 512 x 512 = 262144 维。要拟合这样高维度的分布绝非易事。事实上，根据流形假设，有意义的高维数据通常分布在一个低维的流形上。这个低维空间通常就是所谓的“潜在空间”。

2013 年，Diederik P. Kingma 和 Max Welling 提出了变分自编码器。
## 证据下界

## Score-Based Diffusion