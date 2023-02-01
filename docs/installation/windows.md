---
layout: default
title: Building on Windows
parent: Installation
sdpa_latest_version: 7.3.16
# mumps_url: http://mumps.enseeiht.fr
mumps_url: https://mumps-solver.org
---

# Building SDPA for Python on Windows

SDPA for Python (`sdpa-python`) is a wrapper on top of the SDPA package. We will go over building SDPA first, followed by SDPA for Python.

## Preparation

Owing to the `autotools` based buildsystem of `sdpa` package, we will build it using MinGW (VC++ is currently not supported). It's recommend to download [MSYS2](https://www.msys2.org/) because it has a builtin package manager (`pacman`) which will allow easy installation of not just `mingw` but also `blas`, `lapack` and `spooles` (later needed to build the Python wrapper).

After downloading, follow the installation instructions of MSYS2.

Run **MSYS2 MSYS** shell from the start menu.

The first thing to do is update the package database:

```bash
pacman -Syu
```

If upon finish the last line reads `:: To complete this update all MSYS2 processes including this terminal will be closed. Confirm to proceed [Y/n]`, then repeat the above command after restarting **MSYS2 MSYS** from the start menu.

Even otherwise, restart **MSYS2 MSYS** from the start menu.

Next, install MinGW:

```bash
pacman -S --needed base-devel mingw-w64-x86_64-toolchain
```

This will give us the basic build tools, i.e. `make` and `gcc`/`g++`.

We will remain on the **MSYS2 MSYS** shell until the build step (when we will switch to **MSYS2 MinGW x64**).

## Obtaining and building SDPA

