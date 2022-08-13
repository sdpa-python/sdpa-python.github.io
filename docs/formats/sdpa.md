---
layout: default
title: SDPA Format
parent: SDP Formats
nav_order: 1
---

# The SDPA (and SDPA Sparse) format

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

## Example Problem 1

An example problem in SDPA format is as below.

$$
\begin{aligned}
(\mathcal{P}) \min_{x}. \quad & 48 x_1 - 8 x_2 + 20 x_3 \\
\textrm{s.t.} \quad &
\begin{bmatrix}
10 & 4 \\
4 & 0
\end{bmatrix}
x_1 +
\begin{bmatrix}
0 & 0 \\
0 & -8
\end{bmatrix}
x_2 +
\begin{bmatrix}
0 & -8 \\
-8 & 2
\end{bmatrix}
x_3 -
\begin{bmatrix}
-11 & 0 \\
0 & 23
\end{bmatrix}
\succeq_{\mathbb{S}^n_+} 0
\end{aligned}
$$

$$
\begin{aligned}
(\mathcal{D}) \max_{Y}. \quad & F_0 \cdot Y\\
\textrm{s.t.} \quad & 
48 - 
\begin{bmatrix}
10 & 4 \\
4 & 0
\end{bmatrix}
\cdot Y = 0\\

 & -8 -
\begin{bmatrix}
0 & 0 \\
0 & -8
\end{bmatrix}
\cdot Y = 0\\

 & 20 -
\begin{bmatrix}
0 & -8 \\
-8 & 2
\end{bmatrix}
\cdot Y = 0\\
 & Y \succeq_{\mathbb{S}_+} 0
\end{aligned}
$$

As we can see both $$x, c \in \mathbb{R}^m = \mathbb{R}^3$$ and $$F_i, Y \in {\mathbb{S}^n} = {\mathbb{S}^2}$$, so $$m=3$$ and $$n=2$$.

This problem can be stored in a SDPA formatted (`.dat`) file as below:

```json
"Example 1: mDim = 3, nBLOCK = 1, {2}"
   3  =  mDIM
   1  =  nBLOCK
   2  = bLOCKsTRUCT
{48, -8, 20}
{ {-11,  0}, { 0, 23} }
{ { 10,  4}, { 4,  0} }
{ {  0,  0}, { 0, -8} }
{ {  0, -8}, {-8, -2} }
```

If we are dealing with constraint matrices involving mostly zeros, we can store the same problem equivalently in the SDPA Sparse formatted (`.dat-s`) file as below:

```json
"Example 1: mDim = 3, nBLOCK = 1, {2}"
   3  =  mDIM
   1  =  nBLOCK
   2  = bLOCKsTRUCT
48, -8, 20
0 1 1 1 -11
0 1 2 2 23
1 1 1 1 10
1 1 1 2 4
2 1 2 2 -8
3 1 1 2 -8
3 1 2 2 -2
```

After the row `48, -8, 20` which shows the cost vector, every row has five numbers, which respectively represent

1. Constraint matrix number ($$i$$ in $$F_i$$);
2. block number (this problem has only one block, but blocks should get clear in the next example);
3. (within the block), the row number;
4. (within the block), the column number and;
5. value to store.

**Note**: Since we are dealing with symmetric matrices, the row and column are interchangeable.

This example had a **single block** for each constraint matrix $$F_i$$. In the next example, we look at matrices having multiple blocks.

## Example Problem 2

This example has **three blocks** for each constraint matrix $$F_i$$. For simplicity, we skip the dual problem here.

$$
\begin{aligned}
(\mathcal{P}) \min_{x}. \quad & 1.1 x_1 - 10 x_2 + 6.6 x_3 + 19 x_4 + 4.1 x_5 \\
\textrm{s.t.} \quad & \sum_{i=1}^5 F_i x_i - F_0 \succeq_{\mathbb{S}^n_+} 0
\end{aligned}
$$

where

$$
F_0 = 
\begin{bmatrix}
\color{blue}{-1.4} & \color{blue}{-3.2} & 0.0 & 0.0 & 0.0 & 0.0 & 0.0 \\
\color{blue}{-3.2} & \color{blue}{-28} & 0.0 & 0.0 & 0.0 & 0.0 & 0.0 \\
0.0 & 0.0 & \color{blue}{15} & \color{blue}{-12} & \color{blue}{2.1} & 0.0 & 0.0 \\
0.0 & 0.0 & \color{blue}{-12} & \color{blue}{16} & \color{blue}{-3.8} & 0.0 & 0.0 \\
0.0 & 0.0 & \color{blue}{2.1} & \color{blue}{-3.8} & \color{blue}{15} & 0.0 & 0.0 \\
0.0 & 0.0 & 0.0 & 0.0 & 0.0 & \color{blue}{1.8} & \color{blue}{0.0} \\
0.0 & 0.0 & 0.0 & 0.0 & 0.0 & \color{blue}{0.0} & \color{blue}{-4.0}
\end{bmatrix}
$$

