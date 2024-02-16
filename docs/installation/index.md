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

{: .attention }
**Apple Silicon**
`sdpa-multiprecision` wheels are available for Apple Silicon. The regular variant is currently only installable from source (instructions below). Due to a lack of a Fortran cross compiler, we are not able to provide macOS `arm64` wheels for the regular variant.
