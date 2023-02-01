---
layout: default
title: Building on Linux
parent: Installation
sdpa_latest_version: 7.3.16
# mumps_url: http://mumps.enseeiht.fr
mumps_url: https://mumps-solver.org
---

# Building SDPA for Python on Linux

SDPA for Python (`sdpa-python`) is a wrapper on top of the SDPA package. We will go over building SDPA first, followed by SDPA for Python.

## Obtaining and building SDPA

The primary software package containing SDPA is named simply `sdpa` and the source can be obtained from the [official website](http://sdpa.sourceforge.net/download.html). Currently, the latest version is {{page.sdpa_latest_version}}.

Sourceforge does not allow a direct download link, however, the specific file required is [sdpa_{{page.sdpa_latest_version}}.tar.gz](https://downloads.sourceforge.net/project/sdpa/sdpa/sdpa_{{page.sdpa_latest_version}}.tar.gz).

Once downloaded, unzip it using:

```bash
tar -zxf sdpa_{{page.sdpa_latest_version}}.tar.gz
```

To build SDPA (with or without the Python wrapper), you need to have the basic build tools, i.e. `make` and `gcc`/`g++` on your computer. On Linux, please refer to the instructions for your package manager if you do not already have them.

### Additional Requirements

Building SDPA also requires you to have

1. A **BLAS/LAPACK library**.
    On Linux, `libblas` and `liblapack` are provided by your distribution.

    Mostly Linux would have two alternative packages, `libblas` and `libopenblas`. `libblas` is the [reference implementation](http://www.netlib.org/blas/) while `libopenblas` is the optimized [OpenBLAS](https://www.openblas.net/) implementation. OpenBLAS is recommended as it gives much better performance.

    The best way to check if it's present is to compile a very basic Hello World program with `g++ hello.c -lopenblas` (or `g++ hello.c -lblas -llapack` for the default, i.e. reference BLAS). If you don't have OpenBLAS, you can refer to the instructions for your distribution for installing it.

2. A **FORTRAN compiler**.
    It will be required to build [MUMPS]({{page.mumps_url}}) which is written in Fortran.

    On Linux, `gfortran` is installed if you have `g++` installed.


### Minor fixes

The `sdpa` buildsystem will download [MUMPS]({{page.mumps_url}}) and build it. A `Makefile` has been provided inside the `mumps` subfolder of `sdpa` source. Depending on the version of `gfortran` that you use, you may or may not need the `-funroll-all-loops` flag provided in this file. 

Older versions of `gfortran` will not need this flag and do not recognize it. If while running `make`, you run into a compiler error for `gfortran`, remove this flag.

### Build

The next step should be to run the configure script in the `sdpa` root directory. Assuming you have `sdpa` unzipped in your home folder:

```bash
./configure --prefix=$HOME/sdpa-{{page.sdpa_latest_version}}
```

By default, the configure script will link against the [reference BLAS](http://www.netlib.org/blas/). If you want to use an alternate BLAS/LAPACK, you can override the flags `--with-blas` and `--with-lapack`.

For example, if you want to use OpenBLAS, this should become

```bash
./configure --prefix=$HOME/sdpa-{{page.sdpa_latest_version}} --with-blas="-lopenblas" --with-lapack="-lopenblas"
```

`libopenblas` provides both BLAS and LAPACK.

This should check whether all requirements are met for the build. Once done, issue the make command:

```bash
make
```

This will generate the `sdpa` binary as well as `libsdpa.a` static library. The `libsdpa.a` library is what the Python wrapper will link against.

This will also generate the `make.inc` file which `sdpa-python` would require you to link to.

### Test run the SDPA binary

Let's do a test run on the `sdpa` binary to make sure all is well until this point. While still within the `sdpa-{{page.sdpa_latest_version}}` folder, do

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

There is a file `setupcfg.py` in the root of `sdpa-python`. We need to edit it and provide

1. The link to the `make.inc` in the `sdpa` folder (so it can find `libsdpa.a`). Assuming you extracted it in your home folder:

    ```python
    SDPA_MAKEINC = '/home/yourusername/sdpa-{{page.sdpa_latest_version}}/etc/make.inc'
    ```

2. The location of SPOOLES library and headers. Assuming you extracted it in your home folder:

    ```python
    SPOOLES_INCLUDE = '/home/yourusername/spooles/'
    ```

3. Names of BLAS/LAPACK if you did not use the reference BLAS. Please use the same BLAS/LAPACK that you used while building `sdpa`. Assuming you used OpenBLAS:

    ```python
    LAPACK_NAME = 'openblas'
    BLAS_NAME = 'openblas'
    ```

### Install `sdpa-python` Package

Finally, build and install `sdpa-python` package. While still in the `sdpa-python` directory, do:

```bash
python setup.py install
```

Assuming all goes well, you should be able to do `import sdpap` in Python. If you run into problems, `cd` out of the `sdpa-python` directory. Having folders with the name `sdpap` in your current directory, you will run into problems while doing `import sdpap` in Python (it will complain about a circular dependency problem).

## Cleanup

Once installed, you can delete both the `sdpa` and `sdpa-python` folders.
