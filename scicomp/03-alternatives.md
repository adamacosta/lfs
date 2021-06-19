# Alternative scientific and statistical computing toolchains

## GNU Octave and dependencies

### FLTK

A cross-platform `C++` GUI toolkit.

```sh
wget https://www.fltk.org/pub/fltk/1.3.6/fltk-1.3.6-source.tar.bz2 -P /sources &&

tar xvf /sources/fltk-1.3.6-source.tar.bz2 &&
cd       fltk-1.3.6                        &&

./configure --prefix=/usr   \
            --enable-shared \
            --enable-cairo &&

make              &&
sudo make install &&

sudo mv /usr/share/doc/fltk /usr/share/doc/fltk-1.3.6 &&

cd .. &&
rm -rf fltk-1.3.6
```

### GLPK

The `GNU` Linear Programming Kit.

```sh
wget https://ftp.gnu.org/gnu/glpk/glpk-5.0.tar.gz -P /sources &&

tar xzvf /sources/glpk-5.0.tar.gz &&
cd        glpk-5.0                &&

./configure --prefix=/usr   \
            --enable-dl     \
            --enable-shared \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf glpk-5.0
```

### gnuplot

A portable command-line driven graphing utility.

```sh
wget https://sourceforge.net/projects/gnuplot/files/gnuplot/5.4.2/gnuplot-5.4.2.tar.gz/download -O /sources/gnuplot-5.4.2.tar.gz &&

tar xzvf /sources/gnuplot-5.4.2.tar.gz &&
cd        gnuplot-5.4.2                &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf gnuplot-5.4.2
```

### GraphicsMagick++

The swiss army knife of image processing.

```sh
wget https://sourceforge.net/projects/graphicsmagick/files/graphicsmagick/1.3.36/GraphicsMagick-1.3.36.tar.xz/download -O /sources/GraphicsMagick-1.3.36.tar.xz &&

tar xvf /sources/GraphicsMagick-1.3.36.tar.xz &&
cd       GraphicsMagick-1.3.36                &&

./configure --prefix=/usr          \
            --enable-shared        \
            --disable-static       \
            --enable-magick-compat \
            --with-x &&

make              &&
sudo make install &&

cd .. &&
rm -rf GraphicsMagick-1.3.6
```

### Qhull

Qhull computes the convex hull, Delaunay triangulation, Voronoi diagram, halfspace intersection about a point, furthest-site Delaunay triangulation, and furthest-site Voronoi diagram.

```sh
wget http://www.qhull.org/download/qhull-2020-src-8.0.2.tgz -P /sources &&

tar xzvf /sources/qhull-2020-src-8.0.2.tgz &&
cd        qhull-2020.2                     &&

cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr                   \
      -DDOC_INSTALL_DIR=/usr/share/doc/qhull-2020.2 \
      -DCMAKE_BUILD_TYPE=Release                    \
      -DBUILD_STATIC_LIBS=OFF                       \
      -DBUILD_SHARED_LIBS=ON .. &&

make              &&
ctest             &&
sudo make install &&

cd .. &&
rm -rf qhull-2020.2
```

### QRUPDATE

This is a `Fortran` library and requires that you build `gcc` with support. Note that the `solib` target forces the compiling of shared libraries, but it will still compile and install static libraries as well, in line with `Fortran` conventions.

```sh
wget https://sourceforge.net/projects/qrupdate/files/qrupdate/1.2/qrupdate-1.1.2.tar.gz/download -O /sources/qrupdate-1.1.2.tar.gz &&

tar xzvf /sources/qrupdate-1.1.2.tar.gz &&
cd        qrupdate-1.1.2                &&

make BLAS=-lopenblas solib    &&
sudo make PREFIX=/usr install &&

cd .. &&
rm -rf qrupdate-1.1.2
```

### SuiteSparse

A suite of sparse matrix algorithms. Note that the documentation warns about severe performance degradation using `OpenBLAS` and recommends `MKL` as the `BLAS` implementation, so if you have an Intel processor, you should do that, but I have an AMD.

