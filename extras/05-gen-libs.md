# General Purpose Libraries

These are low-level libaries used by other libraries and tools.

## OpenMPI

Reference implementation of the Open Message Passing Interface.

Fetch:

```sh
curl https://download.open-mpi.org/release/open-mpi/v4.1/openmpi-4.1.1.tar.gz -o openmpi-4.1.1.tar.gz &&
tar xzvf openmpi-4.1.1.tar.gz
```

Build:

```sh
cd openmpi-4.1.1

./configure --prefix=/usr                         \
            --docdir=/usr/share/doc/openmpi-4.1.1 \
            --enable-mpi-cxx
make
```

Test:

```sh
make check
```

Install:

```sh
sudo make install
```

## Boost

Many, many C++ libraries. Is an optional dependency of some other libraries and may result in better performance.

Fetch:

```sh
curl https://boostorg.jfrog.io/artifactory/main/release/1.76.0/source/boost_1_76_0.tar.gz -o boost_1_76_0.tar.gz &&
tar xzvf boost_1_76_0.tar.gz
```

Build:

```sh
./bootstrap.sh --prefix=/usr
./b2 stage -j12 threading=multi link=shared
```

Test:

```sh
pushd tools/build/test; python test_all.py; popd
```

Install:

```sh
sudo ./b2 install threading=multi link=shared
```

## fftw

Fetch:

```sh
curl http://www.fftw.org/fftw-3.3.9.tar.gz -o fftw-3.3.9.tar.gz &&
tar xzvf fftw-3.3.9.tar.gz
```

`fftw` builds libraries for single-precision, double-precision, and long double-precision (32-bit, 64-bit, and 128-bit) floating-point numbers, each separately.

Build double-precision:

```sh
cd fftw-3.3.9

./configure --prefix=/usr    \
            --enable-shared  \
            --disable-static \
            --enable-threads \
            --enable-sse2    \
            --enable-avx
make
```

Test double-precision:

```sh
make check
```

Install double-precision:

```sh
sudo make install
```

Build single-precision:

```sh
make clean &&
./configure --prefix=/usr    \
            --enable-shared  \
            --disable-static \
            --enable-threads \
            --enable-sse2    \
            --enable-avx     \
            --enable-float
make
```

Test single-precision:

```sh
make check
```

Install single-precision:

```sh
sudo make install
```

Build long double-precision:

```sh
make clean &&
./configure --prefix=/usr    \
            --enable-shared  \
            --disable-static \
            --enable-threads \
            --enable-long-double
make
```

Test long double-precision:

```sh
make check
```

Install long double-precision:

```sh
sudo make install
```

## ATLAS and LAPACK

Automatically Tuned Linear Algebra Subsystem.

Fetch:

```sh
curl https://sourceforge.net/projects/math-atlas/files/latest/download -o atlas-3.10.3.tar.bz2 &&
curl https://github.com/Reference-LAPACK/lapack/archive/refs/tags/v3.9.1.tar.gz -o lapack-3.9.1.tar.gz  &&
mkdir atlas-3.10.3       &&
tar xvf atlas-3.10.3.tar.bz2 -C atlas-3.10.3 --strip-components=1
```

Build:

`--nof77` turns off the `Fortran` bindings, which will not build without a `Fortran` compiler. To include them, rebuild `gcc` with support for `Fortran`.

Note that `ATLAS` configure may fail if you have CPU throttling turned on, as it prevents being able to tune the build. Read the INSTALL.txt file in the source tree for instructions on how to disable this.

I have not yet found a way to get `ATLAS` to build successfully inside of a VM, but it is less likely to be useful there anyway.

```sh
cd atlas-3.10.3

mkdir build &&
cd    build &&
../configure --prefix=/usr \
             --nof77       \
             --shared      \
             --with-netlib-lapack-tarfile=../../lapack-3.9.1.tar.gz
make
```

Test:

```sh
make check &&
make ptcheck
```

Install:

```sh
sudo make install
```
