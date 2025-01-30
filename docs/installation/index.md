---
layout: default
title: Installation
has_children: true
nav_order: 2
---

# Installing SDPA for Python

SDPA for Python can be installed via `pip` on Windows, macOS and Linux for `x86_64` (as well as for `arm64` on Linux). Two variants of this package are available on the Python Package Index (PyPI). The package using the SDPA (OpenBLAS) backend can be installed by

```bash
pip install sdpa-python
```

The package using the [SDPA Multiprecision](https://github.com/sdpa-python/sdpa-multiprecision) (GMP) backend can be installed by

```bash
pip install sdpa-multiprecision
```

{: .success }
**Macs with Apple Silicon**
Both `sdpa-python` and `sdpa-multiprecision` wheels are now available for ARM based Macs.

{: .attention }
**macOS Users**
The regular variant (i.e. `sdpa-python` wheels) are built for minimum macOS versions of 13 and 14 (for Intel and ARM respectively). If you cannot install version 0.2.2, please run `pip install --upgrade pip setuptools wheel` before installing `sdpa-python`.