The primary software package containing SDPA is named simply `sdpa` and the source can be obtained from the [official website](http://sdpa.sourceforge.net/download.html). Currently, the latest version is {{page.sdpa_latest_version}}.

Sourceforge does not allow a direct download link, however, the specific file required is [sdpa_{{page.sdpa_latest_version}}.tar.gz](https://downloads.sourceforge.net/project/sdpa/sdpa/sdpa_{{page.sdpa_latest_version}}.tar.gz).

Once downloaded, unzip it using:

```bash
tar -zxf sdpa_{{page.sdpa_latest_version}}.tar.gz
```

**Note**: On Windows, it is common to have a space in the name of the user folder (which is the parent of Desktop). Spaces in folder names will cause problems during the build phase, so it's advised to unzip `sdpa` in a location which does not have spaces. If you do so in `C:`, it's recommended to create a subfolder (e.g. `C:\sdpa`) and give yourself write permissions to it (right click, open Properties and disable "Read-only" option).

To build SDPA (with or without the Python wrapper), you need to have the basic build tools, i.e. `make` and `gcc`/`g++` on your computer. We have obtained them using `pacman` on **MSYS2 MSYS** shell above.

### Additional Requirements

Building SDPA also requires you to have

1. A **BLAS/LAPACK library**.
    Just like Linux, MSYS2 has two alternative packages. [mingw-w64-x86_64-lapack](https://packages.msys2.org/package/mingw-w64-x86_64-lapack) is the [reference implementation](http://www.netlib.org/blas/) while [mingw-w64-x86_64-openblas](https://packages.msys2.org/package/mingw-w64-x86_64-openblas) is the optimized [OpenBLAS](https://www.openblas.net/) implementation.

    OpenBLAS is recommended as it gives much better performance and can be installed using `pacman` on the **MSYS2 MSYS** shell by

    ```bash
    pacman -S mingw-w64-x86_64-openblas
    ```

    The best way to verify installation is to compile a very basic Hello World program with `g++ hello.c -lopenblas` (or `g++ hello.c -lblas -llapack` for the default, i.e. reference BLAS).

2. A **FORTRAN compiler**.
    It will be required to build [MUMPS]({{page.mumps_url}}) which is written in Fortran.

    On MSYS2, `gfortran` is installed along with the other base packages when MinGW is installed.


### Minor fixes

The `sdpa` buildsystem will download [MUMPS]({{page.mumps_url}}) and build it. A `Makefile` has been provided inside the `mumps` subfolder of `sdpa` source. Depending on the version of `gfortran` that you use, you may or may not need the `-funroll-all-loops` flag provided in this file. 

Older versions of `gfortran` will not need this flag and do not recognize it. If while running `make`, you run into a compiler error for `gfortran`, remove this flag.

### Build

At this point, you need to switch to the **MSYS2 MinGW x64** shell in the Start Menu.

`cd` to the directory containing `sdpa`. Assuming you created an `sdpa` subfolder in `C:` drive, do

```bash
cd /c/sdpa/sdpa-{{page.sdpa_latest_version}}/
./configure
```

By default, the configure script will link against the [reference BLAS](http://www.netlib.org/blas/). If you want to use an alternate BLAS/LAPACK, you can override the flags `--with-blas` and `--with-lapack`.

For example, if you want to use OpenBLAS, this should become

```bash
./configure --with-blas="-lopenblas" --with-lapack="-lopenblas"
```

`libopenblas` provides both BLAS and LAPACK.

This should check whether all requirements are met for the build. Once done, issue the make command:

```bash
make
```

This will generate the `sdpa` binary as well as `libsdpa.a` static library. The `libsdpa.a` library is what the Python wrapper will link against.

### Test run the SDPA binary

Let's do a test run on the `sdpa` binary to make sure all is well until this point.

**Note 1**: If you are not doing this on the **MSYS2 MinGW x64** shell (and instead doing it on some other shell, e.g. `PowerShell`), you'll need to add the MinGW `bin` folder to your path. It contains the dynamic libraries that `sdpa` binary will require. If you used the default options while installing MSYS2, the `bin` folder in question is `C:\msys64\mingw64\bin`. Add it to your `PATH` variable.

**Note 2**: On Windows, the line endings are `CRLF` while the examples provided with `sdpa` use UNIX line endings (i.e. just `LF`). Open `example1.dat` in a code editor (e.g. Visual Studio Code) and change the line endings to `CRLF`. Otherwise, the problem data will not be read correctly.

Then while still within the `sdpa-{{page.sdpa_latest_version}}` folder, do

```bash
./sdpa example1.dat example1.out
```

This should complete a test run the `sdpa` binary using one of the provided example files. If you can see the solver output, and `pdOPT` as the status, the build was successful. We can now move onto building the Python wrapper.

## Obtaining and installing SDPA Python wrapper

Once you have built and done a test run on `sdpa`, you have a working SDPA binary, it's finally time to build and install `sdpa-python`.

We need SPOOLES for building `sdpa-python`.

### Obtaining SPOOLES headers

On MSYS2, `libspooles` can be installed (from the **MSYS2 MSYS** shell) using

```bash
pacman -S mingw-w64-x86_64-spooles
```

The best way to verify installation is to compile a very basic Hello World program with `g++ hello.c -lspooles`.

You will however, still need to download it because SPOOLES headers are imported by `sdpa-python` (and you will need to provide the include path for it in `setupcfg.py`). It can be obtained from the official [SPOOLES webpage](http://www.netlib.org/linalg/spooles/spooles.2.2.html) or the [Debian package sources](http://ftp.de.debian.org/debian/pool/main/s/spooles/spooles_2.2.orig.tar.gz).

```bash
wget http://www.netlib.org/linalg/spooles/spooles.2.2.tgz
mkdir spooles
tar -zxf spooles.2.2.tgz -C spooles # (OR spooles_2.2.orig.tar.gz if from Debian Sources)
```

**Before proceeding to the next section, we now need to switch to whatever shell has our desired Python (e.g. Anaconda shell, PowerShell, etc).**

We will however still need MinGW on that shell, so you will need to add the folder containing MinGW binaries to the PATH in that shell (e.g. `C:\msys64\mingw64\bin` if you installed MSYS with default options).

MinGW also comes with a `python` binary, so make sure to add it to the end of the `PATH` variable so it does not override your desired Python in that shell.

For Command Prompt or the Anaconda Shell:

```bat
set PATH=%PATH%;C:\msys64\mingw64\bin
```

For PowerShell or Anaconda PowerShell:

```bash
$env:path = $env:path + ';C:\msys64\mingw64\bin'
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

We need to edit it and provide

1. The link to the `make.inc` in the `sdpa` folder (so it can find `libsdpa.a`). Assuming you extracted it in `C:\sdpa`:

    ```python
    SDPA_DIR =  os.path.join("C:\\", "sdpa", "sdpa-{{page.sdpa_latest_version}}")
    ```

2. The location of SPOOLES library and headers. Assuming you extracted it in `C:\sdpa`:

    ```python
    SPOOLES_INCLUDE =  os.path.join("C:\\", "sdpa", "spooles")
    ```

3. The location of MinGW static libraries. Assuming it was installed with the default options:

    ```python
    MINGW_LIBS    =  os.path.join("C:\\", "msys64", "mingw64", "lib")
    ```

4. Names of BLAS/LAPACK if you did not use the reference BLAS. Please use the same BLAS/LAPACK that you used while building `sdpa`. Assuming you used OpenBLAS:

    ```python
    LAPACK_NAME = 'openblas'
    BLAS_NAME = 'openblas'
    ```

### Build `sdpa-python` Package

```bash
python setup.py build -c mingw32
```

### Install `sdpa-python` Package

Finally, build and install `sdpa-python` package. While still in the `sdpa-python` directory, do:

```bash
python setup.py install
```

Assuming all goes well, you should be able to do `import sdpap` in Python. If you run into problems, `cd` out of the `sdpa-python` directory. Having folders with the name `sdpap` in your current directory, you will run into problems while doing `import sdpap` in Python (it will complain about a circular dependency problem).

## Cleanup

Once installed, you can delete both the `sdpa` and `sdpa-python` folders.