$$
F_1 = 
\begin{bmatrix}
\color{blue}{0.5} & \color{blue}{5.2} & 0.0 & 0.0 & 0.0 & 0.0 & 0.0 \\
\color{blue}{5.2} & \color{blue}{-5.3} & 0.0 & 0.0 & 0.0 & 0.0 & 0.0 \\
0.0 & 0.0 & \color{blue}{7.8} & \color{blue}{-2.4} & \color{blue}{6.0} & 0.0 & 0.0 \\
0.0 & 0.0 & \color{blue}{-2.4} & \color{blue}{4.2} & \color{blue}{6.5} & 0.0 & 0.0 \\
0.0 & 0.0 & \color{blue}{6.0} & \color{blue}{6.5} & \color{blue}{2.1} & 0.0 & 0.0 \\
0.0 & 0.0 & 0.0 & 0.0 & 0.0 & \color{blue}{-4.5} & \color{blue}{0.0}\\
0.0 & 0.0 & 0.0 & 0.0 & 0.0 & \color{blue}{0.0} & \color{blue}{-3.5}
\end{bmatrix}
$$

$$F_2$$ through $$F_5$$ are skipped.

As we can see both $$x, c \in \mathbb{R}^m = \mathbb{R}^5$$ and $$F_i, Y \in {\mathbb{S}^n} = {\mathbb{S}^7}$$, so $$m=5$$ and $$n=7$$.

However, the matrices $$F_i$$ and $$Y$$ are broken down into three blocks each so the $$n=7$$ is decomposed into block matrices in $$\mathbb{S}^2$$, $$\mathbb{S}^3$$ and $$\mathbb{S}^2$$ respectively, giving us three blocks of sizes 2, 3 and 2 respectively. 

If we look at the last block, it's zero off diagonals for all matrices. In this case, the matrix can be represented as a diagonal vector (as seen in the code) which will be converted to a matrix. To specify that this matrix block will be represented with a diagonal vector, we use a minus sign before its size in the `bLOCKsTRUCT` vector.

This problem can be stored in a `.dat` file as below:

```json
*Example 2:
*mDim = 5, nBLOCK = 3, {2,3,-2}
   5  =  mDIM
   3  =  nBLOCK
   2    3   -2   = bLOCKsTRUCT
{1.1, -10, 6.6 , 19 , 4.1}
{
{ { -1.4, -3.2 },
  { -3.2,-28   }   }
{ { 15,  -12,    2.1 },
  {-12,   16,   -3.8 },
  {  2.1, -3.8, 15   }   }
  {  1.8, -4.0 }
}
{
{ {  0.5,  5.2 },
  {  5.2, -5.3 }   }
{ {  7.8, -2.4,  6.0 },
  { -2.4,  4.2,  6.5 },
  {  6.0,  6.5,  2.1 }   }
  { -4.5, -3.5 }
}
{
{ { 1.7,  7.0 },
  { 7.0, -9.3 }   }
{ {-1.9, -0.9, -1.3 },
  {-0.9, -0.8, -2.1 },
  {-1.3, -2.1,  4.0 }   }
  {-0.2, -3.7 }
}
{
{ { 6.3, -7.5 },
  {-7.5, -3.3 }   }
{ { 0.2,  8.8,  5.4 },
  { 8.8,  3.4, -0.4 },
  { 5.4, -0.4,  7.5 }   }
  {-3.3, -4.0 }
}
{
{ { -2.4, -2.5 },
  { -2.5, -2.9 }   }
{ {  3.4, -3.2, -4.5 },
  { -3.2,  3.0, -4.8 },
  { -4.5, -4.8,  3.6 }   }
  {  4.8 , 9.7 }
}
{
{ { -6.5, -5.4 },
  { -5.4, -6.6 }   }
{ {  6.7, -7.2, -3.6 },
  { -7.2,  7.3, -3.0 },
  { -3.6, -3.0, -1.4 }   }
  {  6.1, -1.5 }
}
```

## References

Since it was introduced by the developers of SDPA, the official specification of this format is found in the user manual of SDPA on the [official SDPA website](http://sdpa.sourceforge.net/download.html)