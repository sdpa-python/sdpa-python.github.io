---
layout: default
title: Building on macOS
parent: Installation
sdpa_latest_version: 7.3.17
# mumps_url: http://mumps.enseeiht.fr
mumps_url: https://mumps-solver.org
---

# Building SDPA for Python on macOS

SDPA for Python (`sdpa-python`) is a wrapper on top of the SDPA package. We will go over building SDPA first, followed by SDPA for Python.

## Obtaining and building the backend

As a backend, we can use either the regular SDPA package, or the SDPA Multiprecision variant. SDPA Multiprecision is a fork of SDPA-GMP.

If you choose to use **SDPA Multiprecision**, please follow the instructions in the README of its [GitHub repository](https://github.com/sdpa-python/sdpa-multiprecision), and then skip directly to the [next section](#obtaining-and-installing-sdpa-python-wrapper) on building the Python wrapper.

If you choose to use the **regular SDPA package**, we will download it from the [official website](http://sdpa.sourceforge.net/download.html). Currently, the latest version is {{page.sdpa_latest_version}}.

Sourceforge download links involve redirects. If you are using `wget`, it automatically redirects, however, if you are using `curl`, you have to add `-L` flag to enable redirects:

```bash
curl -L -O https://downloads.sourceforge.net/project/sdpa/sdpa/sdpa_{{page.sdpa_latest_version}}.tar.gz
```

Once downloaded, unzip it using:

```bash
tar -zxf sdpa_{{page.sdpa_latest_version}}.tar.gz
```

To build SDPA (with or without the Python wrapper), you need to have the basic build tools, i.e. `make` and `gcc`/`g++` on your computer. On macOS, these tools come bundled with Xcode Command Line Tools which can be installed by:

```bash
xcode-select --install
```

### Additional Requirements

Building SDPA also requires you to have

1. A **BLAS/LAPACK library**.
    On macOS, they are installed as part of the Apple Accelerate framework. I recommend you stick with Apple's BLAS/LAPACK implementation because it would work regardless of whether you have an Intel based Mac or M1 based Mac.

    The best way to check if they are present is to compile a very basic Hello World program with `g++ hello.c -lblas -llapack`. On macOS, `-lblas` and `-llapack` will use the BLAS/LAPACK provided by Accelerate framework.

2. A **FORTRAN compiler**.
    It will be required to build [MUMPS]({{page.mumps_url}}) which is written in Fortran.

    On macOS, the most convenient to install and recent `gfortran` binary that I found at time of writing is provided by [this GitHub repository](https://github.com/fxcoudert/gfortran-for-macOS).


### Minor fixes

The `sdpa` buildsystem will download [MUMPS]({{page.mumps_url}}) and build it. A `Makefile` has been provided inside the `mumps` subfolder of `sdpa` source. Depending on the version of `gfortran` that you use, you may or may not need the `-fallow-argument-mismatch` flag provided in this file. 

1. Older versions of `gfortran` will not need this flag and do not recognize it. If while running `make`, you run into a compiler error for `gfortran`, remove this flag.

2. Secondly, on macOS you may not have `wget` on your system. The `Makefile` will use `wget` to download the [MUMPS]({{page.mumps_url}}) source. You can modify the following line in the `Makefile` in the `mumps` subfolder to use `curl` instead of `wget`:

    ```bash
    wget http://ftp.de.debian.org/debian/pool/main/m/mumps/${MUMPS_TAR_FILE}
    ```

    so it should look like

    ```bash
    curl -O http://ftp.de.debian.org/debian/pool/main/m/mumps/${MUMPS_TAR_FILE}
    ```

### Build

The next step should be to run the configure script in the `sdpa` root directory. Assuming you have `sdpa` unzipped on your Desktop:

```bash
./configure --prefix=$HOME/Desktop/sdpa-{{page.sdpa_latest_version}}
```

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

Once you have built and done a test run on `sdpa` (or `sdpa_gmp` if using the Multiprecision backend), you have a working SDPA binary, it's finally time to build and install `sdpa-python`.

We need SPOOLES for building `sdpa-python`.

### Obtaining and building the SPOOLES library and headers

If you are using the **SDPA Multiprecision** backend, it contains SPOOLES and will build it as part of the buildsystem. In that case, you can skip directly to the next subsection.

If you are using the regular SDPA backend, you will have to download as well as build the SPOOLES library.

SPOOLES can be obtained from the official [SPOOLES webpage](http://www.netlib.org/linalg/spooles/spooles.2.2.html) or the [Debian package sources](http://ftp.de.debian.org/debian/pool/main/s/spooles/spooles_2.2.orig.tar.gz).

```bash
curl -O http://ftp.de.debian.org/debian/pool/main/s/spooles/spooles_2.2.orig.tar.gz
mkdir spooles
tar -zxf spooles_2.2.orig.tar.gz -C spooles
cd spooles
```

The `Make.inc` file (located in the root of the extracted `spooles` folder) needs to be patched before SPOOLES can be successfully built. Patch and build the library as below.

```bash
curl -O https://raw.githubusercontent.com/sdpa-python/sdpa-multiprecision/main/spooles/patches/patch-Make.inc
patch -p0 < patch-Make.inc
make lib
```

By default, the `Makefile` will load the code into `spooles.a` and this may cause the library to be not locatable (despite providing the search path in `setupcfg.py`). To avoid this issue, please rename this file to `libspooles.a`.

```bash
mv spooles.a libspooles.a
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


1. The location of SDPA headers and static library (i.e. `libsdpa.a` or `libsdpa_gmp.a`):

    ```python
    SDPA_DIR = '/path/to/sdpa_package_name'
    ```

2. Only if using the **regular SDPA** backend, the location of SPOOLES library and headers (for Multiprecision backend these variables need not be changed):

    ```python
    SPOOLES_DIR = '/path/to/spooles/'
    SPOOLES_INCLUDE = SPOOLES_DIR
    ```

3. The location of `libgfortran.a` and `libquadmath.a`. Assuming you installed it from [this GitHub repository](https://github.com/fxcoudert/gfortran-for-macOS) with the default options:

    ```python
    GFORTRAN_LIBS ='/usr/local/gfortran/lib'
    ```

4. Only if using the **SDPA Multiprecision** backend, the location of GMP library and headers:

    ```python
    GMP_DIR = '/path/to/gmp-version'
    ```

5. Lastly, if you are using the **SDPA Multiprecision** backend, please set `USEGMP = True` in this file.

### Install `sdpa-python` Package

Finally, build and install `sdpa-python` package. While still in the `sdpa-python` directory, do:

```bash
python setup.py install
```

Assuming all goes well, you should be able to do `import sdpap` in Python. If you run into problems, `cd` out of the `sdpa-python` directory. Having folders with the name `sdpap` in your current directory, you will run into problems while doing `import sdpap` in Python (it will complain about a circular dependency problem).

## Cleanup

Once installed, you can delete both the `sdpa` and `sdpa-python` folders.
