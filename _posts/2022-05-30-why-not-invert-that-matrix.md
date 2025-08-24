---
title: 为什么要少用逆矩阵
categories: [machine learning]
tags: [machine learning, linear algebra]
permalink: /why-not-invert-that-matrix/
date: 2022-05-30 00:00:00
updated: 2022-06-02 00:00:00
---

<!-- toc -->

我经常会在线性代数教材以及论坛讨论中看到不建议使用逆矩阵$\mathbf{A}^{-1}$来求解线性方程$\mathbf{A}\mathbf{x}=\mathbf{b}$，尽管我一直遵循这样的原则（实践中逆矩阵确实不够稳定），但仍然不明白不使用逆矩阵的理由。本文总结了我在网上看到的一些关于逆矩阵的讨论，希望能解释为什么要少用逆矩阵来求解线性方程。<!--more-->

## 数学原理
### 解线性方程组（逆矩阵）

考虑求解线性方程组中的$\mathbf{x} \in \mathbb{R}^n$：

$$
\begin{equation}
  \mathbf{A} \mathbf{x} = \mathbf{b}
\end{equation}
$$

其中$\mathbf{A} \in \mathbb{R}^{n \times n},\mathbf{b} \in \mathbb{R}^n$。

显然一种简单直观的求解方法是计算$\mathbf{A}$的逆矩阵$\mathbf{A}^{-1}$：

$$
\begin{equation}
  \mathbf{x} = \mathbf{A}^{-1} \mathbf{b}
\end{equation}
$$

求解逆矩阵通常需要[$2n^3$][3]次浮点数操作（floating-point operations，flops），此外计算$\mathbf{A}^{-1}\mathbf{b}$需要$2n^2$ flops，总共$2n^3+2n^2$ flops。

不过实践中不推荐逆矩阵求解，因为可能在[准确性上产生问题][1]。求解线性方程更常用的是分解方法，比如接下来的LU分解。

### 解线性方程组（LU分解）

LU分解（lower–upper decomposition）是一种常用的矩阵分解方法，基本思路是利用高斯消元法将$\mathbf{A}$转化为上三角矩阵$\mathbf{U}$，主要过程如下所示：

$$
\begin{equation}\nonumber
  \overbrace{\begin{bmatrix}
      \times & \times & \times\\
      \times & \times & \times\\
      \times & \times & \times\\
  \end{bmatrix}}^{\mathbf{A}} \rightarrow
  \overbrace{\begin{bmatrix}
      \times & \times & \times\\
      0 & \times & \times\\
      0 & \times & \times\\
  \end{bmatrix}}^{\mathbf{L}_1\mathbf{A}} \rightarrow
  \overbrace{\begin{bmatrix}
      \times & \times & \times\\
      0 & \times & \times\\
      0 & 0 & \times\\
  \end{bmatrix}}^{\mathbf{L}_2\mathbf{L}_1\mathbf{A}}
\end{equation}
$$

其中$\mathbf{L}_1,\mathbf{L}_2$是一系列下三角矩阵，用来表示第n行加上第n-1行乘以某个数的消元操作，所以矩阵$\mathbf{A}$可以消元成为一个上三角矩阵$\mathbf{U}$：

$$
\begin{equation}
  \mathbf{L}_{n-1}\mathbf{L}_{n-2} \cdots \mathbf{L}_{1}\mathbf{A} = \mathbf{U}
\end{equation}
$$

矩阵$\mathbf{A}$的LU分解为：

$$
\begin{equation}
  \begin{split}
    \mathbf{A} &= \left(\mathbf{L}_{1}^{-1} \cdots \mathbf{L}_{n-2}^{-1} \mathbf{L}_{n-1}^{-1}\right) \mathbf{U}\\
    &= \mathbf{L} \mathbf{U}
  \end{split}
\end{equation}
$$

下三角矩阵的逆依然是下三角矩阵，并且一系列下三角矩阵的乘积也是一个三角矩阵，因此$\mathbf{L}$是下三角矩阵。此外，虽然涉及到求$\mathbf{L}_i$的逆，但是这一步骤实施起来是非常简单的，$\mathbf{L}_i$主对角线元素不变、下三角元素全部乘以-1即可得到$\mathbf{L}_i^{-1}$。因为$\mathbf{L}_i$每次都只对第$i$列上的主元做消元操作，所以下三角部分只会在第$i$列存在不为0的元素。以$3 \times 3$矩阵为例，$\mathbf{L}_1$可以写成如下的形式：

