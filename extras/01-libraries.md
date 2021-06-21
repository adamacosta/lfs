# General Purpose Libraries

These are low-level libaries used by other libraries and tools.

## inih

.ini file parser.

```sh
wget https://github.com/benhoyt/inih/archive/r53/inih-r53.tar.gz -P /sources &&

tar xzvf /sources/inih-r53.tar.gz &&
cd        inih-r53                &&

mkdir build &&
cd    build &&

meson --prefix=/usr .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf inih-r53
```

## popt

Command-line option parser. If `doxygen` is not installed, remove the calls to build and install the API documentation.

```sh
wget http://ftp.rpm.org/popt/releases/popt-1.x/popt-1.18.tar.gz -P /sources &&

tar xzvf /sources/popt-1.18.tar.gz &&
cd        popt-1.18                &&

./configure --prefix=/usr \
            --disable-static &&

make                          &&
sed -i 's@\./@src/@' Doxyfile &&
doxygen                       &&
sudo make install             &&

sudo install -v -m755 -d /usr/share/doc/popt-1.18             &&
sudo install -v -m644 doxygen/html/* /usr/share/doc/popt-1.18 &&

cd .. &&
rm -rf popt-1.18
```

## mtdev

Interface to multitouch kernel events.

```sh
wget https://bitmath.org/code/mtdev/mtdev-1.1.6.tar.bz2 -P /sources &&

tar xvf /sources/mtdev-1.1.6.tar.bz2 &&
cd       mtdev-1.1.6                 &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf mtdev-1.1.6
```

## libseccomp

Userspace interface to the Linux kernel seccomp functions.

```sh
wget https://github.com/seccomp/libseccomp/releases/download/v2.5.1/libseccomp-2.5.1.tar.gz -P /sources &&

tar xzvf /sources/libseccomp-2.5.1.tar.gz &&
cd        libseccomp-2.5.1                &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libseccomp-2.5.1
```

## libpcap

Packet capture.

```sh
wget http://www.tcpdump.org/release/libpcap-1.10.0.tar.gz -P /sources &&

tar xzvf /sources/libpcap-1.10.0.tar.gz &&
cd        libpcap-1.10.0                &&

./configure --prefix=/usr &&

make                                                                  &&
sed -i '/INSTALL_DATA.*libpcap.a\|RANLIB.*libpcap.a/ s/^/#/' Makefile &&
sudo make install                                                     &&

cd .. &&
rm -rf libpcap-1.10.0
```

## libnl

```sh
wget https://github.com/thom311/libnl/releases/download/libnl3_5_0/libnl-3.5.0.tar.gz -P /sources &&
wget https://github.com/thom311/libnl/releases/download/libnl3_5_0/libnl-doc-3.5.0.tar.gz -P /sources &&

tar xzvf /sources/libnl-3.5.0.tar.gz &&
cd        libnl-3.5.0                &&

./configure --prefix=/usr     \
            --sysconfdir=/etc \
            --disable-static  &&

make              &&
make -k check     &&
sudo make install &&

sudo mkdir -vp /usr/share/doc/libnl-3.5.0 &&
sudo tar -xf /sources/libnl-doc-3.5.0.tar.gz --strip-components=1 --no-same-owner \
         -C  /usr/share/doc/libnl-3.5.0   &&

cd .. &&
rm -rf libnl-3.5.0
```

## liboauth

```sh
wget https://downloads.sourceforge.net/liboauth/liboauth-1.0.3.tar.gz -P /sources &&
wget https://www.linuxfromscratch.org/patches/blfs/svn/liboauth-1.0.3-openssl-1.1.0-3.patch -P /sources/patches &&

tar xzvf /sources/liboauth-1.0.3.tar.gz &&
cd        liboauth-1.0.3                &&

patch -Np1 -i /sources/patches/liboauth-1.0.3-openssl-1.1.0-3.patch &&

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

## libssh2

```sh
wget https://www.libssh2.org/download/libssh2-1.9.0.tar.gz -P /sources &&
wget https://www.linuxfromscratch.org/patches/blfs/svn/libssh2-1.9.0-security_fixes-1.patch -P /sources/patches &&

tar xzvf /sources/libssh2-1.9.0.tar.gz &&
cd        libssh2-1.9.0                &&

patch -Np1 -i /sources/patches/libssh2-1.9.0-security_fixes-1.patch &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libssh2-1.9.0
```

## libuv

Fast C event loop library.

```sh
wget https://dist.libuv.org/dist/v1.41.0/libuv-v1.41.0.tar.gz -P /sources &&

tar xzvf /sources/libuv-v1.41.0.tar.gz &&
cd        libuv-v1.41.0                &&

sh autogen.sh                &&
./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libuv-v1.41.0
```

## libevent

Provides a mechanism to execute a callback function when a specific event occurs on a file descriptor or after a timeout has been reached.

```sh
wget https://github.com/libevent/libevent/releases/download/release-2.1.12-stable/libevent-2.1.12-stable.tar.gz -P /sources &&

