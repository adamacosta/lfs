# Install temporary tools

At this point, we can build the remainder of our temporary toolchain that will be used to build the real LFS system, now that we no longer have any dependencies on the host system.

From the `/sources` directory:

(if you created an in-memory build tree)

```sh
cd build_dir
```

## libstdc++

First, we need to complete the build of `gcc` pass 2, which did not initially include `libstdc++`.

```sh
tar xvf $LFS/sources/gcc-11.1.0.tar.xz &&
cd       gcc-11.1.0                    &&

ln -s gthr-posix.h libgcc/gthr-default.h &&

mkdir -v build &&
cd       build &&

../libstdc++-v3/configure            \
    CXXFLAGS="-g -O2 -D_GNU_SOURCE"  \
    --prefix=/usr                    \
    --disable-multilib               \
    --disable-nls                    \
    --host=$(uname -m)-lfs-linux-gnu \
    --disable-libstdcxx-pch &&

make         &&
make install &&

cd ../.. &&
rm -rf gcc-11.1.0
```

## gettext

```sh
tar xvf $LFS/sources/gettext-0.21.tar.xz &&
cd       gettext-0.21                    &&

./configure --disable-shared &&

make                                                        &&
cp -v gettext-tools/src/{msgfmt,msgmerge,xgettext} /usr/bin &&

cd .. &&
rm -rf gettext-0.21
```

## bison

```sh
tar xvf $LFS/sources/bison-3.7.6.tar.xz &&
cd       bison-3.7.6                    &&

./configure --prefix=/usr \
            --docdir=/usr/share/doc/bison-3.7.5 &&
make         &&
make install &&

cd .. &&
rm -rf bison-3.7.6
```

## perl

```sh
tar xvf $LFS/sources/perl-5.32.1.tar.xz &&
cd       perl-5.32.1                    &&

sh Configure -des                                        \
             -Dprefix=/usr                               \
             -Dvendorprefix=/usr                         \
             -Dprivlib=/usr/lib/perl5/5.32/core_perl     \
             -Darchlib=/usr/lib/perl5/5.32/core_perl     \
             -Dsitelib=/usr/lib/perl5/5.32/site_perl     \
             -Dsitearch=/usr/lib/perl5/5.32/site_perl    \
             -Dvendorlib=/usr/lib/perl5/5.32/vendor_perl \
             -Dvendorarch=/usr/lib/perl5/5.32/vendor_perl &&

make         &&
make install &&

cd .. &&
rm -rf perl-5.32.1
```

## Python

We do not need `pip` in the temporary environment.

```sh
tar xvf $LFS/sources/Python-3.9.5.tar.xz &&
cd       Python-3.9.5                    &&

./configure --prefix=/usr   \
            --enable-shared \
            --without-ensurepip &&

make         &&
make install &&

cd .. &&
rm -rf Python-3.9.5
```

## texinfo

```sh
tar xvf $LFS/sources/texinfo-6.7.tar.xz &&
cd       texinfo-6.7                    &&

./configure --prefix=/usr &&

make         &&
make install &&

cd .. &&
rm -rf texinfo-6.7
```

## util-linux

```sh
tar xvf $LFS/sources/util-linux-2.36.2.tar.xz &&
cd       util-linux-2.36.2                    &&

mkdir -pv /var/lib/hwclock &&

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
            runstatedir=/run &&

make         &&
make install &&

cd .. &&
rm -rf util-linux-2.36.2
```