$$
\begin{equation}\nonumber
  \begin{bmatrix}
      1 & 0 & 0\\
      a & 1 & 0\\
      b & 0 & 1\\
  \end{bmatrix}
\end{equation}
$$

很自然的可以证明$\mathbf{L}_1^{-1}$为：

$$
\begin{equation}\nonumber
  \begin{bmatrix}
      1 & 0 & 0\\
      -a & 1 & 0\\
      -b & 0 & 1\\
  \end{bmatrix}
\end{equation}
$$

上述步骤的LU分解需要的浮点数操作近似为[$\frac{2}{3}n^3$][2] flops。

接下来我们看看如何利用LU分解解线性方程组。约定$\mathbf{A} \backslash \mathbf{b}$代表$\mathbf{A}\mathbf{x}=\mathbf{b}$的解$\mathbf{x}$（这也是MATLAB求解线性方程组的代码），按照以下步骤求解出$\mathbf{x}$：

$$
\begin{equation}
  \begin{split}
    \mathbf{L} \mathbf{U} &= \mathbf{A}\\
    \mathbf{L} \backslash \mathbf{b} &= \mathbf{y}\\
    \mathbf{U} \backslash \mathbf{y} &= \mathbf{x}\\
  \end{split}
\end{equation}
$$

所以先求解$\mathbf{L}\mathbf{y}=\mathbf{b}$得到中间变量$\mathbf{y}$，再求解$\mathbf{U}\mathbf{x}=\mathbf{y}$得到$\mathbf{x}$，看上去比直接求解$\mathbf{A}\mathbf{x} = \mathbf{b}$复杂了不少，为什么要这么做呢？

原因在于$\mathbf{L}$和$\mathbf{U}$作为三角矩阵能显著简化线性方程组求解，比如$\mathbf{L}\mathbf{y}=\mathbf{b}$，我们有：

$$
\begin{equation}
  \begin{bmatrix}
      1 & & & & \\
      \vdots&\ddots& & &\\
      l_{i,1}&\cdots&1& &\\
      \vdots&\vdots&\vdots&\ddots&\\
      l_{n,1}&\cdots&\cdots&\cdots&1\\
  \end{bmatrix} 

  \begin{bmatrix}
      y_1\\
      \vdots\\
      y_i\\
      \vdots\\
      y_n\\
  \end{bmatrix} = 

  \begin{bmatrix}
      b_1\\
      \vdots\\
      b_i\\
      \vdots\\
      b_n\\
  \end{bmatrix}  

\end{equation}
$$

可以用递归方法求解$\mathbf{y}$：

$$
\begin{equation}
  \begin{split}
    y_1 &= b_1\\
    y_i &= b_i - l_{i,1}y_1 - \cdots - l_{i, i-1}y_{i-1}
  \end{split}
\end{equation}
$$

这种思路叫forward substitution，即从上往下求解$\mathbf{y}$。之后可以用相似的方式从下往上求解$\mathbf{x}$，叫backward substitution。forward substitution的成本为（每一个元素各需要$i-1$次乘法和减法）$n^2-n$ flops，同理可得backward substitution的计算成本为$n^2+n$ flops，所以用LU分解求解线性方程组的总计算成本为$\frac{2}{3}n^3+2n^2$ flops。

### 解线性方程组（QR分解）

QR分解（QR decomposition）也常用于求解线性方程组。QR分解将$\mathbf{A} \in \mathbb{R}^{n \times n}$分解为正交矩阵$\mathbf{Q} \in \mathbb{R}^{n \times n}$和上三角矩阵$\mathbf{R} \in \mathbb{R}^{n \times n}$：

$$
\begin{equation}
  \mathbf{A} = \mathbf{Q} \mathbf{R}
\end{equation}
$$

对于正交矩阵$\mathbf{Q}$，它的逆矩阵$\mathbf{Q}^{-1}$实际就是$\mathbf{Q}^T$本身，所以对于线性方程组可以简化为：

