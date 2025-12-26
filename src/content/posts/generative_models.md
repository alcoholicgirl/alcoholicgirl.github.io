---
title: 生成式模型杂谈
pubDate: '2025-12-19'
---

笼统地讲，生成式模型的任务是生成一个样本 $\hat{\mathbf{x}}$，使其近似地服从某个分布 $p(\mathbf{x})$。

在各种生成式的任务中，$p(\mathbf{x})$ 往往是未知且难以建模的，对应的样本空间 $\Omega$ 是巨大且复杂的，通常情况下我们仅可以管中窥豹地从中采样一系列样本 $\mathbf{x}_1, \mathbf{x}_2, \dots, \mathbf{x}_n$。

一个自然的想法是用 $f(\mathbf{x}; \theta)$ 直接拟合 $p(\mathbf{x})$，两者的 KL 散度为：$$\begin{array}{rl}\\ 
&D_{KL}(p\|f)\\
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

## 基于能量的模型
不管是经典的优化算法退火算法还是现代的扩散模型，都不约而同地从热力学中汲取了灵感——粒子更倾向于从能量高的地方转移到能量低的地方。基于能量的模型 (Energy Based Model, EBM) $E(\mathbf{x}; \theta)$ 并不直接拟合概率分布，它代表的是解空间中能量的分布情况。根据最大熵原理，玻尔兹曼分布是 “能量越低，概率越大” 这个约束下最无偏的分布：

$$
p(\mathbf{x};\theta) = \exp\left(-\frac{E(\mathbf{x};\theta)}{kT}\right)/Z_\theta
$$
$k$、$T$ 分别是玻尔兹曼常数与热力学温度。通常为了简化，使 $kT=1$：
$$
p(\mathbf{x};\theta) = \exp\left(-E(\mathbf{x};\theta)\right)/Z_\theta
$$
其中，$Z_\theta$ 是归一化常数，使 $p(\mathbf{x})$ 在整个积分域上积分为 $1$，其值等于 $\int \exp\left(-E(\mathbf{x};\theta)\right)d\mathbf{x}$。这个分布的对数似然为： $\log p(\mathbf{x};\theta) = -E(\mathbf{x};\theta) - \log Z_\theta$。第二项 $\log Z_\theta$ 涉及在整个积分域上积分，这也成了 EBM 的一个痛点。幸运的是，**分数函数**可以绕开这个无法计算的积分。

## 分数函数、Fisher 散度 
分数函数的定义为 $s(\mathbf{x};\theta) = \nabla_\mathbf{x} \log p(\mathbf{x};\theta)$。
对于 EBM，直接计算可得：$s(\mathbf{x};\theta)=-\nabla_\mathbf{x}\log E(\mathbf{x};\theta)$。注意 $Z_\theta=\int \exp\left(-\frac{E(\mathbf{x};\theta)}{kT}\right)d\mathbf{x}$ 与 $\mathbf{x}$ 无关，$\mathbf{x}$ 仅作为积分变量出现。 

对于分布 $p$ 与分布 $q$，若 $D_F(p\|q)=0$，易证明 $p(x) = q(x)$：
$$
\begin{array}{}
&\nabla_\mathbf{x} \left(\log p(\mathbf{x})-\log q(\mathbf{x})\right)=0\\
\implies & \log p(\mathbf{x})-\log q(\mathbf{x})=C\\
\implies & \log p(\mathbf{x}) = \log q(\mathbf{x}) + C\\
\implies & p(\mathbf{x})=q(\mathbf{x})\exp(C)
\end{array}
$$

由于 $p$、$q$ 满足归一化条件，可以知道二者相等。

此时不再用 KL 散度衡量两个分布的差异，而是以 Fisher 散度取而代之，模型的优化目标也就变成了：
$$
\frac{1}{2}
\mathbb{E}_{\mathbf{x}\sim p(\mathbf{x})} \left\|
    \nabla_{\mathbf{x}} \log p(\mathbf{x}) -
    s(\mathbf{x};\theta)
\right\|^2
$$

现在这个优化目标还不能拿来用，因为它包含了未知分布 $p(\mathbf{x})$ 的分数函数。不妨把这个期望展开：

