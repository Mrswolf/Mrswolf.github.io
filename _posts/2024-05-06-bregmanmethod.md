---
title: Powerful Bregman Methods
categories: [machine learning]
tags: [machine learning]
permalink: /bregmanmethod/
date: 2024-05-06 00:00:00
updated: 2024-05-10 00:00:00
toc: true
---


The Bregman method is an iterative method to solve convex optimization problems with equality constraints. <!--more-->The linearized Bregman method approximates subproblems in Bregman iterations with a linearized version. The split Bregman method decouples variables in the original problem to make subproblems in Bregman iterations much easier to solve, utilizing both the advantages of alternating minimization and Bregman method. In this post, I'll use the [denominator layout]({% post_url 2022-03-28-matrix-calculus %}) for gradients.

## What is Bregman Divergence
The Bregman divergence (or Bregman distance) of a convex function $F(\mathbf{x})$ at point $\mathbf{y}$ is defined as the difference between itself and its 1st-order Talyor expansion:

$$
\begin{equation}
\begin{split}
D_F^{\mathbf{p}}(\mathbf{x}, \mathbf{y}) = F(\mathbf{x}) - F(\mathbf{y}) - \mathbf{p}^T(\mathbf{x} - \mathbf{y})
\end{split}
\end{equation}
$$

where $\mathbf{p} \in \partial F(\mathbf{y})$ is a subgradient.

