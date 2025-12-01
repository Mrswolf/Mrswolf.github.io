---
title: FFTShift without Moving Elements
date: 2025-12-01 00:00:00
update: 2025-12-01 00:00:00
categories: [machine learning, magnetic resonance imaging, c++]
tags: [machine learning, magnetic resonance imaging, c++]
permalink: /fast-fftshift/
toc: true
---

<!-- toc -->

My [previous post](https://mrswolf.github.io/circshift/) explains how to implement an inplace `fftshift/ifftshift` in C++. However, shifting still requires moving elements, which can be costly for large arrays. Fortunately, when `fftshift/ifftshift` are used together with the FFT/iFFT, there is a way to avoid any physical data movement. <!-- more -->

## Some Facts
Let $S = \lfloor N/2 \rfloor$ for an length-$N$ signal $x[n]$, `fftshift` is defined as:

$$
\begin{equation}
\begin{split}
\hat{x}[n] = x[(n - S) \bmod N]
\end{split}
\end{equation}
$$

and `ifftshift` is defined as:

$$
\begin{equation}
\begin{split}
\hat{x}[n] = x[(n + S) \bmod N]
\end{split}
\end{equation}
$$

An important property for $\bmod$:
$$
\begin{equation}
\begin{split}
e^{\pm i 2\pi \alpha \frac{(n \pm S) \bmod N}{N}} = e^{\pm i 2\pi \alpha \frac{n \pm S}{N}}
\end{split}
\end{equation}
$$

In addition, the DFT’s time-shifting and frequency-shifting properties show that shifting in one domain is equivalent to applying a linear phase modulation in the other domain.

## fftshift-fft-ifftshift
This combination of operations can be evaluated by first taking the inner `fft(ifftshift(x))` part with the time-shifting property:

$$
\begin{equation}
\begin{split}
Y[k] = X[k] e^{i2\pi \frac{kS}{N}}
\end{split}
\end{equation}
$$

then the `fftshift`:

$$
\begin{equation}
\begin{split}
Z[k] &= Y[(k-S) \bmod N] \\
&= X[(k-S) \bmod N] e^{i2\pi \frac{S((k-S) \bmod N)}{N}}\\
&= e^{-i2\pi \frac{S^2}{N}} e^{i2\pi \frac{kS}{N}} X[(k-S) \bmod N]\\ 
&= e^{-i2\pi \frac{S^2}{N}} e^{i2\pi \frac{kS}{N}} \mathcal{F}(x[n] e^{i2\pi \frac{nS}{N}}) 
\end{split}
\end{equation}
$$

In other words, the combination `fftshift(fft(ifftshift(x)))` is equivalent to first applying the phase modulation $e^{i2\pi \frac{nS}{N}}$ to the original $x[n]$, then performing the standard fft, and finally appling the phase modulation $e^{i2\pi \frac{kS}{N}}$ to the output, along with a scalar factor $e^{-i2\pi \frac{S^2}{N}}$.

For higher-dimensional arrays, these operations can be performed independently along each dimension.

## fftshift-ifft-ifftshift
This combination of operations can be evaluated by first taking the inner `ifft(ifftshift(x))` part with the frequency-shifting property:

$$
\begin{equation}
\begin{split}
y[n] = x[n] e^{-i2\pi \frac{nS}{N}}
\end{split}
\end{equation}
$$

then the `ifftshift`:

$$
\begin{equation}
\begin{split}
z[n] &= y[(n-S) \bmod N] \\
&= x[(n-S) \bmod N] e^{-i2\pi \frac{S((n-S) \bmod N)}{N}}\\
&= e^{i2\pi \frac{S^2}{N}} e^{-i2\pi \frac{nS}{N}} x[(n-S) \bmod N]\\ 
&= e^{i2\pi \frac{S^2}{N}} e^{-i2\pi \frac{nS}{N}} \mathcal{F}^{-1}(X[k] e^{-i2\pi \frac{kS}{N}}) 
\end{split}
\end{equation}
$$

In other words, the combination `fftshift(ifft(ifftshift(x)))` is equivalent to first applying the phase modulation $e^{-i2\pi \frac{kS}{N}}$ to the original $X[k]$, then performing the standard ifft, and finally appling the phase modulation $e^{-i2\pi \frac{nS}{N}}$ to the output, along with a scalar factor $e^{i2\pi \frac{S^2}{N}}$.

For higher-dimensional arrays, these operations can be performed independently along each dimension.