$$
\mathbb{E}_{p}
\|\nabla_{\mathbf{x}} \log p(\mathbf{x})\|^2
+
\mathbb{E}_{p}
\|s(\mathbf{x};\theta)\|^2
-
2\mathbb{E}_{p}
\nabla_{\mathbf{x}} \log p(\mathbf{x})
\cdot s(\mathbf{x};\theta)
$$
其中第一项与参数 $\theta$ 无关，可以去掉：
$$
\begin{array}{rl}
&\mathbb{E}_{p}
\|s(\mathbf{x};\theta)\|^2
-
2\mathbb{E}_{p}
\nabla_{\mathbf{x}} \log p(\mathbf{x})
\cdot s(\mathbf{x};\theta)\\
=&
\mathbb{E}_{p}
\|s(\mathbf{x};\theta)\|^2
-
2\int p(\mathbf{x})
\nabla_{\mathbf{x}} \log p(\mathbf{x})
\cdot s(\mathbf{x};\theta)
d \mathbf{x}
\\
=&
\mathbb{E}_{p}
\|s(\mathbf{x};\theta)\|^2
-
2\int p(\mathbf{x})
\frac{
\nabla_{\mathbf{x}} p(\mathbf{x})
}{p(\mathbf{x})}
\cdot s(\mathbf{x};\theta)
d \mathbf{x}
\\
=&
\mathbb{E}_{p}
\|s(\mathbf{x};\theta)\|^2
-
2\int
\nabla_{\mathbf{x}} p(\mathbf{x})
\cdot s(\mathbf{x};\theta)
d \mathbf{x}
\\
\end{array}
$$
对于交叉项 $\int\nabla_{\mathbf{x}} p(\mathbf{x})\cdot s(\mathbf{x};\theta)d \mathbf{x}$，采用分布积分：
$$
\begin{array}{rl}
&\int
\nabla_{\mathbf{x}} p(\mathbf{x})
\cdot s(\mathbf{x};\theta)
d \mathbf{x}\\
=&
\int
\sum
\frac{
\partial p
}{
\partial x_i
}
s_i(\mathbf{x};\theta)d\mathbf{x}\\
=&
\int
\sum
\frac{
\partial p
}{
\partial x_i
}
s_i(\mathbf{x};\theta)dx_1\cdots dx_i\cdots dx_n\\
=&
\int
\sum

\left(

\int
s_i(\mathbf{x};\theta)
dp(\mathbf{x})

\right)dx_1\cdots dx_{i-1}dx_{i+1}\cdots dx_n\\
=&
\int
\sum
\left(
    \left[s_i(\mathbf{x};\theta)p(\mathbf{x})\right]_\partial
-\int
p(\mathbf{x})
ds_i(\mathbf{x};\theta)
\right)
dx_1\cdots dx_{i-1}dx_{i+1}\cdots dx_n\\

\end{array}
$$
适当地假设 $p(\mathbf{x})$ 在无穷远处趋于零，则其中的 $\left[s_i(\mathbf{x};\theta)p(\mathbf{x})\right]_\partial$ 为零：
$$
\begin{array}{rll}
&
\int
\sum
\left(
    \left[s_i(\mathbf{x};\theta)p(\mathbf{x})\right]_\partial
-\int
p(\mathbf{x})
ds_i(\mathbf{x};\theta)
\right)
dx_1\cdots dx_{i-1}dx_{i+1}\cdots dx_n\\
=&
\int
\sum
\left(
-\int
p(\mathbf{x})
ds_i(\mathbf{x};\theta)
\right)
dx_1\cdots dx_{i-1}dx_{i+1}\cdots dx_n\\
=&
\int
\sum
\left(
-\int
p(\mathbf{x})\frac{
\partial s_i(\mathbf{x};\theta)
}{
    \partial x_i
}
dx_i
\right)
dx_1\cdots dx_{i-1}dx_{i+1}\cdots dx_n\\
=&-
\int
p(\mathbf{x})
\sum
\frac{
    \partial s_i(\mathbf{x};\theta)
}{
    \partial x_i
}
d\mathbf{x}\\
=&-\mathbb{E}_{\mathbf{x}\sim p(\mathbf{x})}\nabla_{\mathbf{x}}^Ts(\mathbf{x};\theta)
\end{array}
$$

代入到原式中，得到最终的优化目标：
$$\frac{1}{2}
\mathbb{E}_{p}\left[
\|s(\mathbf{x};\theta)\|^2
+2\nabla_{\mathbf{x}}^Ts(\mathbf{x};\theta)
\right]
$$

## 流形假设与证据下界
早期的图片生成模型，如 Disco Diffusion，是直接在像素空间进行的。以 $512\times 512$ 的 RGB 图片生成为例，其维度达到了惊人的 $512 \times 512 \times 3 = 786432$ 维。要在这样的样本空间内训练一个生成式模型绝非易事。

