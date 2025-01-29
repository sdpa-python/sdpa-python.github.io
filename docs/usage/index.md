---
layout: default
title: Usage
has_children: true
nav_order: 4
---

# A Gentle Start

SDPA for Python solves Conic-form Linear Optimization Problems (CLPs). A CLP is of the form:

$$
\begin{aligned}
(\mathcal{P}) \min_{x}. \quad & c^T x\\
\textrm{s.t.} \quad & x \succeq_K 0\\
 & A x - b \succeq_J 0,
\end{aligned}
$$

where $$\mathcal{ J }$$ and $$\mathcal{ K }$$ represents cones, that will be defined in the next subsection.

As an example, we will solve the following specific CLP in this section as a gentle begin, the problems is defined as $$(\mathcal{P})$$ with 

$$
\begin{aligned}
&\boldsymbol{A}=\left(\begin{array}{cccc}
-10 & -4 & -4 & 0 \\
0 & 0 & 0 & 8 \\
0 & 8 & 8 & 2
\end{array}\right), \quad \boldsymbol{b}=\left(\begin{array}{c}
-48 \\
8 \\
-20
\end{array}\right), \quad \boldsymbol{c}=\left(\begin{array}{c}
11 \\
0 \\
0 \\
-23
\end{array}\right) \\
&\text { J.f }=3, \quad \mathrm{~K} . \mathrm{s}=2,
\end{aligned}
$$

where `J.f` and `K.s` represents the data format of the corresponding cones $$\mathcal{ J }$$ and $$\mathcal{ K }$$.


## Data Format in SDPAP

As seen above, the problem data is read into Python variables `A`, `b`, `c`, `K` and `J`.

`A`, `b` and `c` may be defined as `numpy.ndarray` or one of the sparse array formats (e.g. `csr_array` or `csc_array`) in `scipy.sparse`.

`K` and `J` are `SymCone` objects, a data structure provided in `sdpa-python`. They can be initialized using `K = sdpap.SymCone()` and `J = sdpap.SymCone()` respectively. We need to set their parameters according to the problem.

In the CLP problem form shown above, the generalized inequality $$x \succeq_{\mathcal{ K }} 0$$ means $$x\in \mathcal{ K }$$ for cone $$\mathcal{ K}$$. The cones $$\mathcal{ J }$$ and $$\mathcal{ K }$$ are each defined as Cartesian product of three different type of cones:


$$\mathcal{ K } = \mathcal{ K }_{f}\times\mathcal{ K }_{l}\times\mathcal{ K }_{s}$$ and $$\mathcal{ J } = \mathcal{ J }_{f}\times\mathcal{ J }_{l}\times\mathcal{ J }_{s}$$



Corresponding to each type, we will specify for cone $$\mathcal{ K }$$, the parameters `K.f`, `K.l`, `K.s` and similarly for cone $$\mathcal{ J }$$, the parameters `J.f`, `J.l`, `J.s`

These parameters specify the type of constraints, which will be discussed in the following.


## $$f \colon$$ Equality Constraints
​
The cone with subscript $$f$$ represents $$x$$ that satisfies the equality constraint. In code, we use `J.f` or `K.f` to represent the number of equality constraint. For example, for

$$
\left(\begin{array}{cccc}
-10 & -4 & -4 & 0 \\
0 & 0 & 0 & 8 \\
0 & 8 & 8 & 2
\end{array}\right) x-\left(\begin{array}{c}
-48 \\
8 \\
-20
\end{array}\right)=0,
$$

We have three equality constraints. Hence, `J.f = 3`.
​
## $$l \colon$$ LP Cones

The cone with subscript $$l$$ represents the LP cone, i.e., $$x$$ such that $$\left\{x\colon x \succeq_{\mathbb{ R }_{+}^{n}} 0\right\}$$ In code, we use `J.l` or `K.l` to represent the size of variables in the LP cone constraints.

For example, for

$$
\left(\begin{array}{cccc}
-10 & -4 & -4 & 0 \\
0 & 0 & 0 & 8 \\
0 & 8 & 8 & 2
\end{array}\right) x-\left(\begin{array}{c}
-48 \\
8 \\
-20
\end{array}\right)\succeq_{\mathbb{ R }_{+}^{n}} 0,
$$

We have three inequality constraints. Thus, `J.l = 3`.
​
## $$s\colon$$ SDP Cones

The cone with subscript $$s$$ represents the SDP cones. One SDP cone is defined as the set

$$
\left\{\boldsymbol{x}_{s}: \text{vec}^{-1}(\boldsymbol{x}_{s})\succeq_{S_{+}} \mathbf{0}\right\}
$$

where $$S_{+}$$ represents positive-semidefinite cone. The size of $$\boldsymbol{x}_{s}$$ must be a square of an integer, i.e., $$\boldsymbol{x}_{s} \in \mathbb{ R }^{n^2}$$ and $$\text{vec}^{-1}(\boldsymbol{x}_{s}) \in  \mathbb{ R }^{n\times n}$$
​
The symbol $$\text{vec}^{-1}(x)$$ means the matrix formed by the vector $$x$$. That is, if

$$
\boldsymbol{x}_{s}=\left[\begin{array}{l}
x_{1} \\
x_{2} \\
x_{3} \\
x_{4}
\end{array}\right]
$$

