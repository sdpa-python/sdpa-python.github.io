---
layout: default
title: A Gentle Start
parent: Usage
nav_order: 2
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
10 & 4 & 4 & 0 \\
0 & 0 & 0 & -8 \\
0 & -8 & -8 & -2
\end{array}\right), \quad \boldsymbol{b}=\left(\begin{array}{c}
48 \\
-8 \\
20
\end{array}\right), \quad \boldsymbol{c}=\left(\begin{array}{c}
-11 \\
0 \\
0 \\
23
\end{array}\right) \\
&\text { J.f }=3, \quad \mathrm{~K} . \mathrm{s}=2,
\end{aligned}
$$

where `J.f` and `K.s` represents the data format of the corresponding cones $$\mathcal{ J }$$ and $$\mathcal{ K }$$.


## Data Format in SDPAP

As seen above, the problem data is read into Python variables `A`, `b`, `c`, `K` and `J`.

`A`, `b` and `c` may be defined as `numpy.matrix` or `scipy.matrix` objects.

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
10 & 4 & 4 & 0 \\
0 & 0 & 0 & -8 \\
0 & -8 & -8 & -2
\end{array}\right) x-\left(\begin{array}{c}
48 \\
-8 \\
20
\end{array}\right)=0,
$$

We have three equality constraints. Hence, `J.f = 3`.
​
## $$l \colon$$ LP Cones

The cone with subscript $$l$$ represents the LP cone, i.e., $$x$$ such that $$\left\{x\colon x \succeq_{\mathbb{ R }_{+}^{n}} 0\right\}$$ In code, we use `J.l` or `K.l` to represent the size of variables in the LP cone constraints.

For example, for

$$
\left(\begin{array}{cccc}
10 & 4 & 4 & 0 \\
0 & 0 & 0 & -8 \\
0 & -8 & -8 & -2
\end{array}\right) x-\left(\begin{array}{c}
48 \\
-8 \\
20
\end{array}\right)\succeq_{\mathbb{ R }_{+}^{n}} 0,
$$

We have three inequality constraints. Thus, `J.l = 3`.
​
## $$s\colon$$ SDP Cones

The cone with subscript $$s$$ represents the SDP cones. One SOCP cone is defined as the set

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
from scipy import matrix

A = matrix([
    [10, 4, 4, 0],
    [0, 0, 0, -8],
    [0, -8, -8, -2]    
])

b = np.matrix([
    [48],
    [-8],
    [20]
])
print("test")

c = matrix([
    [-11],
    [0],
    [0],
    [23]
])

K = sdpap.SymCone(s=(2,))
J = sdpap.SymCone(f=3)

x, y, sdpapinfo , timeinfo , sdpainfo = sdpap.solve(A,b,c,K,J)
```

The result will be the following:
```
---------- SDPAP Start ----------
Start: getCLPresult
Making result infomation...
========================================
 SDPAP: Result
========================================
    SDPA.phase = pdOPT
     iteration = 10
    convMethod = LMI
     frvMethod = split
  domainMethod = none
   rangeMethod = none
     primalObj = -4.1899999999999977e+01
       dualObj = -4.1900003836133791e+01
    dualityGap = +9.1554502113451059e-08
   primalError = +1.4210854715202004e-14
     dualError = +0.0000000000000000e+00
   convertTime = 0.000308
     solveTime = 0.015920
retrievingTime = 0.000016
     totalTime = 0.018300
---------- SDPAP End ----------
```
