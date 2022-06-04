---
layout: default
title: Building on Linux
parent: Installation
---

# Building SDPA for Python on Linux

SDPA for Python (`sdpa-python`) is a wrapper on top of the SDPA package. We will go over building SDPA first, followed by SDPA for Python.

## Obtaining and installing SDPA

The primary software package containing SDPA is named simply `sdpa` and the source can be obtained from the [official website](http://sdpa.sourceforge.net/download.html). Sourceforge does not allow a direct download link, however, the specific file required is [sdpa_7.3.8.tar.gz](https://downloads.sourceforge.net/project/sdpa/sdpa/sdpa_7.3.8.tar.gz).

Once downloaded, unzip it using:

```bash
tar -zxf sdpa_7.3.8.tar.gz
```

To build SDPA (with or without the Python wrapper), you need to have the basic build tools, i.e. `make` and `gcc`/`g++` on your computer. On Linux, please refer to the instructions for your package manager if you do not already have them.

### Additional Requirements

Building SDPA also requires you to have

1. A **BLAS/LAPACK library**.
    On Linux, `libblas` and `liblapack` are provided by your distribution. The best way to check if you have them installed is to compile a very basic Hello World program with `g++ hello.c -lblas -llapack`. If they aren't there, you can refer to instructions for installing them for your distribution.

    Mostly Linux would have two alternative packages, `libblas` and `libopenblas`. `libblas` is the [reference implementation](http://www.netlib.org/blas/) while `libopenblas` is the optimized [OpenBLAS](https://www.openblas.net/) implementation. OpenBLAS is recommended as it gives much better performance.

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

### Build

The next step should be to run the configure script in the `sdpa` root directory. Assuming you have `sdpa` unzipped on your Desktop:

```bash
./configure --prefix=$HOME/Desktop/sdpa-7.3.8
```

This should check whether all requirements are met for the build. Once done, issue the make command:

```bash
make
```

This will generate the `sdpa` binary as well as `libsdpa.a` static library. The `libsdpa.a` library is what the Python wrapper will link against.

This will generate the `make.inc` file which `sdpa-python` would require you to link to.

### Test run the SDPA binary

Let's do a test run on the `sdpa` binary to make sure all is well until this point. While still within the `sdpa-7.3.8` folder, do

```bash
./sdpa example1.dat example1.out
```

This should complete a test run the `sdpa` binary using one of the provided example files. If you can see the solver output, and `pdOPT` as the status, the build was successful. We can now move onto building the Python wrapper.

## Obtaining and installing SDPA Python wrapper

Once you have built and done a test run on `sdpa`, you have a working SDPA binary, it's finally time to build and install `sdpa-python`.

We need SPOOLES for building `sdpa-python`.

### Obtaining SPOOLES headers

On Linux, `libspooles` is provided by your distribution (as `libspooles` or `libspooles-dev`) and installed system wide (so you need not provide the library path for it in `setupcfg.py`). The best way to check if you have it is to compile a very basic Hello World program with `g++ hello.c -lspooles`. If it throws an error, you can refer to instructions for installing them for your distribution.

You will however, still need to download it because SPOOLES headers are imported by `sdpa-python` (and you will need to provide the include path for it in `setupcfg.py`). It can be obtained from the official [SPOOLES webpage](http://www.netlib.org/linalg/spooles/spooles.2.2.html) or the [Debian package sources](http://ftp.de.debian.org/debian/pool/main/s/spooles/spooles_2.2.orig.tar.gz).

```bash
wget http://www.netlib.org/linalg/spooles/spooles.2.2.tgz
mkdir spooles
tar -zxf spooles.2.2.tgz -C spooles # (OR spooles_2.2.orig.tar.gz if from Debian Sources)
```

### Obtain and prepare `sdpa-python` for build

You can obtain it by

```bash
git clone https://github.com/sdpa-python/sdpa-python.git
```

Then do

```bash
cd sdpa-python
```

There is a file `setupcfg.py` in the root of `sdpa-python`. We need to edit it and provide the link to the `make.inc` in the `sdpa` folder (so it can find `libsdpa.a`) as well the location of SPOOLES library and headers.

Assuming both `sdpa` and `spooles` are located on your desktop, these lines should become:

```python
SDPA_MAKEINC = '/home/yourusername/Desktop/sdpa-7.3.8/etc/make.inc'
```

and

```python
SPOOLES_INCLUDE = '/home/yourusername/Desktop/spooles/' # headers must be provided
```

### Install `sdpa-python` Package

Finally, build and install `sdpa-python` package. While still in the `sdpa-python` directory, do:

```bash
python setup.py install
```

Assuming all goes well, you should be able to do `import sdpap` in Python. If you run into problems, `cd` out of the `sdpa-python` directory. Having folders with the name `sdpap` in your current directory, you will run into problems while doing `import sdpap` in Python (it will complain about a circular dependency problem).

## Cleanup

Once installed, you can delete both the `sdpa` and `sdpa-python` folders.
