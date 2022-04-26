---
layout: default
title: SDP Formats
has_children: true
nav_order: 3
---

# Storing and Representing SDPs

We briefly discuss the file formats for storing SDP problems. Most SDP solvers were developed in the late 1990s, which is when these formats were specified.

There happen to be two widely used formats: one is the **SDPA format** which was introduced by the developers of SDPA itself. The other is the **SeDuMi format** which was introduced by the developer of SeDuMi.

A third format (**CLP**) will be discussed which is used only by SDPAP and is closely related to SeDuMi format.

The two formats are defined at different levels. The SDPA format is specified as the format of a text file to store SDP problem data while the SeDuMi format is specified as a set of variables storing the problem data inside a program. SeDuMi format, can therefore be thought of as a specification in the memory rather than as a specification of the contents of a file on disk.

Interestingly, SDPA can read problems stored in SDPA format, but internally, it uses the SeDuMi representation. SDPAP can also read problems stored in CLP format.

SDPA and SeDuMi formats have been used (either for storage or internally) by many other solvers as well, including SDPT3, Csdp, Dsdp, etc.

# SDPA Format

Since it was introduced by the developers of SDPA, the official specification of this format is found in the user manual of SDPA on the [official SDPA website](http://sdpa.sourceforge.net/download.html).

A commonly used dataset for problems in this format is [SDPLIB](http://euler.nmt.edu/~brian/sdplib/sdplib.html) ([View as a GitHub repo](https://github.com/vsdp/SDPLIB)). This dataset is by Brian Borchers, the founder of [Csdp](https://github.com/coin-or/csdp/) solver.

It has two subtypes, regular **SDPA** and **SDPA Sparse** format. The regular SDPA format does not assume sparsity of the problem matrices. The sparse format should be used when we are dealing with mostly sparse matrices.

The file extension used with the regular SDPA format is `.dat`. The file extension used with the SDPA sparse format is `.dat-s`.

# SeDuMi Format

We discussed that unlike the SDPA format, the SeDuMi format is specified not as a text file, but rather as a set of variables storing the problem data in a workspace.

The SeDuMi format comprises of the variables `A`, `b`, `c`, `K` which define the primal ($$\mathcal{P}$$) and dual ($$\mathcal{D}$$) problems as below:

$$
\begin{aligned}
(\mathcal{P}) \min_{x} \quad & c^T x\\
\textrm{s.t.} \quad & x \succeq_K 0\\
 & A x - b = 0
\end{aligned}
$$

$$
\begin{aligned}
(\mathcal{D}) \max_{y} \quad & b^T y\\
\textrm{s.t.} \quad & c - A^T y \succeq_{K^*} 0
\end{aligned}
$$

where

- $$K$$ is a self-dual homogeneous cone that you describe in the structure `K` and $$K^*$$ is its dual cone.
- Matrix $$A \in \mathbb{R}^{m \times n}$$
- Vectors $$x, c \in \mathbb{R}^n$$
- Vectors $$y, b \in \mathbb{R}^m$$

A commonly used dataset for problems in SeDuMi format is the [DIMACS](http://archive.dimacs.rutgers.edu/Challenges/Seventh/Instances/) ([View as a GitHub repo](https://github.com/vsdp/DIMACS)). The problems in this dataset have the extension `.mat` which can be natively read into MATLAB.

However, the SeDuMi format does not have to be MATLAB. In fact, we mentioned earlier that SDPA internally uses the SeDuMi format while SDPA is written in C++ (and this wrapper is in Python). The only specification of the SeDuMi format is the set of structures `A`, `b`, `c` and `K`, regardless of whatever language we are using.

# CLP Format

SDPAP introduced a third format, called CLP that stands for Conic-form Linear Optimization Problems. The official specification of this format is given in the user manual of SDPAP on the [official SDPA website](http://sdpa.sourceforge.net/download.html).

The CLP format is defined both as a file format, as well as an in memory representation which is basically a generalization the SeDuMi format. In addition to `K`, another cone `J` must as well be specified in the CLP format.

$$
\begin{aligned}
(\mathcal{P}) \min_{x} \quad & c^T x\\
\textrm{s.t.} \quad & x \succeq_K 0\\
 & A x - b \succeq_J 0
\end{aligned}
$$

$$
\begin{aligned}
(\mathcal{D}) \max_{y} \quad & b^T y\\
\textrm{s.t.} \quad & c - A^T y \succeq_{K^*} 0\\
 & y \succeq_{J^*} 0
\end{aligned}
$$

The SeDuMi form is the special case of the above in which the cone `J` is the zero cone (and it's dual $$J^*$$ is the Euclidean space). The constraint $$Ax-b \succeq_J 0$$ then becomes $$Ax-b=0$$ and $$y$$ becomes a free variable, making $$y \succeq_{J^*} 0$$ a redundant constraint.

Internally, SDPAP will check if `J` is the zero cone. If it's not, it will transform the other variables such that `J` becomes the zero cone. Interally, SDPA always solves the problem in SeDuMi format.

After reading the problem data from a CLP file, and before calling the SDPA solver code, SDPAP will convert the CLP format into SeDuMi dual ($$\mathcal{D}$$) problem format using a function named `clp_toLMI`. This conversion involves getting rid of `J` to comply with the SeDuMi format (the structures `A`, `b`, `c` and `K` are adjusted accordingly). 

An unimplemented function `clp_toEQ` is present in SDPAP code which is meant to convert CLP format to SeDuMi primal ($$\mathcal{P}$$) problem format. It's not yet implemented, but we hope to fix that soon.