```sh
wget https://github.com/DrTimothyAldenDavis/SuiteSparse/archive/refs/tags/v5.10.1.tar.gz -O /sources/SuiteSparse-5.10.1.tar.gz &&

tar xzvf /sources/SuiteSparse-5.10.1.tar.gz &&
cd        SuiteSparse-5.10.1                &&

make BLAS=-lopenblas library                   &&
sudo make BLAS=-lopenblas INSTALL=/usr install &&

cd .. &&
rm -rf SuiteSparse-5.10.1
```

### GNU Octave

The `GNU` free software clone of `MATLAB`.

```sh
wget https://ftpmirror.gnu.org/octave/octave-6.2.0.tar.gz -P /sources &&

tar xzvf /sources/octave-6.2.0.tar.gz &&
cd        octave-6.2.0                &&

mkdir build &&
cd    build &&

../configure --prefix=/usr        \
             --with-blas=openblas \
             --with-linux-crypto  \
             --with-openssl=yes   \
             --with-x &&

make              &&
make -k check     &&
sudo make install &&

cd ../.. &&
rm -rf octave-6.2.0
```

## R

`GNU` project implementing a free-software version of the `S-PLUS` statistical programming language, particularly strong at rendering static graphics and reports with mathematical typesetting.

`--enable-R-shlib` creates `libR`, which is only needed if you want to create `C` extensions that link against it.

If you choose to install `TeXLive` or or some other `LaTeX` implementation later, you may want to rebuild `R` as it will enable generating pdf versions of the language manuals. Presently, not having `texi2dvi` somewhere on your PATH will cause one test to fail when it tries to test the built-in function `tools::texi2pdf()`.

```sh
wget https://cloud.r-project.org/src/base/R-4/R-4.1.0.tar.gz -P /sources &&

tar xzvf /sources/R-4.1.0.tar.gz &&
cd        R-4.1.0                &&

./configure --prefix=/usr        \
            --enable-lto=R       \
            --enable-R-shlib     \
            --with-blas=openblas \
            --with-lapack        \
            --with-x             \
            --docdir=/usr/share/doc/R-4.1.0 &&

make              &&
make -k check     &&
sudo make install &&

cd .. &&
rm -rf R-4.1.0
```

## Julia and dependencies

`Julia` vendors its own forks of `libuv` and `LLVM`. Most of the other dependencies would already be on the system if you've gotten this far and installed everything required by the `Python` stack and `Octave`. The exceptions will be built separately first.

### OpenLibm

Replacement for `libm`.

```sh
wget https://github.com/JuliaMath/openlibm/archive/refs/tags/v0.7.5.tar.gz -O /sources/openlibm-0.7.5.tar.gz &&

tar xzvf /sources/openlibm-0.7.5.tar.gz &&
cd        openlibm-0.7.5                &&

make MARCH=native             &&
sudo make prefix=/usr install &&

cd .. &&
rm -rf openlibm-0.7.5
```

### DSFMT

Double precision SIMD-oriented Fast Mersenne Twister for pseudorandom number generation.

```sh
wget https://github.com/MersenneTwister-Lab/dSFMT/archive/refs/tags/v2.2.5.tar.gz -O /sources/dSFMT-2.2.5.tar.gz &&

tar xzvf /sources/dSFMT-2.2.5.tar.gz &&
cd        dSFMT-2.2.5                &&

gcc -DNDEBUG -DDSFMT_MEXP=19937 -fPIC -DDSFMT_DO_NOT_USE_OLD_NAMES -DDSFMT_SHLIB \
	-O3 -finline-functions -fomit-frame-pointer -fno-strict-aliasing \
	--param max-inline-insns-single=1800 -Wall  -std=c99 -shared     \
	-msse2 -march=native -DHAVE_SSE2 dSFMT.c -P libdSFMT.so &&

sudo install -vDm644 dSFMT.h /usr/include &&
sudo install -vDm755 libdSFMT.so /usr/lib &&

cd .. &&
rm -rf dSFMT-2.2.5
```

### libgit2