then

$$
\text{vec}^{-1}\left(\boldsymbol{x}_{s}\right)=\left(\begin{array}{ll}
x_{1} & x_{3} \\
x_{2} & x_{4}
\end{array}\right)
$$


In code, we use `J.s` or `K.s` as a tuple to represents the number of SDP cone constraints and the number of the rows or columns of the matrix in each SDP cone. For example, if we have three SDP cone constraints $$\boldsymbol{x}_{1}\succeq_{S_{+}} 0,\boldsymbol{x}_{2}\succeq_{S_{+}} 0$$ and $$\boldsymbol{x}_{3} \succeq_{S_{+}} 0$$ and $$\boldsymbol{x}_{i}\in \mathbb{ R }^{i^2}$$ for any $$i = 1,2,3$$. Then `K.s = (1,2,3)`.

## Solving the Sample CLP

Now, we can define and solve the sample CLP given in the beginning of this section by the following code:

```python
import sdpap
import numpy as np

A = np.array([
    [-10, -4, -4, 0],
    [0, 0, 0, 8],
    [0, 8, 8, 2]
])

b = np.array([
    [-48],
    [8],
    [-20]
])

c = np.array([
    [11],
    [0],
    [0],
    [-23]
])

K = sdpap.SymCone(s=(2,))
J = sdpap.SymCone(f=3)

x, y, sdpapinfo , timeinfo , sdpainfo = sdpap.solve(A,b,c,K,J)
```

The output will be as below:

```
---------- SDPAP Start ----------
SDPA start at [Sun Jan 26 20:41:26 2025]
NumThreads  is set as 12
Schur computation : DENSE 
Converted to SDPA internal data / Starting SDPA main loop
   mu      thetaP  thetaD  objP      objD      alphaP  alphaD  beta 
 0 1.0e+04 1.0e+00 1.0e+00 -0.00e+00 +1.20e+03 1.0e+00 9.1e-01 2.00e-01
 1 1.6e+03 0.0e+00 9.4e-02 +8.39e+02 +7.51e+01 2.3e+00 9.6e-01 2.00e-01
 2 1.7e+02 2.3e-16 3.6e-03 +1.96e+02 -3.74e+01 1.3e+00 1.0e+00 2.00e-01
 3 1.8e+01 2.9e-16 2.2e-17 -6.84e+00 -4.19e+01 9.9e-01 9.0e+01 1.00e-01
 4 1.9e+00 2.6e-16 1.8e-15 -3.81e+01 -4.19e+01 1.0e+00 1.0e+00 1.00e-01
 5 1.9e-01 2.7e-16 7.5e-18 -4.15e+01 -4.19e+01 1.0e+00 1.0e+00 1.00e-01
 6 1.9e-02 2.8e-16 7.5e-18 -4.19e+01 -4.19e+01 1.0e+00 9.0e+01 1.00e-01
 7 1.9e-03 2.9e-16 1.2e-15 -4.19e+01 -4.19e+01 1.0e+00 1.0e+00 1.00e-01
 8 1.9e-04 2.8e-16 2.2e-17 -4.19e+01 -4.19e+01 1.0e+00 1.0e+00 1.00e-01
 9 1.9e-05 2.7e-16 7.5e-18 -4.19e+01 -4.19e+01 1.0e+00 1.0e+00 1.00e-01
10 1.9e-06 2.9e-16 3.7e-18 -4.19e+01 -4.19e+01 1.0e+00 1.0e+00 1.00e-01

phase.value  = pdOPT     
   Iteration = 10
          mu = +1.9180668442023158e-06
relative gap = +9.1554505917001858e-08
        gap  = +3.8361336223147191e-06
     digits  = +7.0383202767393023e+00
objValPrimal = -4.1899996163866383e+01
objValDual   = -4.1900000000000006e+01
p.feas.error = +3.5690251616723664e-14
d.feas.error = +3.5527136788005009e-15
total time   = 0.003271
  main loop time = 0.003268
      total time = 0.003271
file  check time = 0.000000
file change time = 0.000003
file   read time = 0.000000
Converting optimal solution to CLP format
SDPA end at [Sun Jan 26 20:41:26 2025]
Start: getCLPresult
Making result infomation...
(Re)calculating feasibility errors for CLP converted solution.
/opt/anaconda3/lib/python3.12/site-packages/sdpap/sdpaputils.py:114: RuntimeWarning: k >= N - 1 for N * N square matrix. Attempting to use scipy.linalg.eig instead.
  eig = eigs(mat.toarray(), k=1, which='SM',
========================================
 SDPAP: Result
========================================
    SDPA.phase = pdOPT
     iteration = 10
    convMethod = LMI
     frvMethod = split
  domainMethod = none
   rangeMethod = none
     primalObj = +4.1900000000000006e+01
       dualObj = +4.1899996163866383e+01
    dualityGap = +9.1554505917001858e-08
   primalError = +3.5527136788005009e-15
     dualError = +0.0000000000000000e+00
   convertTime = 0.000166
     solveTime = 0.011680
retrievingTime = 0.000002
     totalTime = 0.011902
---------- SDPAP End ----------
```

The dictionary `sdpainfo` will contain the output of the backend solver. The dictionary `sdpapinfo` will contain the output of SDPA for Python (for the original CLP).
