# Cross compiling temporary tools

Now we use the cross-compile toolchain we just created to build a new set of temporary GNU utilities that will be used to bootstrap the new system.

## m4

```sh
tar xvf $LFS/sources/m4-1.4.18.tar.xz &&
cd       m4-1.4.18                    &&

sed -i 's/IO_ftrylockfile/IO_EOF_SEEN/' lib/*.c        &&
echo "#define _IO_IN_BACKUP 0x100" >> lib/stdio-impl.h &&
mkdir -v build &&
cd       build &&

../configure --prefix=/usr    \
             --host=$LFS_TGT  \
             --build=$(../build-aux/config.guess) &&

make                      &&
make DESTDIR=$LFS install &&

cd ../.. &&
rm -rf m4-1.4.18
```

## ncurses

```sh
tar xzvf $LFS/sources/ncurses-6.2.tar.gz &&
cd        ncurses-6.2                    &&

sed -i s/mawk// configure &&

mkdir -v build &&
cd       build &&

../configure --prefix=/usr                \
             --host=$LFS_TGT              \
             --build=$(./config.guess)    \
             --mandir=/usr/share/man      \
             --with-manpage-format=normal \
             --with-shared                \
             --without-debug              \
             --without-ada                \
             --without-normal             \
             --enable-widec &&

make                                                      &&
make DESTDIR=$LFS TIC_PATH=$(pwd)/build/progs/tic install &&
echo "INPUT(-lncursesw)" > $LFS/usr/lib/libncurses.so     &&

cd ../.. &&
rm -rf ncurses-6.2
```

## bash

```sh
tar xzvf $LFS/sources/bash-5.1.tar.gz &&
cd        bash-5.1                    &&

./configure --prefix=/usr                     \
            --build=$(./support/config.guess) \
            --host=$LFS_TGT                   \
            --without-bash-malloc &&

make -j4                  &&
make DESTDIR=$LFS install &&
ln -sv bash $LFS/bin/sh   &&

cd .. &&
rm -rf bash-5.1
```

## coreutils

```sh
tar xvf $LFS/sources/coreutils-8.32.tar.xz &&
cd       coreutils-8.32                    &&

./configure --prefix=/usr                     \
            --host=$LFS_TGT                   \
            --build=$(build-aux/config.guess) \
            --enable-install-program=hostname \
            --enable-no-install-program=kill,uptime &&

make                      &&
make DESTDIR=$LFS install &&

cd .. &&
rm -rf coreutils-8.32
```

## diffutils

```sh
tar xvf $LFS/sources/diffutils-3.7.tar.xz &&
cd       diffutils-3.7                    &&

./configure --prefix=/usr \
            --host=$LFS_TGT &&

make                      &&
make DESTDIR=$LFS install &&

cd .. &&
rm -rf diffutils-3.7
```

## file

```sh
tar xzvf $LFS/sources/file-5.40.tar.gz &&
cd        file-5.40                    &&

mkdir -v build &&
cd       build &&

../configure --disable-bzlib      \
             --disable-libseccomp \
             --disable-xzlib      \
             --disable-zlib

make  &&
cd .. &&

./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(./config.guess)   &&

make FILE_COMPILE=$(pwd)/build/src/file &&
make DESTDIR=$LFS install               &&

cd .. &&
rm -rf file-5.40
```

## findutils

```sh
tar xvf $LFS/sources/findutils-4.8.0.tar.xz &&
cd       findutils-4.8.0                    &&

./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess) &&

make                      &&
make DESTDIR=$LFS install &&

cd .. &&
rm -rf findutils-4.8.0
```

## gawk

```sh
tar xvf $LFS/sources/gawk-5.1.0.tar.xz &&
cd       gawk-5.1.0                    &&

sed -i 's/extras//' Makefile.in &&

./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(./config.guess) &&

make                      &&
make DESTDIR=$LFS install &&

cd .. &&
rm -rf gawk-5.1.0
```

## grep

```sh
tar xvf $LFS/sources/grep-3.6.tar.xz &&
cd       grep-3.6                    &&

./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --bindir=/bin &&

make                      &&
make DESTDIR=$LFS install &&

cd .. &&
rm -rf grep-3.6
```

## gzip

```sh
tar xvf $LFS/sources/gzip-1.10.tar.xz &&
cd       gzip-1.10                    &&

./configure --prefix=/usr \
            --host=$LFS_TGT &&

make                      &&
make DESTDIR=$LFS install &&

cd .. &&
rm -rf gzip-1.10
```

## make

```sh
tar xzvf $LFS/sources/make-4.3.tar.gz &&
cd        make-4.3                    &&

./configure --prefix=/usr   \
            --without-guile \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess) &&
make                      &&
make DESTDIR=$LFS install &&

cd .. &&
rm -rf make-4.3
```

## patch

```sh
tar xvf $LFS/sources/patch-2.7.6.tar.xz &&
cd       patch-2.7.6                    &&

./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess) &&

make                      &&
make DESTDIR=$LFS install &&

cd .. &&
rm -rf patch-2.7.6
```

## sed

```sh
tar xvf $LFS/sources/sed-4.8.tar.xz &&
cd       sed-4.8                    &&

./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --bindir=/bin &&

make &&
make DESTDIR=$LFS install &&

cd .. &&
rm -rf sed-4.8
```

## tar

```sh
tar xvf $LFS/sources/tar-1.34.tar.xz &&
cd       tar-1.34                    &&

./configure --prefix=/usr                     \
            --host=$LFS_TGT                   \
            --build=$(build-aux/config.guess) \
            --bindir=/bin &&

make                      &&
make DESTDIR=$LFS install &&

cd .. &&
rm -rf tar-1.34
```

## xz

```sh
tar xvf $LFS/sources/xz-5.2.5.tar.xz &&
cd       xz-5.2.5                    &&

./configure --prefix=/usr                     \
            --host=$LFS_TGT                   \
            --build=$(build-aux/config.guess) \
            --disable-static                  \
            --docdir=/usr/share/doc/xz-5.2.5 &&

make                      &&
make DESTDIR=$LFS install &&

cd .. &&
rm -rf xz-5.2.5
```