```sh
wget https://github.com/libgit2/libgit2/releases/download/v1.1.0/libgit2-1.1.0.tar.gz -P /sources &&

tar xzvf /sources/libgit2-1.1.0.tar.gz &&
cd        libgit2-1.1.0                &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr \
      -DCMAKE_BUILD_TYPE=Release .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf libgit2-1.1.0
```

### mbedtls

`C` library providing cryptographic primitives with a very small footprint, intended especially for embedded systems.

Be forewarned that I had to turn off `Werror` to get this to compile, as it would otherwise fail on `Wstringop-overflow`, `Wformat-overflow`, and `Wformat-truncation`. Given the compiler is saying this code is vulnerable to overflow, combined with the patch borrowed from the `Julia` build files force-enabling the broken `MD4`, I would say it is not a good idea to use `Julia` in any security-critical software.

```sh
wget https://github.com/ARMmbed/mbedtls/archive/refs/tags/v2.26.0.tar.gz -O /sources/mbedtls-2.26.0.tar.gz &&

tar xzvf /sources/mbedtls-2.26.0.tar.gz &&
cd        mbedtls-2.26.0                &&

sed -i.org "s|//#define MBEDTLS_MD4_C|#define MBEDTLS_MD4_C|" include/mbedtls/config.h &&
sed -i.org "s/-Werror//" CMakeLists.txt                                                &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr      \
      -DCMAKE_BUILD_TYPE=Release       \
      -DENABLE_PROGRAMS=Off            \
      -DENABLE_TESTING=Off             \
      -DUSE_SHARED_MBEDTLS_LIBRARY=On  \
      -DUSE_STATIC_MBEDTLS_LIBRARY=Off \
      .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf mbedtls-2.26.0
```

### utf8proc

UTF-8 string processing library. This build only creates a static library because that is what the `Julia` build itself does.

```sh
wget https://github.com/JuliaStrings/utf8proc/archive/v2.6.1.tar.gz -O /sources/uft8proc-2.6.1.tar.gz &&

tar xzvf /sources/uft8proc-2.6.1.tar.gz &&
cd        utf8proc-2.6.1                &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr \
      -DCMAKE_BUILD_TYPE=Release  \
      .. &&

make              &&
sudo make install &&

cd .. &&
rm -rf utf8proc-2.6.1
```

### P7Zip

Note that `Julia` also depends upon the POSIX port of the Windows `7-Zip` tool, but as this project is four major versions behind the Windows original and hasn't been updated in five years, and `Julia` provides several vendored patches to address multiple CVEs, I am not installing this directly and will use the vendored version.

### Julia

A garbage-collected, dynamically-typed language specialized for numerical, scientific, and statistical computing, aiming to provide performance approaching that of statically-compiled native code via JIT-compiling and caching. `Julia` tries to solve the so-called "two-language" problem where researchers tend to run experiments and prototype systems using high-level languages like `MATLAB`, `R`, and `Python`, and then port performance-critical or possibly all of the code to lower-level `C++` and `Fortran` for production use.

```sh
wget https://github.com/JuliaLang/julia/releases/download/v1.6.1/julia-1.6.1.tar.gz -P /sources && 

tar xzvf /sources/julia-1.6.1.tar.gz &&
cd        julia-1.6.1                &&

cat > Make.user <<"EOF"
USE_SYSTEM_OPENLIBM=1
USE_SYSTEM_DSFMT=1
USE_SYSTEM_PCRE=1
USE_SYSTEM_BLAS=1
USE_SYSTEM_LAPACK=1
USE_SYSTEM_GMP=1
USE_SYSTEM_LIBGIT2=1
USE_SYSTEM_MBEDTLS=1
USE_SYSTEM_LIBSSH2=1
USE_SYSTEM_NGHTTP2=1
USE_SYSTEM_CURL=1
USE_SYSTEM_MPFR=1
USE_SYSTEM_LIBSUITESPARSE=1
USE_SYSTEM_UTF8PROC=1
USE_SYSTEM_ZLIB=1
MARCH=native
EOF

make prefix=/usr sysconfdir=/etc              &&
sudo make prefix=/usr sysconfdir=/etc install &&

cd .. &&
rm -rf julia-1.6.1
```


















