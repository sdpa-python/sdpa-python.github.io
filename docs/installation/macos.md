---
layout: default
title: Building on macOS
parent: Installation
---

# Building SDPA for Python on macOS

SDPA for Python (`sdpa-python`) is a wrapper on top of the SDPA package. We will go over building SDPA first, followed by SDPA for Python.

## Obtaining and installing SDPA

The primary software package containing SDPA is named simply `sdpa` and the source can be obtained from the [official website](http://sdpa.sourceforge.net/download.html). Sourceforge does not allow a direct download link, however, the specific file required is [sdpa_7.3.8.tar.gz](https://downloads.sourceforge.net/project/sdpa/sdpa/sdpa_7.3.8.tar.gz).

Once downloaded, unzip it using:

```bash
tar -zxf sdpa_7.3.8.tar.gz
```

To build SDPA (with or without the Python wrapper), you need to have the basic build tools, i.e. `make` and `gcc`/`g++` on your computer. On macOS, these tools come bundled with Xcode Command Line Tools which can be installed by:

```bash
xcode-select --install
```

### Additional Requirements

Building SDPA also requires you to have

1. A **BLAS/LAPACK library**.
    On macOS, they are installed as part of the Apple Accelerate framework. I recommend you stick with Apple's BLAS/LAPACK implementation because it would work regardless of whether you have an Intel based Mac or M1 based Mac.

2. A **FORTRAN compiler**.
    It will be required to build [MUMPS](http://mumps.enseeiht.fr) which is written in Fortran.

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

### Obtaining SPOOLES library and headers

On macOS we have to download as well as build the SPOOLES library.

SPOOLES can be obtained from the official [SPOOLES webpage](http://www.netlib.org/linalg/spooles/spooles.2.2.html) or the [Debian package sources](http://ftp.de.debian.org/debian/pool/main/s/spooles/spooles_2.2.orig.tar.gz).

```bash
curl -O http://www.netlib.org/linalg/spooles/spooles.2.2.tgz
mkdir spooles
tar -zxf spooles.2.2.tgz -C spooles # (OR spooles_2.2.orig.tar.gz if from Debian Sources)
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

There is a file `setupcfg.py` in the root of `sdpa-python`. We need to edit it and provide the link to the `make.inc` in the `sdpa` folder (so it can find `libsdpa.a`) as well the location of SPOOLES library and headers.

Assuming both `sdpa` and `spooles` are located on your desktop, these lines should become:

```python
SDPA_MAKEINC = '/Users/yourusername/Desktop/sdpa-7.3.8/etc/make.inc'
```

and

```python
SPOOLES_INCLUDE = '/Users/yourusername/Desktop/spooles/' # headers must be provided
```

### Install `sdpa-python` Package

Finally, build and install `sdpa-python` package. While still in the `sdpa-python` directory, do:

```bash
python setup.py install
```

Assuming all goes well, you should be able to do `import sdpap` in Python. If you run into problems, `cd` out of the `sdpa-python` directory. Having folders with the name `sdpap` in your current directory, you will run into problems while doing `import sdpap` in Python (it will complain about a circular dependency problem).

### Known issues

If while doing `import sdpap` you still run into problems pertaining to `libgfortran` or `libgcc`, make sure that the `libgfortran` and `libgcc` that is being dynamically loaded is the same one that was used during this build. If you obtained `gfortran` from [the GitHub repository](https://github.com/fxcoudert/gfortran-for-macOS) linked above, copy `libgfortran.5.dylib` and `libgcc_s.1.1.dylib` from `/usr/local/gfortran/lib/` into the paths that Python searches for dynamic libraries.

## Cleanup

Once installed, you can delete both the `sdpa` and `sdpa-python` folders.
