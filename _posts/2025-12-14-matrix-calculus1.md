---
title: Complex-valued Matrix Differentiation
categories: [machine learning,]
tags: [machine learning,]
permalink: /matrix-calculus1/
date: 2025-12-14 00:00:00
updated: 2025-12-28 00:00:00
toc: true
---

<!-- toc -->

This post summerizes some common tricks of complex-valued matrix differentiaion.<!--more--> The post is based on the paper *[Complex-Valued Matrix Differentiation: Techniques and Key Results](https://ieeexplore.ieee.org/document/4203075/)*, following the same numerator layout and differential way used in my last [post](https://mrswolf.github.io/matrix-calculus/).

## Why complex derivatives may not exist 

Consider a simple complex-valued function $f(z)=\|z\|^2$. For the complex derivative to exist, the limit must be the same in all directions. However, it's easy to see that, if $z$ goes to 0 along the real axis, the limit is $2x$; if z goes to 0 along the imaginary axis, the limit is $2y$. This differenence indicates that the strict complex derivative doesn't exist for this function.

For the existence of the complex derivative, the function should satisfy the Cauchy-Riemann equations:

$$
\begin{equation}
  \begin{split}
    \partial u / \partial x &= \partial v / \partial y\\
    \partial u / \partial y &= -\partial v / \partial x
  \end{split}
\end{equation}
$$

where $f(z) = u(x,y) + i v(x, y), z=x+iy$. Functions satisfing the Cauchy-Riemann equations are holomorphic. Unfortunately, most cost functions in complex optimization problems are not holomorphic. In complex analysis, the only holomorphic functions that take complex inputs and give real outputs are constant functions. 

## Wirtinger Calculus

Though these functions are not holomorphic in the strict sense, they are real-differentiable if we treat the real and imaginary parts as separate variables. 

Wirtinger calculus provides a way to repackage the partial derivatives with respect to the real values ($x$ and $y$) into derivatives with respect to the complex values ($z$ and $\bar{z}$). This allows us to apply the elegant symbolic rules of linear algebra directly to complex variables.

Consider a function $f(\mathbf{z})$, where $\mathbf{z}$ is a complex vector, the Wirtinger calculus follows the following steps:

1. Compute the differential $df$
2. Manipulate the expression into the form $df = D_{\mathbf{z}}f d\mathbf{z} + D_{\bar{\mathbf{z}}}f d\bar{\mathbf{z}}$
3. Read out the partial derivatives $\partial f/ \partial \mathbf{z} = D_{\mathbf{z}}f$ and $\partial f/ \partial \bar{\mathbf{z}} = D_{\bar{\mathbf{z}}}f$

The complex differential rules are the same as those in real differentials. For example, the chain rule is still valid. Consider a function $h(\mathbf{z}, \bar{\mathbf{z}})=f(g(\mathbf{z}, \bar{\mathbf{z}}))$, the partial derivatives are:

$$
\begin{equation}
  \begin{split}
    D_{\mathbf{z}}h &= D_{g}f D_{\mathbf{z}}g + D_{\bar{g}}f D_{\mathbf{z}} \bar{g}\\
    D_{\bar{\mathbf{z}}}h &= D_{g}f D_{\bar{\mathbf{z}}}g + D_{\bar{g}}f D_{\bar{\mathbf{z}}} \bar{g}
  \end{split}
\end{equation}
$$

For complex-valued optimization problem, the stationary points are found by the following two equivalent equations:

$$
\begin{equation}
  \begin{split}
    D_{\mathbf{z}}f &= \mathbf{0}\\
    D_{\bar{\mathbf{z}}}f &= \mathbf{0} \\
  \end{split}
\end{equation}
$$

or

$$
\begin{equation}
  \begin{split}
    D_{\mathbf{z}}f &= \mathbf{0}\\
  \end{split}
\end{equation}
$$