Here I list some of its properties and more can be found on [wiki](https://en.wikipedia.org/wiki/Bregman_divergence):
- $D_F^{\mathbf{p}}(\mathbf{x}, \mathbf{y}) \ge 0$
- $D_F^{\mathbf{p}}(\mathbf{y}, \mathbf{y}) = 0$
- $D_F^{\mathbf{p}}(\mathbf{x}, \mathbf{y}) + D_F^{\mathbf{q}}(\mathbf{y}, \mathbf{z}) - D_F^{\mathbf{q}}(\mathbf{x}, \mathbf{z}) = (\mathbf{p}-\mathbf{q})^T(\mathbf{y}-\mathbf{x})$

## Bregman Method
Consider the following constrained problem:

$$
\DeclareMathOperator*{\argmin}{arg\,min}
\begin{equation}
\begin{split}
\argmin_{\mathbf{x}}\ &F(\mathbf{x}) \\
&s.t. G(\mathbf{x}) = 0
\end{split}
\label{eq:2}
\end{equation}
$$

where $F(\mathbf{x})$ and $G(\mathbf{x})$ are convex, $G(\mathbf{x})$ is differentiable.

An intuitive way to solve the above problem is to transform it into an unconstrained form:

$$
\begin{equation}
\begin{split}
\argmin_{\mathbf{x}}\ F(\mathbf{x}) + \lambda G(\mathbf{x})
\end{split}
\label{eq:3}
\end{equation}
$$

where $\lambda \rightarrow \infty$ to enforce that $G(\mathbf{x}) \approx 0$. However, for many problems, a large $\lambda$ would make the problem numerically unstable. It's also difficult to determine how "large" is large for $\lambda$.

The Bregman method turns Equation (\ref{eq:2}) into a sequence of unconstrained problems. **Rather than choosing a large $\lambda$, the Bregman method approaches the equality with arbitrary precision by solving subproblems iteratively, with a fixed but smaller $\lambda$**.

Instead of solving Equation (\ref{eq:3}), the Bregman method solves the following problem:

$$
\begin{equation}
\begin{split}
\mathbf{x}_{k+1} &= \argmin_{\mathbf{x}}\ D_F^{\mathbf{p}_k}(\mathbf{x}, \mathbf{x}_k) + \lambda G(\mathbf{x})\\
&= \argmin_{\mathbf{x}}\ F(\mathbf{x}) - \mathbf{p}_k^T\mathbf{x} + \lambda G(\mathbf{x})
\end{split}
\end{equation}
$$

By the subgradient optimality condition, we know that:

$$
\begin{equation}
\begin{split}
0 \in  \partial F(\mathbf{x}_{k+1})  - \mathbf{p}_k + \lambda \nabla G(\mathbf{x}_{k+1})\\
\end{split}
\end{equation}
$$

<p>
Let $\mathbf{p}_{k+1} \in \partial F(\mathbf{x}_{k+1})$, the Bregman iteration is going to like:
</p>

$$
\begin{equation}
\begin{split}
\mathbf{x}_{k+1} &= \argmin_{\mathbf{x}}\ F(\mathbf{x}) - \mathbf{p}_k^T\mathbf{x} + \lambda G(\mathbf{x})\\
\mathbf{p}_{k+1} &= \mathbf{p}_k - \lambda \nabla G(\mathbf{x}_{k+1})
\end{split}
\label{eq:6}
\end{equation}
$$

<p>
where $\mathbf{x}_0 = \argmin_{\mathbf{x}} G(\mathbf{x})$ and $\mathbf{p}_k = 0$.
</p>

It can prove that as $k \rightarrow \infty$, $G(\mathbf{x}_k) \rightarrow 0$, thus the minimization problem in Equation (6) approximates the original problem in Equation (\ref{eq:2}).

<p>
If $G(\mathbf{x}_k) = \frac{1}{2}\|\mathbf{A}\mathbf{x} -\mathbf{b} \|_2^2$, then $\nabla G(\mathbf{x}_{k+1}) = \mathbf{A}^T\left(\mathbf{A}\mathbf{x}_{k+1} - \mathbf{b}\right)$. Equation (6) can be transformed into a simpler form:
</p>

$$
\begin{equation}
\begin{split}
\mathbf{x}_{k+1} &= \argmin_{\mathbf{x}}\ F(\mathbf{x}) + \frac{\lambda}{2} \|\mathbf{A}\mathbf{x} -\mathbf{b}_k \|_2^2\\
\mathbf{b}_{k+1} &= \mathbf{b}_k + \mathbf{b} - \mathbf{A}\mathbf{x}_k
\end{split}
\label{eq:7}
\end{equation}
$$

<p>
which simply adds the error in the constraint back to the right-hand side. Equation (\ref{eq:7}) can be proved by merging $\mathbf{p}_k^T\mathbf{x}$ into $G(\mathbf{x})$ and substituting $\mathbf{p}_k$ with its explict form $\mathbf{p}_k = \mathbf{p}_0 - \lambda \sum_{i=1}^{k} \nabla G(\mathbf{x}_i)$.
</p>

## Linearized Bregman Method
The Bregman method doesn't reduce the complexity of solving Equation (\ref{eq:3}), especially when $F(\mathbf{x})$ is not differentiable (e.g. $l_1$ norm) and not separable (its elements are coupled with each other). **Suppose that $F(\mathbf{x})$ is separable, it would be easier to solve the problem if we could separate elements in $G(\mathbf{x})$ either.**

<p>
That is what the linearized Bregman method does. It linearizes $G(\mathbf{x})$ with its 1st-order Talyor expansion at $\mathbf{x}_k$:
</p>

$$
\begin{equation}
\begin{split}
\mathbf{x}_{k+1} &= \argmin_{\mathbf{x}}\ D_F^{\mathbf{p}_k}(\mathbf{x}, \mathbf{x}_k)\\ &+ \lambda G(\mathbf{x}_k) + \lambda\nabla G(\mathbf{x}_k)^T(\mathbf{x} - \mathbf{x}_k) + \frac{\lambda\mu}{2} \|\mathbf{x} -\mathbf{x}_k\|_2^2\\
&= \argmin_{\mathbf{x}}\ D_F^{\mathbf{p}_k}(\mathbf{x}, \mathbf{x}_k) \\&+ \frac{\lambda\mu}{2} \|\mathbf{x} -\left(\mathbf{x}_k - \frac{1}{\mu} \nabla G(\mathbf{x}_k)  \right)\|_2^2\\
\end{split}
\label{eq:8}
\end{equation}
$$

where $\frac{\mu}{2} \|\mathbf{x} -\mathbf{x}\_k\|\_2^2$ is a penalty term since this approximation only works when $\mathbf{x}$ nears $\mathbf{x}\_k$. Note that $\nabla G(\mathbf{x}\_k)$ is merged into the $l_2$ norm as the same Equation (6) in post [RPCA]({% post_url 2023-09-12-rpca %}). Let the $l\_2$ norm term as a new function, then we can derive $\mathbf{p}\_{k+1}$ by Equation (\ref{eq:6}):

$$
\begin{equation}
\begin{split}
\mathbf{p}_{k+1} = \mathbf{p}_{k} -\lambda\mu (\mathbf{x} - \mathbf{x}_k + \frac{1}{\mu} \nabla G(\mathbf{x}_k))
\end{split}
\end{equation}
$$

Equation (\ref{eq:8}) is much easier to solve since all elements of $\mathbf{x}$ are separable.


## Split Bregman Method
The split Bregman method is used when $F(\mathbf{x})$ is not obviously separable. **The key idea of the split Bregman is to decouple $l_1$ and $l_2$ terms by equality constraints.**

Consider the following optimization problem:

$$
\begin{equation}
\begin{split}
\argmin_{\mathbf{x}}\ \|F(\mathbf{x})\|_1 + \lambda G(\mathbf{x})
\end{split}
\end{equation}
$$

where $F(\mathbf{x})$ and $G(\mathbf{x})$ are both convex and differentiable. Let $\mathbf{d} = F(\mathbf{x})$, then we transform the original problem into a constrained problem:

$$
\begin{equation}
\begin{split}
\argmin_{\mathbf{x}, \mathbf{d}}\ &\|\mathbf{d}\|_1 + \lambda G(\mathbf{x})\\
&s.t.\ F(\mathbf{x}) - \mathbf{d} = 0
\end{split}
\end{equation}
$$

which is a joint optimization over $\mathbf{x}$ and $\mathbf{b}$. This can be further transformed into an unconstrained problem with a penalty as the Bregman method did:

$$
\begin{equation}
\begin{split}
\argmin_{\mathbf{x}, \mathbf{d}}\ &\|\mathbf{d}\|_1 + \lambda G(\mathbf{x}) + \frac{\mu}{2}\|F(\mathbf{x}) - \mathbf{d}\|_2^2\\
\end{split}
\end{equation}
$$

Let $H(\mathbf{x}, \mathbf{d}) = \|\mathbf{d}\|_1 + \lambda G(\mathbf{x})$ and $A(\mathbf{x}, \mathbf{d}) = F(\mathbf{x}) - \mathbf{d}$ (both separable for $\mathbf{x}$ and $\mathbf{d}$), by Equation (\ref{eq:6}), we have:

$$
\begin{equation}
\begin{split}
\mathbf{x}_{k+1},\mathbf{d}_{k+1} &= \argmin_{\mathbf{x}, \mathbf{d}}\ H(\mathbf{x}, \mathbf{d}) - \mathbf{p}_{\mathbf{x},k}^T\mathbf{x} - \mathbf{p}_{\mathbf{d},k}^T\mathbf{d} \\&+ \frac{\mu}{2}\|A(\mathbf{x}, \mathbf{d})\|_2^2\\
&= \argmin_{\mathbf{x}, \mathbf{d}}\ H(\mathbf{x}, \mathbf{d}) - \mathbf{p}_{\mathbf{x},k}^T\mathbf{x} - \mathbf{p}_{\mathbf{d},k}^T\mathbf{d} \\&+ \frac{\mu}{2}\|F(\mathbf{x}) - \mathbf{d}\|_2^2\\
\mathbf{p}_{\mathbf{x},k+1} &= \mathbf{p}_{\mathbf{x},k} - \mu \left(\nabla_{\mathbf{x}} A(\mathbf{x}_{k+1}, \mathbf{d}_{k+1})\right)^T A(\mathbf{x}_{k+1}, \mathbf{d}_{k+1})\\
&= \mathbf{p}_{\mathbf{x},k} - \mu \left(\nabla F(\mathbf{x}_{k+1})\right)^T(F(\mathbf{x}_{k+1}) - \mathbf{d}_{k+1})\\
\mathbf{p}_{\mathbf{d},k+1} &= \mathbf{p}_{\mathbf{d},k} - \mu \left(\nabla_{\mathbf{d}} A(\mathbf{x}_{k+1}, \mathbf{d}_{k+1})\right)^T A(\mathbf{x}_{k+1}, \mathbf{d}_{k+1})\\
&= \mathbf{p}_{\mathbf{d},k} - \mu (\mathbf{d}_{k+1}- F(\mathbf{x}_{k+1}))\\
\end{split}
\end{equation}
$$

If $F(\mathbf{x})$ is linear, then the above iteration can be simplified as that in Equation (\ref{eq:7}):

$$
\begin{equation}
\begin{split}
\mathbf{x}_{k+1},\mathbf{d}_{k+1} &= \argmin_{\mathbf{x}, \mathbf{d}}\ H(\mathbf{x}, \mathbf{d}) + \frac{\mu}{2}\|F(\mathbf{x}) - \mathbf{d} - \mathbf{b}_k \|_2^2\\
\mathbf{b}_{k+1} &= \mathbf{b}_k - F(\mathbf{x}_{k+1}) + \mathbf{d}_{k+1}
\end{split}
\label{eq:14}
\end{equation}
$$

Equation (\ref{eq:14}) is still a joint minimizaiton problem, which can be solved by ([alternating minimization](https://web.eecs.umich.edu/~fessler/course/598/l/n-06-alt.pdf) or [coordinate descent](https://www.stat.cmu.edu/~ryantibs/convexopt-F18/lectures/coord-desc.pdf)) minimizing with respect to $\mathbf{x}$ and $\mathbf{d}$, separately:
$$
\begin{equation}
\begin{split}
\mathbf{x}_{k+1} &= \argmin_{\mathbf{x}}\ \lambda G(\mathbf{x}) + \frac{\mu}{2}\|F(\mathbf{x}) - \mathbf{d}_k - \mathbf{b}_k \|_2^2\\
\mathbf{d}_{k+1} &= \argmin_{\mathbf{d}}\ \|\mathbf{d}\|_1 + \frac{\mu}{2}\|F(\mathbf{x}_{k+1}) - \mathbf{d} - \mathbf{b}_k \|_2^2\\
\mathbf{b}_{k+1} &= \mathbf{b}_k - F(\mathbf{x}_{k+1}) + \mathbf{d}_{k+1}
\end{split}
\end{equation}
$$

The first subproblem can be solved by setting the gradient to zero. The second subproblem has an explicit solution with soft-thresholding operator in [this post]({% post_url 2023-09-12-rpca %}). Note that these two subproblem are just one iteration of the alternating minimization method. To achieve the same convergence rate of Equation ($\ref{eq:14}$), it requires more iterations which are called inner loops (in constrast, the Bregman iterations are called outer loops). For most applications, one inner loop is enough.