$$
\begin{equation}
  \begin{split}
    \mathbf{A} \mathbf{x} &= \mathbf{b}\\
    \mathbf{Q}\mathbf{R}\mathbf{x} &= \mathbf{b}\\
    \mathbf{R}\mathbf{x} &= \mathbf{Q}^T\mathbf{b}\\
  \end{split}
\end{equation}
$$

因为$\mathbf{R}$是上三角矩阵，对于$\mathbf{x}$的求解可以采用LU分解中介绍的backward substitution方式求解，从而避免了求逆的操作。

QR分解可以采用[Gram-Schmidt正交化][8]过程计算得到，这一过程与正交的几何表示密切相关，因此易于理解，不过这种正交化过程容易数值不稳定，因此很少直接在实践中使用。实践中更常用的是使用[Givens rotations][8]方法来实现QR分解。

Givens rotations的步骤与LU分解相似，对于矩阵$\mathbf{A}$，采用一系列旋转矩阵$\mathbf{G}_i^j$将$\mathbf{A}$逐步转化为上三角矩阵：

$$
\begin{equation}\nonumber
  \overbrace{\begin{bmatrix}
      \times & \times & \times\\
      \times & \times & \times\\
      \times & \times & \times\\
  \end{bmatrix}}^{\mathbf{A}} \rightarrow
  \overbrace{\begin{bmatrix}
      \times & \times & \times\\
      \times & \times & \times\\
      0 & \times & \times\\
  \end{bmatrix}}^{\mathbf{G}_3^1\mathbf{A}} \rightarrow
  \overbrace{\begin{bmatrix}
      \times & \times & \times\\
      0 & \times & \times\\
      0 & \times & \times\\
  \end{bmatrix}}^{\mathbf{G}_2^1\mathbf{G}_3^1\mathbf{A}} \rightarrow
  \overbrace{\begin{bmatrix}
      \times & \times & \times\\
      0 & \times & \times\\
      0 & 0 & \times\\
  \end{bmatrix}}^{\mathbf{G}_3^2\mathbf{G}_2^1\mathbf{G}_3^1\mathbf{A}}
\end{equation}
$$

对于需要置0的第$a_{ij}$个元素$(i \gt j)$，$\mathbf{G}_i^j$构造为：

$$
\begin{equation}
  \mathbf{G}_i^j = 
  \begin{bmatrix}
      1 & \cdots & 0 & \cdots & 0 & \cdots & 0\\
      \vdots & \ddots & \vdots &  & \vdots &  & \vdots\\
      0 & \cdots & c_{jj} & \cdots & -s_{ji} & \cdots & 0\\
      \vdots &  & \vdots & \ddots & \vdots &  & \vdots\\
      0 & \cdots & s_{ij} & \cdots & c_{ii} & \cdots & 0\\
      \vdots &  & \vdots &  & \vdots & \ddots & \vdots\\
      0 & \cdots & 0 & \cdots & 0 & \cdots & 1\\
  \end{bmatrix}
\end{equation}
$$

其中$c_{jj} = c_{ii} = \cos(\theta)$，$s_{ij} = s_{ji} = \sin(\theta)$。显而易见这是一个对$ij$平面上的向量进行旋转的矩阵，对于任意向量，若想将其在第$i$个轴上的分量置0，则需要将其按$\theta$角旋转至对应的坐标轴上，旋转变化不会改变除第$i$、$j$行外的其他元素，因此通过反复运用旋转变换，可以将$\mathbf{A}$的下三角区域置0，有：

$$
\begin{equation}
  \mathbf{G}_n^{n-1} \cdots \mathbf{G}_{n-1}^1 \mathbf{G}_n^1 \mathbf{A} = \mathbf{R}
\end{equation}
$$

又因为旋转矩阵本身就是正交矩阵，所以其逆就是矩阵转置本身，所以有：

$$
\begin{equation}
  \begin{split}
  \mathbf{A} &= \left(\mathbf{G}_n^{n-1} \cdots \mathbf{G}_{n-1}^1 \mathbf{G}_n^1\right)^T \mathbf{R}\\
             &= \mathbf{Q}\mathbf{R}
  \end{split}
\end{equation}
$$

Givens rotations实现的QR分解的复杂度大约是[$\frac{7}{3}n^3$ flops][9]（我也不确定），优势在于可以较为容易的实现并行化操作。

### 复杂度分析

