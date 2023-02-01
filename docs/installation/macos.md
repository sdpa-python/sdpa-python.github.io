---
layout: default
title: Building on macOS
parent: Installation
sdpa_latest_version: 7.3.16
# mumps_url: http://mumps.enseeiht.fr
mumps_url: https://mumps-solver.org
---

# Building SDPA for Python on macOS

SDPA for Python (`sdpa-python`) is a wrapper on top of the SDPA package. We will go over building SDPA first, followed by SDPA for Python.

## Obtaining and building SDPA

The primary software package containing SDPA is named simply `sdpa` and the source can be obtained from the [official website](http://sdpa.sourceforge.net/download.html). Currently, the latest version is {{page.sdpa_latest_version}}.

Sourceforge does not allow a direct download link, however, the specific file required is [sdpa_{{page.sdpa_latest_version}}.tar.gz](https://downloads.sourceforge.net/project/sdpa/sdpa/sdpa_{{page.sdpa_latest_version}}.tar.gz).

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

The `sdpa` buildsystem will download [MUMPS]({{page.mumps_url}}) and build it. A `Makefile` has been provided inside the `mumps` subfolder of `sdpa` source. Depending on the version of `gfortran` that you use, you may or may not need the `-funroll-all-loops` flag provided in this file. 

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

Once you have built and done a test run on `sdpa`, you have a working SDPA binary, it's finally time to build and install `sdpa-python`.

We need SPOOLES for building `sdpa-python`.

### Obtaining SPOOLES library and headers

On macOS we have to download as well as build the SPOOLES library.

SPOOLES can be obtained from the official [SPOOLES webpage](http://www.netlib.org/linalg/spooles/spooles.2.2.html) or the [Debian package sources](http://ftp.de.debian.org/debian/pool/main/s/spooles/spooles_2.2.orig.tar.gz).

```bash
curl -O http://ftp.de.debian.org/debian/pool/main/s/spooles/spooles_2.2.orig.tar.gz
mkdir spooles
tar -zxf spooles_2.2.orig.tar.gz -C spooles
```

Open `Make.inc` (located in the root of the extracted `spooles` folder) in a text editor and remove the line `CC = /usr/lang-4.0/bin/cc`. This will let it use the default compiler on your system (otherwise it will throw an error).

To build it, cd to the directory where you extracted SPOOLES and do `make lib`.

```bash
cd spooles
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


1. The link to the `make.inc` in the `sdpa` folder (so it can find `libsdpa.a`). Assuming you extracted it on your Desktop:

    Assuming you extracted it on your Desktop:

    ```python
    SDPA_MAKEINC = '/Users/yourusername/Desktop/sdpa-{{page.sdpa_latest_version}}/etc/make.inc'
    ```

2. The location of SPOOLES library and headers. Assuming you extracted it on your Desktop:

    ```python
    SPOOLES_INCLUDE = '/Users/yourusername/Desktop/spooles/'
    ```

3. The location of `libgfortran.a` and `libquadmath.a`. Assuming you installed it from [this GitHub repository](https://github.com/fxcoudert/gfortran-for-macOS) with the default options:

    ```python
    GFORTRAN_LIBS ='/usr/local/gfortran/lib'
    ```

### Install `sdpa-python` Package

Finally, build and install `sdpa-python` package. While still in the `sdpa-python` directory, do:

```bash
python setup.py install
```

Assuming all goes well, you should be able to do `import sdpap` in Python. If you run into problems, `cd` out of the `sdpa-python` directory. Having folders with the name `sdpap` in your current directory, you will run into problems while doing `import sdpap` in Python (it will complain about a circular dependency problem).

## Cleanup

Once installed, you can delete both the `sdpa` and `sdpa-python` folders.
