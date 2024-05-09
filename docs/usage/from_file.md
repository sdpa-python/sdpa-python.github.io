---
layout: default
title: Solving from File
parent: Usage
nav_order: 2
---

# Solving problems from a file

SDPA for Python can read and solve problems saved in CLP and SDPA Sparse formats.

The below example shows how to read and solve a CLP formatted file (i.e. one having `.clp` extension):

```python
import sdpap

A, b, c, K, J = sdpap.readproblem('example1.clp')
x, y, sdpapinfo, timeinfo, sdpainfo = sdpap.solve(A,b,c,K,J)
```

Example CLP formatted problems can be found in the `examples` folder of the SDPA for Python [GitHub repository](https://github.com/sdpa-python/sdpa-python).

The below example shows how to read and solve an SDPA sparse formatted file (i.e. one having `.dat-s` extension):

```python
import sdpap

A, b, c, K, J = sdpap.fromsdpa('mcp500-3.dat-s')
x, y, sdpapinfo, timeinfo, sdpainfo = sdpap.solve(-A,-b,-c,K,J)
```

The sign flipping is a consequence of the [inverse relationship]({% link docs/formats/sdpa_sedumi.md %}) between SDPA and SeDuMi formats (and that the (CLP) format used by SDPA for Python is a generalization of the SeDuMi format).

SDPLIB is a database of problems in SDPA sparse format. It's available as a [GitHub repository](https://github.com/vsdp/SDPLIB).
