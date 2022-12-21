---
layout: default
title: SDPA-SeDuMi Interconversion
parent: SDP Formats
nav_order: 2
---

# Interconversion between SDPA and SeDuMi Formats

As discussed on the [SDPA Formats](index.html) page, the SDPA and SeDuMi formats are reverses of each other. 
In SDPA, the primal problem is a linear-matrix-inequality form while in SeDuMi, the dual problem is a linear-matrix-inequality form.

It's also discussed that SDPA has the ability to solve the problem in either format. To this end, SDPA comes with a preprocessor directive `REVERSE_PRIMAL_DUAL`.

- With `REVERSE_PRIMAL_DUAL = 1`, we use the SDPA form. This is the default setting (i.e. SDPA is configured to treat the primal problem as a linear-matrix-inequality form).
- With `REVERSE_PRIMAL_DUAL = 0`, we use the SeDuMi form.

Below we show how setting `REVERSE_PRIMAL_DUAL = 0` gives us the SeDuMi format. We start off by writing the primal-dual pair that SDPA solves with the default setting (i.e. with `REVERSE_PRIMAL_DUAL = 1`).

$$
\begin{aligned}
(\mathcal{P}) \min_{x}. \quad & c^T x\\
\textrm{s.t.} \quad & \sum_{i=1}^m F_i x_i - F_0 \succeq_{\mathbb{S}^n_+} 0
\end{aligned}
$$

$$
\begin{aligned}
(\mathcal{D}) \max_{Y}. \quad & F_0 \cdot Y\\
\textrm{s.t.} \quad & c_i - F_i \cdot Y = 0, \quad i = 1, ..., m \\
 & Y \succeq_{\mathbb{S}_+} 0
\end{aligned}
$$

The primal variable (i.e. vector $$x$$) with `REVERSE_PRIMAL_DUAL = 1` should become the dual variable with `REVERSE_PRIMAL_DUAL = 0` and the dual variable (i.e. the matrix $$Y$$) should become the primal variable. To this end, we will make the below substitutions.

$$
\begin{aligned}
x &\longrightarrow y\\
Y &\longrightarrow X\\
\end{aligned}
$$

To achieve similarity with SeDuMi, we will also perform the below substitutions.

$$
\begin{aligned}
F_i &\longrightarrow -A_i\\
c &\longrightarrow -b\\
\end{aligned}
$$

After these substitutions, we get the pair

$$
\begin{aligned}
(\mathcal{?}) \min_{y}. \quad & -b^T y\\
\textrm{s.t.} \quad & \sum_{i=1}^m -A_i y_i + A_0 \succeq_{\mathbb{S}^n_+} 0
\end{aligned}
$$

$$
\begin{aligned}
(\mathcal{?}) \max_{X}. \quad & -A_0 \cdot X\\
\textrm{s.t.} \quad & -b_i + A_i \cdot X = 0, \quad i = 1, ..., m \\
 & X \succeq_{\mathbb{S}_+} 0
\end{aligned}
$$

We can now flip the $$\min$$ and $$\max$$ and remove the minus signs from the objectives. Since by convention the primal problem is a minimization problem and the dual is a maximization problem, we also exchange their roles as primal and dual and create the pair corresponding to `REVERSE_PRIMAL_DUAL = 0` as below:

$$
\begin{aligned}
(\mathcal{P'}) \min_{X}. \quad & A_0 \cdot X\\
\textrm{s.t.} \quad & A_i \cdot X = b_i, \quad i = 1, ..., m \\
 & X \succeq_{\mathbb{S}_+} 0
\end{aligned}
$$

$$
\begin{aligned}
(\mathcal{D'}) \max_{y}. \quad & b^T y\\
\textrm{s.t.} \quad & A_0 - \sum_{i=1}^m A_i y_i \succeq_{\mathbb{S}^n_+} 0
\end{aligned}
$$

In general, conversion from matrices to vectors and vice versa can be obtained very easily using the $$\text{vec}(...)$$ and $$\text{vec}^{-1}(...)$$ operations. Knowing that $$\text{vec}(X)$$ in $$(\mathcal{P'})$$ above becomes $$x$$ in $$(\mathcal{P''})$$ below, and the fact that SDP cone is self dual (i.e. $$\mathbb{S}^n_+ = {\mathbb{S}^n_+}^*$$) we now see the similarity with [the SeDuMi format](index.html#sedumi-format):

$$
\begin{aligned}
(\mathcal{P''}) \min_{x}. \quad & c^T x\\
\textrm{s.t.} \quad & A x - b = 0\\
 & x \succeq_K 0
\end{aligned}
$$

$$
\begin{aligned}
(\mathcal{D''}) \max_{y}. \quad & b^T y\\
\textrm{s.t.} \quad & c - A^T y \succeq_{K^*} 0
\end{aligned}
$$

Another difference is that the cone $$K$$ supported by SeDuMi is a generalization of the SDP cone, in sense that it also supports SOCP constraints.