从上面的分析中可以看出，LU分解大概需要$\frac{2}{3}n^3$ flops，逆矩阵大概需要$2n^3$ flops，是LU分解的3倍，因此线性方程组求解采用逆矩阵方法会更慢一些。尽管逆矩阵求解存在[更快的算法][4]，但是通常来说LU分解还是更有效率，所以一般而言还是应当直接求解线性方程组而不是计算逆矩阵。

此外，[逆矩阵计算往往需要更大的内存][1]。如果$\mathbf{A}$是稀疏矩阵（大多数元素是0），则可以使用更加有效率的稀疏结构存储。但是$\mathbf{A}^{-1}$通常是稠密的，因此存储逆矩阵需要更多的内存。

### 准确性分析

逆矩阵的另一个缺点在于求解的误差更大，特别在$\mathbf{A}$是病态矩阵的情况下尤为明显。

数值分析领域常用前向误差（forward error）和反向误差（backward error）来分析误差。比如对于函数$y=f(x)$，用$\hat{y}$来近似输出$y$，那么前向误差被定义为$\|\hat{y}-y\|$（这似乎就是我们常说的误差），而[反向误差则衡量问题的敏感性][6]，即为了产生近似解，输入数据同真实数据的偏移情况。$\hat{y}$的反向误差是满足$\hat{y}=f(x+\Delta x)$的最小$\Delta x$。

前向误差和反向误差联合起来可以分析一个问题是良态（well-conditioned）还是病态（ill-conditioned）的。对于良态问题，输入较小的改变会导致输出产生较小的改变；而病态问题，输入较小的改变会导致输出较大的改变。[条件数（condition number）][5]就是用来判断良态和病态的工具，条件数越大，输入很小的差异就会产生较大的输出误差，更倾向于病态问题。矩阵的条件数$\kappa(\mathbf{A})$定义为：

$$
\begin{equation}
  \kappa(\mathbf{A}) = \|\mathbf{A}\| \|\mathbf{A}^{-1}\|
\end{equation}
$$

如果采用矩阵的2范数，上式可以进一步简化为最大特征值$\lambda_1$与最小特征值$\lambda_n$之比：

$$
\begin{equation}
  \kappa(\mathbf{A}) = \frac{\lambda_1}{\lambda_n}
\end{equation}
$$

[stackexchange][7]和[这篇blog][2]详细分析了前向误差$\|\mathbf{x}-\hat{\mathbf{x}}\|$和反向误差$\|\mathbf{b}-\mathbf{A}\hat{\mathbf{x}}\|$（注意这里$\mathbf{x}$是我们求解问题的输出），有如下结论：

- 对于良态矩阵$\mathbf{A}$，LU分解的前向误差和逆矩阵的前向误差是很接近的
- 对于病态矩阵$\mathbf{A}$，逆矩阵的反向误差会远大于LU分解的反向误差
- 即使对于良态矩阵$\mathbf{A}$，逆矩阵的反向误差也比LU分解的反向误差更大

此外，前向误差和反向误差有如下不等式关系：

$$
\begin{equation}
  \|\mathbf{x}-\hat{\mathbf{x}}\| \le \kappa(\mathbf{A}) \|\mathbf{b}-\mathbf{A}\hat{\mathbf{x}}\|
\end{equation}
$$

综合来看的话，利用LU分解、QR分解等方法直接求解线性方程组比逆矩阵求解要更加准确。


[1]: https://www.johndcook.com/blog/2010/01/19/dont-invert-that-matrix/
[2]: http://gregorygundersen.com/blog/2020/12/09/matrix-inversion/
[3]: https://math.stackexchange.com/questions/3528736/why-is-lu-preferred-over-a-1-to-solve-matrix-equations
[4]: https://en.wikipedia.org/wiki/Computational_complexity_of_mathematical_operations#endnote_blockinversion
[5]: https://en.wikipedia.org/wiki/Condition_number
[6]: https://nhigham.com/2020/03/25/what-is-backward-error/
[7]: https://math.stackexchange.com/questions/2397476/the-benefit-of-lu-decomposition-over-explicitly-computing-the-inverse
[8]: https://en.wikipedia.org/wiki/QR_decomposition
[9]: https://people.inf.ethz.ch/arbenz/ewp/Lnotes/chapter4.pdf