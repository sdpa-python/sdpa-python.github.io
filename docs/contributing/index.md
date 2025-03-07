---
layout: default
title: Contributing
has_children: false
nav_order: 5
---

# Contributing

SDPA for Python is a fork of SDPAP, SDPA's official Python wrapper. SDPA Multiprecision is a fork of SDPA-GMP, and adds an Application Binary Interface to allow its usage as a callable library. This project is an effort to maintain both for the Python ecosystem. Below is a non-exhaustive list of opportunities for contributing to one or other other.

### Common projects

1. You can help by improving this documentation website.
2. Since building from source requires compiling multiple libraries, sometimes issues may arise during compilation in some environments which otherwise do not show up in the CI infrastructure (the Python wheels are built on GitHub runners). If you run into problems building from source, please open an issue in the relevant GitHub repository so it can be investigated.

### SDPA for Python (SDPAP) specific projects

SDPAP contains (Python translations of) some routines from [SparseCoLO](https://github.com/sdpa-python/SparseCoLO) [1] to provide a more general API than SDPA, and also to exploit sparsity in nonlinear matrix inequalities. Currently only a subset of these routines are present in SDPA for Python. If you can help translate other routines, that can extend the abilities of SDPA for Python. The [Solver Options]({% link docs/usage/params.md %}) contains more information on the unimplemented routines.

### SDPA Multiprecision specific projects

SDPA Multiprecision uses MPACK, a multiprecision version of BLAS/LAPACK based on the GNU Multiprecision Library. In future, this may be migrated to use [MPLAPACK](https://github.com/nakatamaho/mplapack). Any effort to this end will be appreciated.

### Testing

A GitHub Actions pipeline is setup to automatically test the Python code using CVXPY tests (using both SDPA and SDPA Multiprecision backends). Additional tests may be added under the `tests` folder to test features not otherwise tested by CVXPY tests.

If you want to contribute to SDPA for Python, you can use SparseCoLO as a reference.

[1] Sunyoung Kim, Masakazu Kojima, Martin Mevissen and Makoto Yamashita, "Exploiting sparsity in linear and nonlinear matrix inequalities via positive semidefinite matrix completion," Mathematical Programming, 129(1), 33–68. doi: [10.1007/s10107-010-0402-6](https://doi.org/10.1007/s10107-010-0402-6)
