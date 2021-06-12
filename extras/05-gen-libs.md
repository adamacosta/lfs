# General Purpose Libraries

These are low-level libaries used by other libraries and tools.

## libseccomp

Userspace interface to the Linux kernel seccomp functions.

```sh
curl https://github.com/seccomp/libseccomp/releases/download/v2.5.1/libseccomp-2.5.1.tar.gz -o /sources/libseccomp-2.5.1.tar.gz &&

tar xzvf /sources/libseccomp-2.5.1.tar.gz &&
cd        libseccomp-2.5.1                &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libseccomp-2.5.1
```

## liboauth

```sh
curl https://downloads.sourceforge.net/liboauth/liboauth-1.0.3.tar.gz -o /sources/liboauth-1.0.3.tar.gz &&
curl https://www.linuxfromscratch.org/patches/blfs/svn/liboauth-1.0.3-openssl-1.1.0-3.patch -o /sources/liboauth-1.0.3-openssl-1.1.0-3.patch &&

tar xzvf /sources/liboauth-1.0.3.tar.gz &&
cd        liboauth-1.0.3                &&

patch -Np1 -i /sources/liboauth-1.0.3-openssl-1.1.0-3.patch &&

./configure --prefix=/usr \
            --disable-static &&

make                                                 &&
make dox                                             &&
sudo make install                                    &&
sudo install -v -dm755 /usr/share/doc/liboauth-1.0.3 &&
sudo cp -rv doc/html/* /usr/share/doc/liboauth-1.0.3 &&

cd .. &&
rm -rf liboauth-1.0.3
```

## icu

```sh
curl https://github.com/unicode-org/icu/releases/download/release-69-1/icu4c-69_1-src.tgz -o /sources/icu4c-69_1-src.tgz &&

tar xzvf /sources/icu4c-69_1-src.tgz &&
cd        icu/source                 &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf icu
```

## libxml2

```sh
curl http://xmlsoft.org/sources/libxml2-2.9.10.tar.gz -o /sources/libxml2-2.9.10.tar.gz &&
curl http://www.linuxfromscratch.org/patches/blfs/10.1/libxml2-2.9.10-security_fixes-1.patch -o /sources/libxml2-2.9.10-security_fixes-1.patch &&

tar xzvf /sources/libxml2-2.9.10.tar.gz &&
cd        libxml2-2.9.10                &&

patch -p1 -i /sources/libxml2-2.9.10-security_fixes-1.patch   &&
sed -i '/if Py/{s/Py/(Py/;s/)/))/}' python/{types.c,libxml.c} &&
sed -i 's/test.test/#&/' python/tests/tstLastError.py         &&
sed -i 's/ TRUE/ true/' encoding.c                            &&

./configure --prefix=/usr    \
            --disable-static \
            --with-history &&

make              &&
sudo make install &&

cd .. &&
rm -rf libxml2-2.9.10
```

## JSON-C

```sh
curl https://s3.amazonaws.com/json-c_releases/releases/json-c-0.15.tar.gz -o /sources/json-c-0.15.tar.gz &&

tar xzvf /sources/json-c-0.15.tar.gz &&
cd        json-c-0.15                &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr \
      -DCMAKE_BUILD_TYPE=Release \
      -DBUILD_STATIC_LIBS=OFF    \
      .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf json-c-0.15
```

## xerces-c

Validating xml parser for C++.

```sh
curl https://mirrors.sonic.net/apache//xerces/c/3/sources/xerces-c-3.2.3.tar.xz -o /sources/xerces-c-3.2.3.tar.xz &&

tar xvf /sources/xerces-c-3.2.3.tar.xz &&
cd       xerces-c-3.2.3                &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr                          \
      -DCMAKE_INSTALL_DOCDIR=/usr/share/doc/xerces-c-3.2.3 \
      -DCMAKE_BUILD_TYPE=Release                           \
      -DBUILD_STATIC_LIBS=OFF                              \
      -Dnetwork-accessor=curl                              \
      -Dtranscoder=icu                                     \
      -Dmutex-manager=posix                                \
      .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf xerces-c-3.2.3
```

## libyaml

```sh
curl https://github.com/yaml/libyaml/archive/0.2.5/libyaml-0.2.5.tar.gz -o /sources/libyaml-0.2.5.tar.gz &&

tar xzvf /sources/libyaml-0.2.5.tar.gz &&
cd        libyaml-0.2.5                &&

./bootstrap                  &&
./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libyaml-0.2.5
```

## hdf5

Hierarchical data format.

```sh
curl "https://www.hdfgroup.org/package/cmake-hdf5-1-12-0-tar-gz/?wpdmdl=14580&refresh=60c269659fecd1623353701" -o /sources/hdf5-1.12.0.tar.gz &&

tar xzvf /sources/hdf5-1.12.0.tar.gz    &&
cd        CMake-hdf5-1.12.0/hdf5-1.12.0 &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr \
      -DCMAKE_BUILD_TYPE=Release  \
      -DBUILD_STATIC_LIBS=OFF     \
      .. &&

make              &&
sudo make install &&

cd ../../.. &&
rm -rf CMake-hdf5-1.12.0
```