tar xzvf /sources/libevent-2.1.12-stable.tar.gz &&
cd        libevent-2.1.12-stable                &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libevent-2.1.12-stable
```

## c-ares

C library for asynchronous DNS requests. Used by `NodeJS`.

```sh
wget https://c-ares.haxx.se/download/c-ares-1.17.1.tar.gz -P /sources &&

tar xzvf /sources/c-ares-1.17.1.tar.gz &&
cd        c-ares-1.17.1                &&

mkdir build &&
cd    build &&

cmake  -DCMAKE_INSTALL_PREFIX=/usr .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf c-ares-1.17.1
```

## mozjs78

`polkit` and much of the `GNOME` desktop depends upon the extended support release 78 version of the Mozilla JavaScript engine, which has to be extracted from `Firefox-78esr` and installed separately.

This depends upon `rustc` being installed. Unfortunately, this also requires an older version of `autoconf`, `2.13`, so that will need to be installed first. We have to use the `--program-suffix` parameter to disambiguate.

```sh
wget https://ftp.gnu.org/gnu/autoconf/autoconf-2.13.tar.gz -P /sources &&
wget https://www.linuxfromscratch.org/patches/blfs/svn/autoconf-2.13-consolidated_fixes-1.patch -P /sources/patches &&

tar xzvf /sources/autoconf-2.13.tar.gz &&
cd        autoconf-2.13                &&

patch -Np1 -i /sources/patches/autoconf-2.13-consolidated_fixes-1.patch &&
mv -v autoconf.texi autoconf213.texi                                    &&
rm -v autoconf.info                                                     &&

./configure --prefix=/usr \
            --program-suffix=2.13 &&

make                                                          &&
sudo make install                                             &&
sudo install -v -m644 autoconf213.info /usr/share/info        &&
sudo install-info --info-dir=/usr/share/info autoconf213.info &&

cd .. &&
rm -rf autoconf-2.13
```

```sh
wget https://archive.mozilla.org/pub/firefox/releases/78.11.0esr/source/firefox-78.11.0esr.source.tar.xz -P /sources &&

tar xvf /sources/firefox-78.11.0esr.source.tar.xz &&
cd       firefox-78.11.0                          &&

mkdir obj &&
cd    obj &&

CC=gcc CXX=g++ \
../js/src/configure --prefix=/usr           \
                    --with-intl-api         \
                    --with-system-zlib      \
                    --with-system-icu       \
                    --disable-jemalloc      \
                    --disable-debug-symbols \
                    --enable-readline &&

make                                                &&
sudo make install                                   &&
sudo rm  -v /usr/lib/libjs_static.ajs               &&
sudo sed -i '/@NSPR_CFLAGS@/d' /usr/bin/js78-config &&

cd ../.. &&
rm -rf firefox-78.11.0
```

## libproxy

Optional `Firefox` dependency. Includes `Python` and `Perl` bindings.

```sh
wget https://github.com/libproxy/libproxy/archive/refs/tags/0.4.17.tar.gz -P /sources &&

tar xzvf /sources/libproxy-0.4.17.tar.gz &&
cd        libproxy-0.4.17                &&

cmake -DCMAKE_INSTALL_PREFIX=/usr .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf libproxy-0.4.17
```

## polkit

Toolkit for allowing unprivileged processes to communicate with privileged processes. Many Linux desktops use it as the backend for something similar to Windows UAC, allowing user-based privilege escalation without `sudo`.

Beware that versions of `polkit` `>= 0.113 && <= 0.118` are vulnerable to [CVE-2021-3560](https://access.redhat.com/security/cve/CVE-2021-3560), effectively allowing any account root access, so be sure you install only 0.119 or a later version if one is available.

`polkit` requires a system user to run the daemon:

```sh
sudo groupadd -fg 27 polkitd &&
sudo useradd  -c "PolicyKit Daemon Owner" -d /etc/polkit-1 -u 27 \
              -g polkitd -s /bin/false polkitd
```

To build and install:

```sh
wget https://www.freedesktop.org/software/polkit/releases/polkit-0.119.tar.gz -P /sources &&

tar xzvf /sources/polkit-0.119.tar.gz &&
cd        polkit-0.119                &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static     \
            --with-os-type=LFS &&

make              &&
sudo make install &&

sudo chown polkitd /etc/polkit-1/rules.d &&
sudo chmod 700     /etc/polkit-1/rules.d &&

cd .. &&
rm -rf polkit-0.119
```

`polkit` then requires `PAM` configuration:

```sh
sudo su -
cat > /etc/pam.d/polkit-1 << "EOF"
# Begin /etc/pam.d/polkit-1

auth     include        system-auth
account  include        system-account
password include        system-password
session  include        system-session

