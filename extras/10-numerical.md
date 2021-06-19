# Numerical and geospatial libraries

Many packages listed here are `Fortran` libraries, so require that you build `gcc` with support for the `Fortran` language first.

## ATLAS

Automatically Tuned Linear Algebra Subsystem.

Note that, in my example here, I am using the latest developer version. The `ATLAS` documentation claims this is up to twice as fast as the last stable release from 2016, but of course do so at your own risk. You are probably not building Linux From Scratch for a production environment where numerical accuracy is absolutely critical, so have fun.

```sh
wget https://sourceforge.net/projects/math-atlas/files/Developer%20%28unstable%29/3.11.41/atlas3.11.41.tar.bz2/download -O /sources/atlas-3.11.41.tar.bz2 &&

mkdir atlas-3.11.41 &&
tar xvf /sources/atlas-3.11.41.tar.bz2 -C atlas-3.11.41 --strip-components=1
```

`ATLAS` configure will fail if you have CPU throttling turned on, as it prevents being able to tune the build. Read the INSTALL.txt file in the source tree for instructions on how to disable this. Please read the [ATLAS installation guide](http://math-atlas.sourceforge.net/atlas_install/atlas_install.html) and [errata](http://math-atlas.sourceforge.net/errata.html) for tips and gotchas. You will most likely need to set the CPU governor to `performance`, enable `on demand`, and disable thermal throttling. This should only be done on desktop or server systems, but it doesn't make much sense to be attempting to run high performance numerical code on a laptop anyway. Ensure you have a quality build with good cooling. Water is pumping if liquid cooled, airflow is good if air cooled, etc.

`ATLAS` automatically sets `make` to run in parallel mode in sections where it is safe to do so, which means you must run `make` in single-threaded mode by default. If you have set `MAKEFLAGS`, ensure you unset it before building.

```sh
unset MAKEFLAGS
```

Now run the configure step. `--nof77` turns off the `Fortran` bindings, which will not build without a `Fortran` compiler. To include them, rebuild `gcc` with support for `Fortran`.

```sh
cd atlas-3.11.41 &&

mkdir build &&
cd    build &&
../configure --prefix=/usr \
             --nof77       \
             --shared
```

To compile with `Fortran` bindings:

```sh
cd atlas-3.11.41 &&

mkdir build &&
cd    build &&
../configure --prefix=/usr \
             --shared      \
             --with-netlib-lapack-tarfile=/sources/lapack-3.9.1.tar.gz
```

You'll want to inspect the output from `configure` being proceeding to ensure it meets what you would expect. There are tips in the complete installation guide on how to ensure the correct settings are discovered and how to configure them manually if they are not. Once you are satisifed, proceed with building, sanity testing, and installation.

```sh
make              &&
make -k check     &&
make -k ptcheck   &&
sudo make install &&

cd ../.. &&
rm -rf atlas-3.11.41
```

`ATLAS` may take an extremely long time to build. The website warns it may take days on some systems. This is not likely to be the case on a modern multicore computer, but due to the fact that the build process involves running diagnostic tests to find the best compute kernels for your specific hardware, this may take a long time even on very fast machines, as the number of iterations and tests it needs to run to come to a decision cannot be determined in advance.

It is important that not much else is happening on the machine while `ATLAS` builds, in order to get accurate and reproducible timings. It will, of course, also take longer because we are building `LAPACK` in place rather than using a pre-installed version.

## OpenBLAS

This is an alternative optimized `BLAS` for environments where `ATLAS` is not feasible or will not work.

Note that for both `ATLAS` and `OpenBLAS` we do not disable static libraries, in spite of what Linux From Scratch generally does for system libs. That is because `BLAS` libraries have traditionally been statically linked into executables that use them and shared library support is still considered experimental for most implementations.

```sh
wget https://github.com/xianyi/OpenBLAS/releases/download/v0.3.15/OpenBLAS-0.3.15.tar.gz -P /sources &&

tar xzvf /sources/OpenBLAS-0.3.15.tar.gz &&
cd        OpenBLAS-0.3.15                &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr \
      -DCMAKE_BUILD_TYPE=Release  \
      -DBUILD_SHARED_LIBS=ON      \
      .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf OpenBLAS-0.3.15
```

## LAPACK

Linear Algebra Package.

```sh
wget https://github.com/Reference-LAPACK/lapack/archive/refs/tags/v3.9.1.tar.gz -P /sources &&

tar xzvf /sources/lapack-3.9.1.tar.gz &&
cd        lapack-3.9.1                &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr \
      -DCMAKE_BUILD_TYPE=Release  \
      -DBUILD_SHARED_LIBS=ON      \
      -DUSE_OPTIMIZED_BLAS=ON     \
      .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf lapack-3.9.1
```

## ARPACK-NG

`Fortran` libraries for solving sparse matrices. Modern replacement for the original `ARPACK`, which has not been maintained for 20 years.

```sh
wget https://github.com/opencollab/arpack-ng/archive/refs/tags/3.8.0.tar.gz -O /sources/arpack-ng-3.8.0.tar.gz &&

tar xzvf /sources/arpack-ng-3.8.0.tar.gz &&
cd        arpack-ng-3.8.0                &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr \
      -DBUILD_SHARED_LIBS=ON      \
      -DMPI=ON .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf arpack-ng-3.8.0
```

## SuperLU

`SuperLU` is a general purpose library for the direct solution of large, sparse, nonsymmetric systems of linear equations.

```sh
wget https://portal.nersc.gov/project/sparse/superlu/superlu_5.2.2.tar.gz -P /sources &&

tar xzvf /sources/superlu_5.2.2.tar.gz &&
cd        superlu-5.2.2                &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr   \
      -DBUILD_SHARED_LIBS=on        \
      -Denable_internal_blaslib=NO  \
      -DTPL_BLAS_LIBRARIES=openblas \
      -DCMAKE_BUILD_TYPE=Release .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf superlu-5.2.2
```

## Armadillo

Linear algebra libraries beyond the BLAS.

```sh
wget http://sourceforge.net/projects/arma/files/armadillo-10.5.1.tar.xz -P /sources &&

tar xvf /sources/armadillo-10.5.1.tar.xz &&
cd       armadillo-10.5.1                &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr \
      -DBUILD_SHARED_LIBS=ON      \
      -DCMAKE_BUILD_TYPE=Release .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf armadillo-10.5.1
```

## Eigen

C++ template library for matrices.

```sh
wget https://gitlab.com/libeigen/eigen/-/archive/3.3.9/eigen-3.3.9.tar.gz -P /sources &&

tar xzvf /sources/eigen-3.3.9.tar.gz &&
cd        eigen-3.3.9                &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr .. &&

make doc          &&
make -k check     &&
sudo make install &&

cd ../.. &&
rm -rf eigen-3.3.9
```

## PROJ

Library to convert between different coordinate systems.

```sh
wget https://download.osgeo.org/proj/proj-8.0.1.tar.gz -P /sources &&

tar xzvf /sources/proj-8.0.1.tar.gz &&
cd        proj-8.0.1                &&

./configure --prefix=/usr                      \
            --disable-static                   \
            --docdir=/usr/share/doc/proj-8.0.1 \
            --enable-lto &&

make              &&
make -k check     &&
sudo make install &&

cd .. &&
rm -rf proj-8.0.1
```

Now populate the system database, if you wish. You can of course only download data files you think you will need, as `--all` will download the entire world and take quite a while.

```sh
sudo projsync --system-directory --all
```

## GeoTIFF

Augmentation of the TIFF image format that includes geolocation metadata.

```sh
wget https://github.com/OSGeo/libgeotiff/releases/download/1.6.0/libgeotiff-1.6.0.tar.gz -P /sources &&

tar xzvf /sources/libgeotiff-1.6.0.tar.gz &&
cd        libgeotiff-1.6.0                &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libgeotiff-1.6.0
```

## GDAL

Geospatial data library.

```sh
wget https://github.com/OSGeo/gdal/releases/download/v3.3.0/gdal-3.3.0.tar.gz -P /sources &&

tar xzvf /sources/gdal-3.3.0.tar.gz &&
cd        gdal-3.3.0                &&

./configure --prefix=/usr    \
            --disable-static \
            --enable-lto &&

make              &&
sudo make install &&

cd .. &&
rm -rf gdal-3.3.0
```
