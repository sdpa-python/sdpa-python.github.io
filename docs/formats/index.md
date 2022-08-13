---
layout: default
title: SDP Formats
has_children: true
nav_order: 3
---

# Storing and Representing SDPs

We briefly discuss the file formats for storing SDP problems. Most SDP solvers were developed in the late 1990s, which is when these formats were specified.

{: .attention }
This section is an optional read if you want to use SDPA for Python. For quick start, please see the [Usage](https://sdpa-python.github.io/docs/usage/) section.

{: .warning }
*SDPA for Python* solves an problem in a different standard form than *SDPA*. Below, when we say SDPA, we mean the backend software used by SDPA for Python. SDPA for Python, can however read problems stored in SDPA sparse format.

There happen to be two widely used formats: one is the **SDPA format** which was introduced by the developers of SDPA itself. The other is the **SeDuMi format** which was introduced by the developer of SeDuMi.

A third format (**CLP**) will be discussed which is a generalization of the SeDuMi format and is used by SDPA for Python (fork of SDPAP).

The primal-dual pairs of problems represented in SDPA and SeDuMi formats are reverse of each other. In SDPA, the primal problem is in a linear-matrix-inequality form. In SeDuMi, the dual problem is in a linear-matrix-inequality form.

Internally, SDPA has the capability to solve the problem in either SeDuMi or SDPA format, depending on how it's compiled.

The SDPA format also includes a specification of the contents of a file on a disk to store the problem data. The SeDuMi format does not have such a specification, but the problem variables (in any programming language) can be stored to disk. Datasets of problems exist in both formats.

SDPA and SeDuMi formats have been used (either for storage or internally) by many other solvers as well, including SDPT3, Csdp, Dsdp, etc.

# SDPA Format

SDPA solves the following standard form semidefinite program and its dual.

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

where

- $$x$$ is the primal variable (vector),
- $$c$$ is the cost vector and
- both $$x, c \in \mathbb{R}^m$$.
- $$F_i$$ are the constraint matrices,
- $$Y$$ is the dual variable (matrix) and 
- all $$F_i, Y \in {\mathbb{S}^n}$$

The SDPA format is also specified as the contents of a text file which can store a semidefinite program in the form represented above. Please see the subsection [SDPA Format](sdpa.html) for details. The file extension used with the regular SDPA format is `.dat`. The file extension used with the SDPA sparse format is `.dat-s`.

A commonly used dataset for problems in this format is [SDPLIB](http://euler.nmt.edu/~brian/sdplib/sdplib.html) ([View as a GitHub repo](https://github.com/vsdp/SDPLIB)). This dataset is by Brian Borchers, the founder of [Csdp](https://github.com/coin-or/csdp/) solver.

The primal problem in SDPA is a linear-matrix-inequality form.

# SeDuMi Format

The SeDuMi solver solves the following primal ($$\mathcal{P}$$) and dual ($$\mathcal{D}$$) problems:

$$
\begin{aligned}
(\mathcal{P}) \min_{x}. \quad & c^T x\\
\textrm{s.t.} \quad & x \succeq_K 0\\
 & A x - b = 0
\end{aligned}
$$

$$
\begin{aligned}
(\mathcal{D}) \max_{y}. \quad & b^T y\\
\textrm{s.t.} \quad & c - A^T y \succeq_{K^*} 0
\end{aligned}
$$

where

- $$K$$ is a self-dual homogeneous cone that you describe in the structure `K` and $$K^*$$ is its dual cone.
- Matrix $$A \in \mathbb{R}^{m \times n}$$
- Vectors $$x, c \in \mathbb{R}^n$$
- Vectors $$y, b \in \mathbb{R}^m$$

Unlike the SDPA format, the SeDuMi format does not have a specification as the contents of a text file, but rather as a set of variables storing the problem data in a program workspace. It comprises of the variables `A`, `b`, `c`, `K` which define the primal ($$\mathcal{P}$$) and dual ($$\mathcal{D}$$) problems as shown above.

A commonly used dataset for problems in SeDuMi format is the [DIMACS](http://archive.dimacs.rutgers.edu/Challenges/Seventh/Instances/) ([View as a GitHub repo](https://github.com/vsdp/DIMACS)). The problems in this dataset have the extension `.mat` which can be natively read into MATLAB and (using some libraries) into Python as well.

The primal-dual pair in SeDuMi is the reverse of that in SDPA. In SeDuMi, the dual problem is a linear-matrix-inequality form.

# CLP Format

SDPAP (of which this wrapper is a fork) introduced a third format, called CLP that stands for Conic-form Linear Optimization Problems. The official specification of this format is given in the [user manual of SDPAP](https://sourceforge.net/projects/sdpa/files/sdpa-p/sdpap_manual.pdf) on the [official SDPA website](http://sdpa.sourceforge.net/download.html).

The CLP format is defined both as a file format, as well as an in memory representation which is basically a generalization the SeDuMi format. In addition to `K`, another cone `J` must as well be specified in the CLP format.

$$
\begin{aligned}
(\mathcal{P}) \min_{x}. \quad & c^T x\\
\textrm{s.t.} \quad & x \succeq_K 0\\
 & A x - b \succeq_J 0
\end{aligned}
$$

$$
\begin{aligned}
(\mathcal{D}) \max_{y}. \quad & b^T y\\
\textrm{s.t.} \quad & c - A^T y \succeq_{K^*} 0\\
 & y \succeq_{J^*} 0
\end{aligned}
$$

The SeDuMi form is the special case of the above in which the cone `J` is the zero cone (and it's dual $$J^*$$ is the Euclidean space). The constraint $$Ax-b \succeq_J 0$$ then becomes $$Ax-b=0$$ and $$y$$ becomes a free variable, making $$y \succeq_{J^*} 0$$ a redundant constraint.

Internally, this Python wrapper will check if `J` is the zero cone. If it's not, it will transform the other variables such that `J` becomes the zero cone. Internally, SDPA always solves the problem in SeDuMi format.

After reading the problem data from a CLP file, and before calling the SDPA solver code, this wrapper will convert the CLP format into SeDuMi dual ($$\mathcal{D}$$) problem format using a function named `clp_toLMI`. This conversion involves getting rid of `J` to comply with the SeDuMi format (the structures `A`, `b`, `c` and `K` are adjusted accordingly). 

An unimplemented function `clp_toEQ` is present in the code which is meant to convert CLP format to SeDuMi primal ($$\mathcal{P}$$) problem format. It's not yet implemented, but we hope to fix that soon.
