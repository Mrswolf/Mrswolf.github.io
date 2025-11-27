---
title: Convolution, Cross-Correlation and their Adjoints
date: 2025-09-25 00:00:00
update: 2025-09-25 00:00:00
categories: [machine learning, magnetic resonance imaging]
tags: [machine learning, magnetic resonance imaging]
permalink: /conv-corr-adjoint/
toc: true
---

<!-- toc -->

Last week I was reading Lustig and Pauly's classical paper *[SPIRiT: Iterative Self-consistent Parallel Imaging Reconstruction from Arbitrary k-Space](https://onlinelibrary.wiley.com/doi/abs/10.1002/mrm.22428)* while also working through their matlab code. I realized that the relationship between convolution and cross-correlation in the complex domain is important for understanding many MRI reconstruction implementaions, since these operations are often used interchangeably. <!-- more -->

## Convolution and Cross-Correlation
For complex-valued functions $f$ and $h$, convolution is defined as:

$$
\begin{equation}
\begin{split}
(h * f)(t) &= \int h(t-\tau)f(\tau) d\tau \\
\end{split}
\end{equation}
$$

For the same functions $f$ and $h$ , cross-correlation is defined as:

$$
\begin{equation}
\begin{split}
(h \star f)(t) &= \int h^*(\tau-t)f(\tau) d\tau \\
\end{split}
\end{equation}
$$

We can use one to represent the other with the following equalities:

$$
\begin{equation}
\begin{split}
(h * f)(t) &= \int h(t-\tau)f(\tau) d\tau \\
&= \int (h^*(-(\tau-t)))^*f(\tau) d\tau \\
&= \int \hat{h}^*(\tau-t)f(\tau) d\tau,\ \hat{h}(t) = h^*(-t)\\
&= (\hat{h} \star f)\\

(h \star f)(t) &= \int h^*(\tau-t)f(\tau) d\tau \\
&= \int h^*(-(t-\tau))f(\tau) d\tau \\
&=\int \hat{h}(\tau-t)f(\tau) d\tau,\ \hat{h}(t)=h^*(-t) \\
&= (\hat{h} * f)(t)
\end{split}
\end{equation}
$$

In other words, the convolution of $h$ and $f$ can be obtained by flipping and conjugating $h$, and then performing a cross-correlation with $f$; the cross-correlation of $h$ and $f$ can be obtained by flipping and conjugating $h$, and then performing a convolution with $f$.

## The Adjoint Operators
An operator $A$ and its adjoint $A^*$ should satisfy the following equation:

$$
\begin{equation}
\begin{split}
\langle Ax, y\rangle = \langle x, A^*y \rangle
\end{split}
\end{equation}
$$

<p>
The convolution operator $C$ convolves the input $x$ with a specified kernel $h$, while its adjoint $C^*$ performs the cross-correlation of $y$ with the same kernel. As shown previously, cross-correlation can also be implemented as a convolution with a flipped and conjugated kernel $\hat{h}=h^*(-t)$.
</p>

Equivalently, the adjoint of cross-correlation is the convolution of $y$ with the kernel $h$.

## Speeding Up Convolution with FFT
Using the FFT is a common way to accelerate convolution, since FFT implementations are highly optimized on modern hardware. Convolution Theorem is the core to achieve the speedup:

$$
\begin{equation}
\begin{split}
\mathcal{F}((h * f))(k) &= \mathcal{F}(h)(k) \cdot F(f)(k)\\
\end{split}
\end{equation}
$$

When it comes to the cross-correlation, the formula is as follows:

$$
\begin{equation}
\begin{split}
\mathcal{F}((h \star f))(k) &= \mathcal{F}((\hat{h} * f))(k)\\
&=\mathcal{F}(\hat{h})(k) \cdot \mathcal{F}(f)(k)\\
&= \mathcal{F}(h^*(-t))(k) \cdot \mathcal{F}(f)(k)\\
&= \mathcal{F}(h)^*(k) \cdot \mathcal{F}(f)(k)
\end{split}
\end{equation}
$$

The last equality is due to the following properties:

$$
\begin{equation}
\begin{split}
\mathcal{F}(h^*(t))(k) &= \mathcal{F}(h)^*(-k)\\
\mathcal{F}(h(-t))(k) &= \mathcal{F}(h)(-k)\\
\mathcal{F}(h^*(-t))(k) &= \mathcal{F}(h)^*(k)\\
\end{split}
\end{equation}
$$

