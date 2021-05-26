# Compiling the cross-toolchain pass 1

First, create the initial directory structure. Then descend into the `/sources` directory in the new system and build each package.

```sh
mkdir -pv $LFS/{usr,lib,lib64,var,etc,bin,sbin,tools} &&
ln -s usr/lib $LFS/lib                                &&
ln -s usr/lib $LFS/lib64                              &&
ln -s usr/bin $LFS/bin                                &&
ln -s usr/bin $LFS/sbin                               &&
ln -s lib     $LFS/usr/lib64                          &&
ln -s bin     $LFS/usr/sbin                           &&
ln -s bin     $LFS/usr/local/sbin                     &&
cd $LFS/sources
```

This differs from the approach in Linux From Scratch of creating separate directories, but simplifies things. This way, all executables and libraries will be installed into the `/usr` prefix and anything looking elsewhere will simply get a link. Also, there is no need for a separate architecture-specific lib dir since we will not be doing any multilib builds. This is more in line with the way a typical modern Linux distro handles directory layout, and removes the need to move executables and libraries out of the `/usr` tree and into the `/` tree to deal with tools that hardcode an expected path.

Now we unpack and build each package used for the initial cross-compile toolchain. I recommend setting `MAKEFLAGS` to use unlimited processors by default and using the `-j<N>` flag explicitly for packages known not to compile correctly when building in parallel:

```sh
export MAKEFLAGS="-j"
echo "export MAKEFLAGS=\"-j\"" >> ~/.bashrc
```

`LFS_TGT` describes the machine triplet of the target architecture you are building for. See [Implementation of Cross-Compilation for LFS](https://www.linuxfromscratch.org/lfs/view/stable-systemd/partintro/toolchaintechnotes.html) for details, but this is likely to be `x86_64-pc-linux-gnu` on your real system. Recall that Linux From Scratch sets it to `x86_64-lfs-linux-gnu` during the cross-compilation phase. This is to trick the various `configure` scripts out of using host tools and forcing them to use the cross-compilation tools.

## binutils

```sh
tar xvf binutils-2.36.1.tar.xz &&
cd  binutils-2.36              &&
mkdir -v build                 &&
cd       build

../configure --prefix=$LFS/tools \
             --with-sysroot=$LFS \
             --target=$LFS_TGT   \
             --disable-nls       \
             --disable-werror

make &&
make install
cd ../.. &&
rm -rf binutils-2.36
```

## gcc

```sh
tar xvf gcc-10.2.0.tar.xz &&
cd gcc-10.2.0

tar -xf ../mpfr-4.1.0.tar.xz &&
mv  -v mpfr-4.1.0 mpfr       &&
tar -xf ../gmp-6.2.1.tar.xz  &&
mv  -v gmp-6.2.1 gmp         &&
tar -xf ../mpc-1.2.1.tar.gz  &&
mv  -v mpc-1.2.1 mpc

sed -e '/m64=/s/lib64/lib/' -i.orig gcc/config/i386/t-linux64 &&
mkdir -v build &&
cd       build

../configure                 \
    --target=$LFS_TGT        \
    --prefix=$LFS/tools      \
    --with-sysroot=$LFS      \
    --with-newlib            \
    --without-headers        \
    --enable-initfini-array  \
    --disable-nls            \
    --disable-shared         \
    --disable-multilib       \
    --disable-decimal-float  \
    --disable-threads        \
    --disable-libatomic      \
    --disable-libgomp        \
    --disable-libquadmath    \
    --disable-libssp         \
    --disable-libvtv         \
    --disable-libstdcxx      \
    --enable-languages=c,c++

make -j 4
make install

cd .. &&
cat gcc/limitx.h gcc/glimits.h gcc/limity.h > \
  `dirname $($LFS_TGT-gcc -print-libgcc-file-name)`/install-tools/include/limits.h &&
cd .. &&
rm -rf gcc-10.2.0
```

## Linux API headers

```sh
tar xvf linux-5.10.17.tar.xz &&
cd linux-5.10.17

make   mrproper                       &&
make   headers                        &&
find   usr/include -name '.*' -delete &&
rm     usr/include/Makefile           &&
cp -rv usr/include $LFS/usr

cd .. &&
rm -rf linux-5.10.17
```

## glibc

```sh
tar xvf glibc-2.33.tar.xz &&
cd glibc-2.33

ln -sfv ../lib/ld-linux-x86-64.so.2 $LFS/lib64 &&
ln -sfv ../lib/ld-linux-x86-64.so.2 $LFS/lib64/ld-lsb-x86-64.so.3

patch -Np1 -i ../glibc-2.33-fhs-1.patch

mkdir -v build &&
cd       build &&

../configure                             \
      --prefix=/usr                      \
      --host=$LFS_TGT                    \
      --build=$(../scripts/config.guess) \
      --enable-kernel=3.2                \
      --with-headers=$LFS/usr/include    \
      libc_cv_slibdir=/lib

make &&
make DESTDIR=$LFS install

cd ../.. &&
rm -rf glibc-2.33
```

## Install limit.h

```sh
$LFS/tools/libexec/gcc/$LFS_TGT/10.2.0/install-tools/mkheaders
```

## libstc++

```sh
tar xvf gcc-10.2.0.tar.xz &&
cd gcc-10.2.0

mkdir -v build &&
cd       build &&

../libstdc++-v3/configure           \
    --host=$LFS_TGT                 \
    --build=$(../config.guess)      \
    --prefix=/usr                   \
    --disable-multilib              \
    --disable-nls                   \
    --disable-libstdcxx-pch         \
    --with-gxx-include-dir=/tools/$LFS_TGT/include/c++/10.2.0

make &&
make DESTDIR=$LFS install
```
