---
layout: default
title: Building on Linux
parent: Installation
---

# Building SDPA for Python on Linux

SDPA for Python (`sdpa-python`) is a wrapper on top of the SDPA package. We will go over installing SDPA first, followed by SDPA for Python.

## Otaining and installing SDPA

The primary software package containing SDPA is named simply `sdpa` and the source can be obtained from the [official website](http://sdpa.sourceforge.net/download.html).

SDPAP is a Python wrapper on top of SDPA and can also be obtained from the same link. SDPA can be built by itself or as a part of the SDPAP package, which would require you to link to the SDPA source in the config file as you install it.

To build SDPA (directly or through SDPAP), you need to have the basic build tools, i.e. `make` and `gcc`/`g++` on your computer. If you are using Linux, I assume you already have these tools, or know how to obtain them from your package manager.

### Additional Requirements

Building SDPA also requires you to have

1. A **BLAS/LAPACK library**.
    On Linux, `libblas` and `liblapack` are provided by your distribution. Mostly Linux would use the OpenBLAS implementation, owing to it's permissive license. The best way to check if you have them installed is to compile a very basic Hello World program with `g++ hello.c -lblas -llapack`. If they aren't there, you can refer to instructions for installing them for your distribution.

    <!-- You may want to use some other BLAS/LAPACK implementation than the usual ones for your operating system. In that case, please read [this guide](#) to have an overview of contemporary BLAS/LAPACK implementations and which one to choose. -->

2. A **FORTRAN compiler**.
    It will be required to build [MUMPS](http://mumps.enseeiht.fr) which is written in Fortran.

    On Linux, `gfortran` is installed if you have `g++` installed.


### Minor fixes

Since `sdpa` hasn't been updated for a while you **might** need to add `-fallow-argument-mismatch` flag for the Fortran compiler. **Do this only if you run into a compiler error** for `gfortran`. You need to add this in the string defined on the following line in the `configure` script in the root folder:

Assuming you haven't changed this line for compiler optimization reasons, this line should look line this:

```bash
FCFLAGS="$FCFLAGS -Wall -fPIC -funroll-all-loops"
```

Modify it so it should look like

```bash
FCFLAGS="$FCFLAGS -Wall -fPIC -funroll-all-loops -fallow-argument-mismatch"
```

You may want to modify the other flags for processor specific reasons, but we will not cover them in this post.

### Configure (and test build)

The next step should be to run the configure script in the `sdpa` root directory. Assuming you have `sdpa` unzipped on your Desktop:

```bash
./configure --prefix=$HOME/Desktop/sdpa-7.3.8
```

This should check whether all requirements are met for the build. Once done, issue the make command:

```bash
make
```

This will generate the `make.inc` file which `sdpa-python` would require you to link to.

### Test run the SDPA binary

When the `make` completes, the `sdpa` binary will be in the same directory. You can test run it using one of the provided example files:

```bash
./sdpa example1.dat example1.out
```

This should complete a test run the `sdpa` binary.

## Otaining and installing SDPA Python wrapper

Once you have built and done a test run on `sdpa`, you have a working SDPA binary, it's finally time to build and install `sdpa-python`.

You can obtain it by

```bash
git clone https://github.com/sdpa-python/sdpa-python.git
```

Then do

```bash
cd sdpa-python
```

There is a file `setupcfg.py` in the root directory of the `sdpap` package. We need to edit it and provide the link to the `make.inc` in the `sdpa` folder. Assuming it's located on your desktop:

```bash
SDPA_MAKEINC = '$HOME/Desktop/sdpa-7.3.8/etc/make.inc'
```

### Otaining SPOOLES headers

Another thing you need to specify is the location of the SPOOLES headers

On Linux, `libspooles` is provided by your distribution (as `libspooles` or `libspooles-dev`) and installed system wide (so you need not provide the library path for it in `setupcfg.py`). The best way to check if you have it is to compile a very basic Hello World program with `g++ hello.c -lspooles`. If it throws an error, you can refer to instructions for installing them for your distribution.

You will however, still need to download it because SPOOLES headers are imported by the `sdpa-python` (and you will need to provide the include path for it in `setupcfg.py`). It can be obtained from the official [SPOOLES webpage](http://www.netlib.org/linalg/spooles/spooles.2.2.html). [Debian package sources](http://ftp.de.debian.org/debian/pool/main/s/spooles/spooles_2.2.orig.tar.gz) also provides it.

Once downloaded, edit the following line in `setupcfg.py` to specify the SPOOLES headers location

```bash
SPOOLES_INCLUDE = '/path/to/spooles/' # headers must be provided
```

### Install `sdpa-python` Package

Finally, build and install `sdpa-python` package. While still in the `sdpa-python` directory, do:

```bash
python setup.py install
```

Assuming all goes well, you should be able to do `import sdpap` in Python. If you run into problems, `cd` out of the `sdpa-python` directory. Having folders with the name `sdpap` in your current directory, you will run into problems while doing `import sdpap` in Python (it will complain about a circular dependency problem).

## Cleanup

Once installed, you can delete both the `sdpa` and `sdpa-python` folders.
