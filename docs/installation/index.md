---
layout: default
title: Installation
has_children: true
nav_order: 2
---

# Installing SDPA for Python

SDPA for Python can be installed for `x86_64` through `pip`. Two variants of this package are available on the Python Package Index (PyPI). The package using the SDPA (OpenBLAS) backend can be installed by

```bash
pip install sdpa-python
```

The package using the SDPA Multiprecision (GMP) backend can be installed by

```bash
pip install sdpa-multiprecision
```

Wheels for other architectures are not currently available but we have provided build instructions for Windows, Linux and macOS if you would like to build it from source. `sdpa-python` has currently not been tested on other architectures.