# End /etc/pam.d/polkit-1
EOF
exit
```

## icu

International Components for Unicode.

```sh
wget https://github.com/unicode-org/icu/releases/download/release-69-1/icu4c-69_1-src.tgz -P /sources &&

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
wget http://xmlsoft.org/sources/libxml2-2.9.10.tar.gz -P /sources &&
wget http://www.linuxfromscratch.org/patches/blfs/10.1/libxml2-2.9.10-security_fixes-1.patch -P /sources/patches &&

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
wget https://s3.amazonaws.com/json-c_releases/releases/json-c-0.15.tar.gz -P /sources &&

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
wget https://mirrors.sonic.net/apache//xerces/c/3/sources/xerces-c-3.2.3.tar.xz -P /sources &&

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
wget https://github.com/yaml/libyaml/archive/0.2.5/libyaml-0.2.5.tar.gz -P /sources &&

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

## liblinear

Library for classification via Support Vector Machines. Used by `nmap`.

```sh
curl -L https://github.com/cjlin1/liblinear/archive/v242/liblinear-242.tar.gz -P /sources &&

tar xzvf /sources/liblinear-242.tar.gz &&
cd        liblinear-242                &&

make lib                                          &&
sudo install -vm644 linear.h /usr/include         &&
sudo install -vm755 liblinear.so.4 /usr/lib       &&
sudo ln -sfv liblinear.so.4 /usr/lib/liblinear.so &&

cd .. &&
rm -rf liblinear-242
```

## OpenMPI

Reference implementation of the Open Message Passing Interface.

```sh
wget https://download.open-mpi.org/release/open-mpi/v4.1/openmpi-4.1.1.tar.gz -P /sources &&

tar xzvf /sources/openmpi-4.1.1.tar.gz &&
cd        openmpi-4.1.1                &&

./configure --prefix=/usr                         \
            --sysconfdir=/etc                     \
            --localstatedir=/var                  \
            --docdir=/usr/share/doc/openmpi-4.1.1 \
            --enable-mpi-fortran                  \
            --enable-mpi-cxx &&

make              &&
make -k check     &&
sudo make install &&

cd .. &&
rm -rf openmpi-4.1.1
```

## hwloc

Libraries to aid HPC applications that use OpenMPI to discover hardware locality features. Includes the handy `lstopo` utility that visualizes your PCI bus connections in a terminal, information you would normally need to pull from the manuals for your processor and motherboard.

```sh
wget https://download.open-mpi.org/release/hwloc/v2.4/hwloc-2.4.1.tar.gz -P /sources &&

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

Many, many C++ libraries. Is an optional dependency of some other libraries and may result in better performance. Many later tools explicitly depend on it.

```sh
wget https://boostorg.jfrog.io/artifactory/main/release/1.76.0/source/boost_1_76_0.tar.gz -P /sources &&

tar xzvf /sources/boost_1_76_0.tar.gz &&
cd        boost_1_76_0                &&

./bootstrap.sh --prefix=/usr                &&
./b2 stage -j12 threading=multi link=shared &&

pushd tools/build/test; python test_all.py; popd &&

sudo ./b2 install threading=multi link=shared &&

cd .. &&
sudo rm -rf boost_1_76_0
```

## exempi

Implementation of Adobe XMP.

```sh
wget https://libopenraw.freedesktop.org/download/exempi-2.5.2.tar.bz2 -P /sources &&

tar xvf /sources/exempi-2.5.2.tar.bz2 &&
cd       exempi-2.5.2                 &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf exempi-2.5.2
```

## Exiv2

Library and CLI for managing image and video metadata.

```sh
wget https://www.exiv2.org/builds/exiv2-0.27.4-Source.tar.gz -P /sources &&

tar xzvf /sources/exiv2-0.27.4-Source.tar.gz &&
cd        exiv2-0.27.4-Source                &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr  \
      -DCMAKE_BUILD_TYPE=Release   \
      -DEXIV2_ENABLE_VIDEO=yes     \
      -DEXIV2_ENABLE_WEBREADY=yes  \
      -DEXIV2_ENABLE_CURL=yes      \
      -DEXIV2_BUILD_SAMPLES=no     \
      -G "Unix Makefiles" .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf exiv2-0.27.4-Source
```

## GLib

Integral core library for `GNOME` but is also used by many other tools.

```sh
wget https://download.gnome.org/sources/glib/2.68/glib-2.68.3.tar.xz -P /sources &&
wget https://www.linuxfromscratch.org/patches/blfs/svn/glib-2.68.3-skip_warnings-1.patch -P /sources/patches &&

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
wget https://github.com/rockdaboot/libpsl/releases/download/0.21.1/libpsl-0.21.1.tar.gz -P /sources &&

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

HTTP client/server library intended for `GNOME` and uses the `GObject` introspection system, but also used by other tools.

```sh
wget https://download.gnome.org/sources/libsoup/2.72/libsoup-2.72.0.tar.xz -P /sources &&

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
