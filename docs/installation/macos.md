---
layout: default
title: Building on macOS
parent: "Installation"
---

# Building SDPA for Python on macOS

SDPA for Python (`sdpa-python`) is a wrapper on top of the SDPA package. We will go over installing SDPA first, followed by SDPA for Python.

## Otaining and installing SDPA

The primary software package containing SDPA is named simply `sdpa` and the source can be obtained from the [official website](http://sdpa.sourceforge.net/download.html).

SDPAP is a Python wrapper on top of SDPA and can also be obtained from the same link. SDPA can be built by itself or as a part of the SDPAP package, which would require you to link to the SDPA source in the config file as you install it.

To build SDPA (directly or through SDPAP), you need to have the basic build tools, i.e. `make` and `gcc`/`g++` on your computer. On macOS, these tools come bundled with Xcode Command Line Tools which can be installed by:

```bash
xcode-select --install
```

### Additional Requirements

Building SDPA also requires you to have

1. A **BLAS/LAPACK library**.
    <!-- On **Windows**, they are not there by default. I recommend you get the BLAS/LAPACK through Intel's oneAPI or AMD AOCC, depending on your processor. -->

    On macOS, they are installed as part of the Apple Accelerate framework. I recommend you stick with Apple's BLAS/LAPACK implementation because it would work regardless of whether you have an Intel based Mac or M1 based Mac.

    <!-- You may want to use some other BLAS/LAPACK implementation than the usual ones for your operating system. In that case, please read [this guide](#) to have an overview of contemporary BLAS/LAPACK implementations and which one to choose. -->

2. A **FORTRAN compiler**.
    It will be required to build [MUMPS](http://mumps.enseeiht.fr) which is written in Fortran.

    <!-- On **Windows**, I recommend you stick with the fortran compiler that comes with Intel's oneAPI or AMD AOCC depending on your processor. You will be installing oneAPI or AOCC in any case, to get the BLAS/LAPACK libraries. -->

    On macOS, the most convenient to install and recent `gfortran` binary that I found at time of writing is provided by [this GitHub repository](https://github.com/fxcoudert/gfortran-for-macOS).


### Minor fixes

Since `sdpa` hasn't been updated for a while, there's another couple of fixes you might need to do before you install.

1. If you are not on **Linux**, you may not have `wget` on your system. The `Makefile` will use `wget` to download the [MUMPS](http://mumps.enseeiht.fr) source. You can modify the following line in the `Makefile` in the `mumps` subfolder to use `curl` instead of `wget`:

    ```bash
    wget http://ftp.de.debian.org/debian/pool/main/m/mumps/${MUMPS_TAR_FILE}
    ```

    so it should look like

    ```bash
    curl -O http://ftp.de.debian.org/debian/pool/main/m/mumps/${MUMPS_TAR_FILE}
    ```

2. If using `gfortran` on macOS, you **might** need to add `-fallow-argument-mismatch` flag for the Fortran compiler. **Do this only if you run into a compiler error** for `gfortran`. You need to add this in the string defined on the following line in the `configure` script in the root folder:

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

### Otaining SPOOLES library and headers

Another thing you need to specify is the location of the SPOOLES headers

On <!--**Windows** and -->macOS you will have to download as well as build it. You will need to provide both the include and library path for it in `setupcfg.py`. To build it, cd to the directory where you extracted SPOOLES and do `make lib`.

**Note 1**: Remove the line `CC = /usr/lang-4.0/bin/cc` in `Make.inc` to let it use the default compiler on your system (otherwise it will throw an error)

**Note 2**: By default, the `Makefile` will load the code into `spooles.a` and this may cause the library to be not locateable (despite providing the search path in `setupcfg.py`). To avoid it, please rename this file to `libspooles.a`.

Once downloaded and built, edit the following line in `setupcfg.py` to specify the SPOOLES headers location

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
