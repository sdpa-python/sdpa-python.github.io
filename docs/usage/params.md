---
layout: default
title: Solver Options
parent: Usage
nav_order: 1
---

# Solver Options

In addition to `A`, `b`, `c`, `K`, `J`, an optional parameter may be specified to set the solver options. This is a dictionary containing the set of parameters.

```python
sdpap_options = {}
sdpap_options['print'] = 'no'
x, y, sdpapinfo, timeinfo, sdpainfo = sdpap.solve(A, b, c, K, J, sdpap_options)
```

The details of the parameters are as below:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `maxIteration` | The maximum number of iterations. | `100` |
| `epsilonStar` | The accuracy of an approximate optimal solution for primal and dual SDP. **If using the multiprecision backend, this parameter may be reduced by orders of magnitude to achieve extended precision.** | `1.0E-7` |
| `lambdaStar` | An initial point. | `1.0E2` |
| `omegaStar` | The search region for an optimal solution. | `2.0` |
| `lowerBound` | Lower bound of the minimum objective value of the primal SDP. | `-1.0E5` |
| `upperBound` | Upper bound of the maximum objective value of the dual SDP. | `1.0E5` |
| `betaStar` | The parameter for controlling the search direction if the current point is feasible. | `0.1` |
| `betaBar` | The parameter for controlling the search direction if the current point is infeasible. | `0.2` |
| `gammaStar` | A reduction factor for the primal and dual step lengths. | `0.9` |
| `epsilonDash` | The relative accuracy of an approximate optimal solution between primal and dual SDP. **If using the multiprecision backend, this parameter may be reduced by orders of magnitude to achieve extended precision.** | `1.0E-7` |
| `mpfPrecision` | Numerical precision used by the MPF class provided by the GNU Multiprecision Library. **This is only available on the multiprecision variant, and will be ignored by the regular variant.** | `200` |
| `isSymmetric` | Specify whether to check the symmetricity of input matrices. | `False` |
| `isDimacs` | Specify whether to compute DIMACS ERROR. | `False` |
| `numThreads` | Number of Threads for internal computation. | `multiprocessing.cpu_count()` |
| `convMethod` | Conversion method from CLP to LMI or EQ standard form (`'LMI'` or `'EQ'`). <span style="color:red">At the moment, `convMethod = 'EQ'` is not yet implemented in the code. Please only use `'LMI'`.</span> | `'LMI'` |

The following parameters specify algorithms to exploit sparsity.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `domainMethod` | Algorithm option for domain space. Can be `'none'` (exploiting no sparsity in the domain space), `'clique'` (applying `dconv_cliquetree`) or `'basis'` (applying `dconv_basisrep`) | `'none'` |
| `rangeMethod` | Algorithm option for range space. Can be `'none'` (exploiting no sparsity in the range space), `'clique'` (applying `rconv_cliquetree`) or `'decomp'` (applying `rconv_matdecomp`) | `'none'` |

Two recommended sets of parameter values for exploiting sparsity:

1. `convMethod = 'EQ'`, `domainMethod = 'clique'`, `rangeMethod  = 'decomp'`
2. `convMethod = 'LMI'`, `domainMethod = 'basis'`, `rangeMethod  = 'clique'`

<span style="color:red">At the moment, `domainMethod = 'clique'` and `rangeMethod = 'clique'` are not yet implemented in the code. Please only use the other available options for the `domainMethod` and `rangeMethod` parameters.</span>

The following parameters are for free variables elimination.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `frvMethod` | The method to eliminate free variables (`'split'` or `'elimination'`) | `'split'` |
| `rho` | The parameter of range in split method or pivoting in elimination method. | `0.0` |
| `zeroPoint` | The zero point of matrix operation, determine unboundness, or LU decomposition. | `1.0E-12` |

The below options set printing options.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `print` | Whether to display solver output to stdout. If set to `'display'`, output is printed and if set to `'no'` or empty, no message is print out. | `'display'` |
| `resultFile` | Destination of detail file output. | `''` |
| `sdpaResult` | Destination of file output for SDPA result. | `''` |
| `xPrint` | Format specifier for `yMat` printed to the file. To adjust, see [`fprintf` documentation](https://www.cplusplus.com/reference/cstdio/fprintf/) (for regular backend) or [`gmp_printf` documentation](https://gmplib.org/manual/Formatted-Output-Strings) (for multiprecision backend). `NOPRINT` skips printout. | `'%+8.3e'` (regular backend), `'%+18.12Fe'` (multiprecision backend) |
| `yPrint` | Format specifier for `xVec` printed to the file. To adjust, see [`fprintf` documentation](https://www.cplusplus.com/reference/cstdio/fprintf/) (for regular backend) or [`gmp_printf` documentation](https://gmplib.org/manual/Formatted-Output-Strings) (for multiprecision backend). `NOPRINT` skips printout. | `'%+8.3e'` (regular backend), `'%+18.12Fe'` (multiprecision backend) |
| `sPrint` | Format specifier for `xMat` printed to the file. To adjust, see [`fprintf` documentation](https://www.cplusplus.com/reference/cstdio/fprintf/) (for regular backend) or [`gmp_printf` documentation](https://gmplib.org/manual/Formatted-Output-Strings) (for multiprecision backend). `NOPRINT` skips printout. | `'%+8.3e'` (regular backend), `'%+18.12Fe'` (multiprecision backend) |
| `infPrint` | Format specifier for problem information printed to the file. See [`fprintf` documentation](https://www.cplusplus.com/reference/cstdio/fprintf/) to set it. `NOPRINT` skips printout. | `'%+10.16e'` |
