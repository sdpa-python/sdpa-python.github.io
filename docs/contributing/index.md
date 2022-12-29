---
layout: default
title: Contributing
has_children: false
nav_order: 5
---

# Contributing

Below is a non-exhaustive list of opportunities for contributing to the development of SDPA for Python.

### Small scope projects

1. Improving this documentation.
2. Writing unit tests/benchmarks.

### Medium scope projects

1. Implementation of `clp_toEQ` and `result_fromEQ` routines in [convert.py](https://github.com/sdpa-python/sdpa-python/blob/main/sdpap/convert.py) to allow user to choose SeDuMi Primal format as the internal representation of the problem (i.e. allow user to set `convMethod` to `EQ` in the solver options). Currently only `clp_toLMI` and `result_fromLMI` are present which convert the problem to SeDuMi Dual format.
2. Implementation of domain and range space conversion method using clique trees (i.e. `rconv_cliquetree`, `rconv_cliqueresult`, `dconv_cliquetree` and `dconv_cliqueresult` in [spcolo.py](https://github.com/sdpa-python/sdpa-python/blob/main/sdpap/spcolo/spcolo.py)

### Large scope projects

1. Add support for SOCP constraints to make the package a complete SQLP solver. This is a contribution to SDPA (and not just SDPA for Python).