## OpenMPI

Reference implementation of the Open Message Passing Interface.

```sh
curl https://download.open-mpi.org/release/open-mpi/v4.1/openmpi-4.1.1.tar.gz -o /sources/openmpi-4.1.1.tar.gz &&

tar xzvf /sources/openmpi-4.1.1.tar.gz &&
cd        openmpi-4.1.1                &&

./configure --prefix=/usr                         \
            --docdir=/usr/share/doc/openmpi-4.1.1 \
            --enable-mpi-cxx &&

make              &&
make check        &&
sudo make install &&

cd .. &&
rm -rf openmpi-4.1.1
```

## hwloc

Libraries to aid HPC applications that use OpenMPI to discover hardware locality features.

```sh
curl https://download.open-mpi.org/release/hwloc/v2.4/hwloc-2.4.1.tar.gz -o /sources/hwloc-2.4.1.tar.gz &&

tar xzvf /sources/hwloc-2.4.1.tar.gz &&
cd        hwloc-2.4.1                &&

./configure --prefix=/usr \
            --docdir=/usr/share/doc/hwloc-2.4.1 &&

make              &&
sudo make install &&

cd .. &&
rm -rf hwloc-2.4.1
```

## Boost

Many, many C++ libraries. Is an optional dependency of some other libraries and may result in better performance.

```sh
curl https://boostorg.jfrog.io/artifactory/main/release/1.76.0/source/boost_1_76_0.tar.gz -o /sources/boost_1_76_0.tar.gz &&

tar xzvf /sources/boost_1_76_0.tar.gz &&
cd        boost_1_76_0                &&

./bootstrap.sh --prefix=/usr                &&
./b2 stage -j12 threading=multi link=shared &&

pushd tools/build/test; python test_all.py; popd &&

sudo ./b2 install threading=multi link=shared &&

cd .. &&
sudo rm -rf boost_1_76_0
```

## TBB

Threading building blocks for C++.

```sh
curl https://github.com/oneapi-src/oneTBB/archive/refs/tags/v2021.2.0.tar.gz -o /sources/tbb-2021.2.0.tgz &&

tar xzvf /sources/tbb-2021.2.0.tgz &&
cd        oneTBB-2021.2.0          &&

mkdir build &&
cd    build &&

CFLAGS="-mtune=generic -march=x86-64"   \
CXXFLAGS="-mtune=generic -march=x86-64" \
cmake -DCMAKE_INSTALL_PREFIX=/usr       \
      -DBUILD_SHARED_LIBS=on            \
      -DCMAKE_BUILD_TYPE=Release        \
      .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf oneTBB-2021.2.0
```

## GLib

```sh
curl https://download.gnome.org/sources/glib/2.68/glib-2.68.3.tar.xz -o /sources/glib-2.68.3.tar.xz &&
curl https://www.linuxfromscratch.org/patches/blfs/svn/glib-2.68.3-skip_warnings-1.patch -o /sources/glib-2.68.3-skip_warnings-1.patch &&

tar xvf /sources/glib-2.68.3.tar.xz &&
cd       glib-2.68.3                &&

patch -Np1 -i /sources/glib-2.68.3-skip_warnings-1.patch &&

mkdir build &&
cd    build &&
meson --prefix=/usr      \
      -Dman=true         \
      -Dselinux=disabled \
      ..    &&

ninja                                                                           &&
sudo ninja install                                                              &&
sudo mkdir -p /usr/share/doc/glib-2.68.3                                        &&
sudo cp -r ../docs/reference/{NEWS,gio,glib,gobject} /usr/share/doc/glib-2.68.3 &&

cd ../.. &&
rm -rf glib-2.68.3
```

## libpsl

Library for accessing and resolving information from the Public Suffix List.

```sh
curl https://github.com/rockdaboot/libpsl/releases/download/0.21.1/libpsl-0.21.1.tar.gz -o /sources/libpsl-0.21.1.tar.gz &&

tar xzvf /sources/libpsl-0.21.1.tar.gz &&
cd        libpsl-0.21.1                &&

sed -i 's/env python/&3/' src/psl-make-dafsa &&
./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libpsl-0.21.1
```

## libsoup

```sh
curl https://download.gnome.org/sources/libsoup/2.72/libsoup-2.72.0.tar.xz -o /sources/libsoup-2.72.0.tar.xz &&

tar xvf /sources/libsoup-2.72.0.tar.xz &&
cd       libsoup-2.72.0                &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Dvapi=enabled      \
      -Dgssapi=disabled .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libsoup-2.72.0
```