流形假设提出，有意义的高维数据通常分布在一个低维的流形上。这个低维空间通常就是所谓的“潜在空间”(Latent Space)。毋庸置疑地，在潜在空间里训练模型比在原先高维的样本空间内要实惠很多。只要我们能找到一种在高低维间“编码解码”的办法，就能实现用低维空间代理高维空间，从而降本增效。

自编码器（Auto Encoder）正是最早尝试实现这种 “高低维映射” 的模型之一。它通过编码器将高维输入数据压缩到低维潜在空间，再通过解码器将潜在向量重构回高维数据空间，训练目标是最小化输入与重构输出的误差。这种训练模式是无监督的，仅需要无标签的输入样本 $\mathcal{x}$，就能通过 “编码——解码” 的闭环结构完成训练，学习到高维数据与潜在空间之间的映射。

但自编码器的致命弱点是没有天然的几何约束，它不能保证这种映射是平滑、连续的；其潜在空间的大部分点是无效的区域，将它们重构为高维数据时可能只能得到没有意义的噪声。

为了让潜在向量的分布服从某种已知的、连续的概率分布，2013 年，Diederik P. Kingma 和 Max Welling 提出了**变分自编码器**（VAE）。

VAE 的编码器学习 $q_\phi(\mathbf{z}\;|\;\mathbf{x})$（原始数据在潜在空间中的分布），解码器学习 $p_\theta(\mathbf{x}\;|\;\mathbf{z})$（从潜在空间重构后的高维数据在原始样本空间中的分布）。比起朴素的自编码器，对于样本空间 $\mathcal{X}$ 与潜在空间 $\mathcal{Z}$，VAE 不再学习 $\mathbf{x}\to \mathbf{z} \to \hat{\mathbf{x}}$ 的单一映射，而是学习一个平滑的分布。VAE 做出的一个重要假设是，这个分布是正态分布 $\mathcal{N}(\mu, \sigma^2)$。因此编码器具体的学习目标是 $\mu$ 与 $\sigma^2$。

在朴素的自编码器中，为了最小化输入与重构输出的误差，我们通常用 $\mathcal{L}_{loss}=|\hat{\mathbf{x}} - \mathbf{x}|^2$ 作为损失函数。但是这在 VAE 的框架下是不合理的——为了最小化这个损失函数，模型会尽可能地让 $\sigma$ 变小从而试图 “完美匹配” 原始数据，这导致原先的一切假设工作都前功尽弃，VAE 将退化为普通的自编码器。

假设有原始数据集 $\{\mathbf{x}_i\} \subset \mathcal{X}$，$\mathbf{x}_i$ 在潜在空间中对应的是 $\mathbf{z}_i$。我们的目标是最大化对数边缘似然函数： $\mathcal{L}(\theta)=\sum\log p_\theta(\mathbf{x}_i)=\sum\log \int p_\theta(\mathbf{x}_i,\mathbf{z})d\mathbf{z}$。但是正如上文提到的，在高于二维的空间内的整个积分域上积分是不可能的，这个边缘似然的计算是 “intractable” 的。我们希望能用某种方式避免直接计算这个边缘似然，例如计算它的一个下界：

$$
\begin{array}{rl}&\log p_\theta(\mathbf{x})\\
=&\log \int p_\theta(\mathbf{x}_i,\mathbf{z})d\mathbf{z}\\
=&\log \int \frac {p_\theta(\mathbf{x}_i,\mathbf{z})q_\phi(\mathbf{z}|\mathbf{x})}   {q_\phi(\mathbf{z}|\mathbf{x})}d\mathbf{z}\\
=&\log\mathbb{E}_{q_\phi(\mathbf{z}|\mathbf{x})}\left[
\frac {p_\theta(\mathbf{x}_i,\mathbf{z})}   {q_\phi(\mathbf{z}|\mathbf{x})}\right]\\
\ge& \mathbb{E}_{q_\phi(\mathbf{z}|\mathbf{x})}\left[\log 
\frac {p_\theta(\mathbf{x}_i,\mathbf{z})}   {q_\phi(\mathbf{z}|\mathbf{x})}\right]\\
\end{array}
$$
我们称它为证据下界（**E**vidence **L**ower **BO**und，**ELBO**）。实际上可以经过计算得知：

$$
\log p_\theta(\mathbf{x})=\mathbb{E}_{q_\phi(\mathbf{z}|\mathbf{x})}\left[\log \frac {p_\theta(\mathbf{x}_i,\mathbf{z})}   {q_\phi(\mathbf{z}|\mathbf{x})}\right]+D_{KL}(q_\phi(\mathbf{z}|\mathbf{x})\|p_\theta(\mathbf{x}))
$$
