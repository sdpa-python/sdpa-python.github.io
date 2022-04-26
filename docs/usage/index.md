---
layout: default
title: Usage
has_children: true
nav_order: 4
---

# Using SDPA for Python

At the moment, we are still working on this documentation. In the meanwhile, you may refer to the `sdpa` as well as `sdpap` manuals on the [official SDPA website](http://sdpa.sourceforge.net/download.html).

In the meanwhile, below example shows how to read and solve a CLP formatted file (in `examples` folder):

```python
import sdpap

A, b, c, K, J = sdpap.readproblem('example1.clp')
x, y, sdpapinfo , timeinfo , sdpainfo = sdpap.solve(A,b,c,K,J)
```

And here's how to read and solve an SDPA sparse formatted file (`.dat-s` extension):

```python
import sdpap

A, b, c, K, J = sdpap.fromsdpa('mcp500-3.dat-s')
x, y, sdpapinfo , timeinfo , sdpainfo = sdpap.solve(A,b,c,K,J)
```

SDPLIB is a database of problems in SDPA sparse format. It's available as a [GitHub repository](https://github.com/vsdp/SDPLIB).
