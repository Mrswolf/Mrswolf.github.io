---
title: 矩阵微积分
categories: [machine learning,]
tags: [machine learning,]
permalink: matrix-calculus
date: 2022-03-28 00:00:00
updated: 2022-03-31 00:00:00
---

<!-- toc -->

矩阵微分和矩阵求导几乎是求解优化问题不可避免的必学内容，这一方面的内容老实说我很难完全掌握。这里记录一下一些常用的矩阵微分求导的规范和技巧。<!-- more -->

## 符号约定和布局规范

首先微分（differential）是在自变量微小变化下造成的因变量的微小变化，而导数（derivative）则是这种变化的速率。联系导数和微分的方程式叫做微分方程（differential equation）。比如函数$y=\mathrm{f}(x)$，$x$的微小变化用符号$dx$表示，导致的$y$的微小变化用$dy$表示，$x$的变化引起的$y$的变化的速率用$\frac{\partial y}{\partial x}$表示，函数$\mathrm{f}(x)$的微分方程就是：

$$
\begin{equation}
  dy=\frac{\partial y}{\partial x} dx
\end{equation}
$$

矩阵微分/导数同函数微分/导数基本一致，只不过现在输入输出都是矩阵（向量也是矩阵的一种）的表现形式，比如$\mathbf{y} = \mathrm{f}(\mathbf{x})$。这里用加粗大写字母表示矩阵，例如$\mathbf{A}$；用加粗小写字母表示向量，例如$\mathbf{a}$；用不加粗小写字母表示标量，例如$a$。

矩阵微分求导的难点在于没有固定的布局规范，导致有些文章和教材看起来互相冲突。比如对于$\frac{\partial \mathbf{y}}{\partial \mathbf{x}}$，其中$\mathbf{y} \in \mathbb{R}^{m \times 1}$，$\mathbf{x} \in \mathbb{R}^{n \times 1}$，向量对向量的导数用矩阵形式可以有两种表示布局方案：

- **分子布局（numerator layout）**，这种布局要求$\mathbf{y}$是按列排的，$\mathbf{x}$是按行排列的（即$\mathbf{x}^T$），最终输出$\frac{\partial \mathbf{y}}{\partial \mathbf{x}} \in \mathbb{R}^{m \times n}$，我觉得可以简单理解为与分子上矩阵元素相关的输出排列规则不变（跟原来一致），而与分母上矩阵元素相关输出排列规则应当是原来元素的转置。
- **分母布局（denominator layout）**，这种布局要求$\mathbf{y}$是按行排的（即$\mathbf{y}^T$），$\mathbf{x}$是按列排列的，最终输出$\frac{\partial \mathbf{y}}{\partial \mathbf{x}} \in \mathbb{R}^{n \times m}$，正好跟分子布局相反。

为了方便推导和记忆，我选择采用分子布局，那么矩阵间导数的形式应当是这样的：

|  |$y$|$\mathbf{y} \in \mathbb{R}^{m \times 1}$|$\mathbf{Y} \in \mathbb{R}^{m \times n}$|
|:-:|:-:|:-:|:-:|
|$x$|$\frac{\partial y}{\partial x}$|$\frac{\partial \mathbf{y}}{\partial x} \in \mathbb{R}^{m \times 1}$|$\frac{\partial \mathbf{Y}}{\partial x} \in \mathbb{R}^{m \times n}$|
|$\mathbf{x} \in \mathbb{R}^{p \times 1}$|$\frac{\partial y}{\partial \mathbf{x}} \in \mathbb{R}^{1 \times p}$|$\frac{\partial \mathbf{y}}{\partial \mathbf{x}} \in \mathbb{R}^{m \times p}$||
|$\mathbf{X} \in \mathbb{R}^{p \times q}$|$\frac{\partial y}{\partial \mathbf{X}} \in \mathbb{R}^{q \times p}$|||

## 微分规则

矩阵求导常需要用到sum rule、product rule和chain rule，其中链式法则不适用于matrix-by-scalar或scalar-by-matrix的形式，所以直接复合函数求导蛮麻烦的。[wiki][1]上说可以先用微分的规则求微分，然后再转换成导数的形式。

sum rules：

$$
\begin{equation}
  \begin{split}
    h(x)  &= f(x) + g(x)\\
    dh(x) &= df(x) + dg(x)\\
  \end{split}
\end{equation}
$$

product rules：

$$
\begin{equation}
  \begin{split}
    h(x) &= f(x)g(x)\\
    dh(x) &= df(x)g(x) + f(x)dg(x)\\
  \end{split}
\end{equation}
$$

chain rules：

$$
\begin{equation}
  \begin{split}
    h(x) &= f(g(x))\\
    dh(x) &= f(g(x+dx)) - f(g(x))\\
          &= f(g(x)+dg(x)) - f(g(x))\\
          &= df(y)|_{y=g(x),dy=dg(x)}\\
  \end{split}
\end{equation}
$$

trace tricks：

$$
\begin{equation}
  \begin{split}
    a &= \mathrm{tr}(a)\\
    \mathrm{tr}(\mathbf{A}) &= \mathrm{tr}(\mathbf{A}^T)\\
    \mathrm{tr}(\mathbf{A}+\mathbf{B}) &= \mathrm{tr}(\mathbf{A}) + \mathrm{tr}(\mathbf{B})\\
    \mathrm{tr}(\mathbf{A}\mathbf{B}\mathbf{C}) &= \mathrm{tr}(\mathbf{B}\mathbf{C}\mathbf{A})\\
    &= \mathrm{tr}(\mathbf{C}\mathbf{A}\mathbf{B})\\
  \end{split}
