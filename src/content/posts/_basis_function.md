---
title: 基函数杂谈
pubDate: '2025-12-25'
---
笔者在学拉普拉斯变换时有一个疑惑。在计算$F(s)=\int^{+\infty}_{0}f(t)\exp(-st)dt$ 时用到了一个结论：
$$
\int^{+\infty}_{0} \exp(-st) dt = \frac{1}{s}
$$

这在 $\text{Re}(s)<0$ 时完全不收敛，为什么要 “解析延拓” 到整个复平面上呢？

其实要从    “拉普拉斯变换在干什么” 这个角度理解。拉普拉斯变换试图将 $f(t)$ 分解为一系列**基函数**：
$$
f(t) = \sum_k C_k \exp(s_k t)
$$
可得：
$$
\begin{array}{rl}
\mathcal{L}\{f(t)\} &= \int f(t)\exp(-s t)dt\\
&= \int \left(\sum C_k \exp(s_k t)\right)\exp(-s t)dt\\
&= \sum \left(\int C_k\exp(s_kt)\exp(-s t)dt\right)\\
\end{array}
$$
当 $s = s_k$ 时，$\mathcal{L}(s)$ 因含有一项 $\int^{+\infty} _{0} C_k dt$ 而发散。此时的 $s$ 是 $s$ 平面上的奇点。

当 $s \neq s_k$ 时，这个级数的每一项变为 $C_k\int \exp\left((s_k-s)t\right) dt$。
