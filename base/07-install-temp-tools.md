# Install temporary tools

## libstdc++

```sh
cd sources
tar xvf gcc-10.2.0.xz
cd gcc-10.2.0

ln -s gthr-posix.h libgcc/gthr-default.h

mkdir -v build
cd build

../libstdc++-v3/configure            \
    CXXFLAGS="-g -O2 -D_GNU_SOURCE"  \
    --prefix=/usr                    \
    --disable-multilib               \
    --disable-nls                    \
    --host=$(uname -m)-lfs-linux-gnu \
    --disable-libstdcxx-pch
make -j
make install

cd ../..
rm -rf gcc-10.2.0
```

## gettext

```sh
tar xvf gettext-0.21.tar.xz
cd gettext-0.21

./configure --disable-shared
make -j
cp -v gettext-tools/src/{msgfmt,msgmerge,xgettext} /usr/bin

cd ..
rm -rf gettext-0.21
```

## bison

```sh
tar xvf bison-3.7.5.tar.xz
cd bison-3.7.5

./configure --prefix=/usr --docdir=/usr/share/doc/bison-3.7.5
make -j
make install

cd ..
rm bison-3.7.5
```

## perl

```sh
tar xvf perl-5.32.1.tar.xz
cd perl-5.32.1

sh Configure -des                                        \
             -Dprefix=/usr                               \
             -Dvendorprefix=/usr                         \
             -Dprivlib=/usr/lib/perl5/5.32/core_perl     \
             -Darchlib=/usr/lib/perl5/5.32/core_perl     \
             -Dsitelib=/usr/lib/perl5/5.32/site_perl     \
             -Dsitearch=/usr/lib/perl5/5.32/site_perl    \
             -Dvendorlib=/usr/lib/perl5/5.32/vendor_perl \
             -Dvendorarch=/usr/lib/perl5/5.32/vendor_perl
make -j
make install

cd ..
rm perl-5.32.1
```

## Python

```sh
tar xvf Python-3.9.2.tar.xz
cd Python-3.9.2

./configure --prefix=/usr   \
            --enable-shared \
            --without-ensurepip
make -j
make -j install

cd ..
rm -rf Python-3.9.2
```

## texinfo

```sh
tar xvf textinfo-6.7.tar.xz
cd textinfo-6.7

./configure --prefix=/usr
make -j
make -j install
```

## util-linux

```sh
tar xvf util-linux-2.36.2.tar.xz
cd util-linux-2.36.2

mkdir -pv /var/lib/hwclock

./configure ADJTIME_PATH=/var/lib/hwclock/adjtime    \
            --docdir=/usr/share/doc/util-linux-2.36.2 \
            --disable-chfn-chsh  \
            --disable-login      \
            --disable-nologin    \
            --disable-su         \
            --disable-setpriv    \
            --disable-runuser    \
            --disable-pylibmount \
            --disable-static     \
            --without-python     \
            runstatedir=/run
make -j
make -j install

cd ..
rm util-linux-2.36.2
```