\end{equation}
$$

kronecker product rules:

$$
\begin{equation}
  \begin{split}
  \mathbf{A}\otimes(\mathbf{B}+\mathbf{C}) &= \mathbf{A}\otimes\mathbf{B} + \mathbf{A}\otimes\mathbf{C}\\
  (k\mathbf{A})\otimes\mathbf{B} &= \mathbf{A}\otimes(k\mathbf{B})=k(\mathbf{A}\otimes\mathbf{B})\\
  (\mathbf{A}\otimes\mathbf{B})\otimes\mathbf{C} &= \mathbf{A}\otimes(\mathbf{B}\otimes\mathbf{C})\\
  (\mathbf{A}\otimes\mathbf{B})(\mathbf{C}\otimes\mathbf{D}) &= (\mathbf{A}\mathbf{C})\otimes(\mathbf{B}\mathbf{D})\\
  (\mathbf{A}\otimes\mathbf{B})\circ(\mathbf{C}\otimes\mathbf{D}) &= (\mathbf{A}\circ\mathbf{C})\otimes(\mathbf{B}\circ\mathbf{D})\\
  (\mathbf{A}\otimes\mathbf{B})^{-1} &= \mathbf{A}^{-1} \otimes \mathbf{B}^{-1}\\
  (\mathbf{A}\otimes\mathbf{B})^{T} &= \mathbf{A}^{T} \otimes \mathbf{B}^{T}\\
  \mathrm{tr}(\mathbf{A}\otimes\mathbf{B}) &= \mathrm{tr}(\mathbf{A})\mathrm{tr}(\mathbf{B})\\
  \end{split}
\end{equation}
$$

hadamard product rules:

$$
\begin{equation}
  \begin{split}
  \mathbf{A}\circ\mathbf{B} &= \mathbf{B}\circ\mathbf{A}\\
  \mathbf{A}\circ(\mathbf{B}+\mathbf{C}) &= \mathbf{A}\circ\mathbf{B} + \mathbf{A}\circ\mathbf{C}\\
  (k\mathbf{A})\circ\mathbf{B} &= \mathbf{A}\circ(k\mathbf{B})=k(\mathbf{A}\circ\mathbf{B})\\
  (\mathbf{A}\circ\mathbf{B})\circ\mathbf{C} &= \mathbf{A}\circ(\mathbf{B}\circ\mathbf{C})\\
  \end{split}
\end{equation}
$$

总结矩阵常用的微分规则如下：

|说明|表达式|微分结果|
|:--|:----|:------|
|$\mathbf{A}$不是$\mathbf{X}$的函数|$d\left(\mathbf{A}\right)$|$\mathbf{0}$|
|$a$不是$\mathbf{X}$的函数|$d(a\mathbf{X})$|$ad\mathbf{X}$|
||$d(\mathbf{X} \otimes \mathbf{Y})$|$(d\mathbf{X}) \otimes \mathbf{Y} + \mathbf{X} \otimes (d\mathbf{Y})$|
||$d(\mathbf{X} \circ \mathbf{Y})$|$(d\mathbf{X}) \circ \mathbf{Y} + \mathbf{X} \circ (d\mathbf{Y})$|
||$d(\mathbf{X}^T)$|$(d\mathbf{X})^T$|
|共轭转置|$d(\mathbf{X}^H)$|$(d\mathbf{X})^H$|
||$d(\mathbf{X}^{-1})$|$-\mathbf{X}^{-1}(d\mathbf{X})\mathbf{X}^{-1}$|
|$n$是正整数|$d(\mathbf{X}^n)$|$\sum_{i=0}^{n-1} \mathbf{X}^i(d\mathbf{X})\mathbf{X}^{n-1-i}$|
||$d(e^{\mathbf{X}})$|$\int_0^1 e^{a\mathbf{X}}(d\mathbf{X}) e^{(1-a)\mathbf{X}}da$|
||$d(\mathrm{log}(\mathbf{X}))$|$\int_0^{\infty} (\mathbf{X}+z\mathbf{I})^{-1}(d\mathbf{X})(\mathbf{X}+z\mathbf{I})^{-1}dz$|
||$d(\mathrm{tr}(\mathbf{X}))$|$\mathrm{tr}(d\mathbf{X})$|
||$d(\det(\mathbf{X}))$|$\det(\mathbf{X})\mathrm{tr}(\mathbf{X}^{-1}d\mathbf{X})$|
||$d(\log(\det(\mathbf{X})))$|$\mathrm{tr}(\mathbf{X}^{-1}d\mathbf{X})$|

## 微分-导数转换

在获得表达式的微分形式后，可以按如下规则进行导数形式的转化：

|微分形式|导数形式|
|:-----:|:----:|
|$dy=adx$|$\frac{\partial y}{\partial x}=a$|
|$dy=\mathbf{a}^Td\mathbf{x}$|$\frac{\partial y}{\partial \mathbf{x}}=\mathbf{a}^T$|
|$dy=\mathrm{tr}(\mathbf{A}d\mathbf{X})$|$\frac{\partial y}{\partial \mathbf{X}}=\mathbf{A}$|
|$d\mathbf{y}=\mathbf{a}dx$|$\frac{\partial \mathbf{y}}{\partial x}=\mathbf{a}$|
|$d\mathbf{y}=\mathbf{A} d\mathbf{x}$|$\frac{\partial \mathbf{y}}{\partial \mathbf{x}}=\mathbf{A}$|
|$d\mathbf{Y}=\mathbf{A}dx$|$\frac{\partial \mathbf{Y}}{\partial x}=\mathbf{A}$|


[1]: https://en.wikipedia.org/wiki/Matrix_calculus