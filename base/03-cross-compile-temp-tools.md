# Cross compiling temporary tools

## m4

```sh
tar -xvf m4-1.4.18.tar.xz
cd m4-1.4.18

sed -i 's/IO_ftrylockfile/IO_EOF_SEEN/' lib/*.c
echo "#define _IO_IN_BACKUP 0x100" >> lib/stdio-impl.h

mkdir -v build
cd build

../configure --prefix=/usr    \
             --host=$LFS_TGT  \
             --build=$(../build-aux/config.guess)

make -j
make DESTDIR=$LFS install

cd ../..
rm -rf m4-1.4.18
```

## ncurses

```sh
tar xzvf ncurses-6.2.tar.gz
cd ncurses-6.2

sed -i s/mawk// configure

mkdir -v build
cd build

./configure --prefix=/usr                \
            --host=$LFS_TGT              \
            --build=$(./config.guess)    \
            --mandir=/usr/share/man      \
            --with-manpage-format=normal \
            --with-shared                \
            --without-debug              \
            --without-ada                \
            --without-normal             \
            --enable-widec

make -j
make DESTDIR=$LFS TIC_PATH=$(pwd)/build/progs/tic install
echo "INPUT(-lncursesw)" > $LFS/usr/lib/libncurses.so

mv -v $LFS/usr/lib/libncursesw.so.6* $LFS/lib
ln -sfv ../../lib/$(readlink $LFS/usr/lib/libncursesw.so) $LFS/usr/lib/libncursesw.so

cd ..
rm -rf ncurses-6.2
```

## bash

```sh
tar xzvf bash-5.1.tar.gz
cd bash-5.1

mkdir -v build
cd build

../configure --prefix=/usr                      \
             --build=$(../support/config.guess) \
             --host=$LFS_TGT                    \
             --without-bash-malloc

make -j
make DESTDIR=$LFS install
mv $LFS/usr/bin/bash $LFS/bin/bash
ln -sv bash $LFS/bin/sh

cd ../..
rm -rf bash-5.1
```

## coreutils

```sh
tar xvf coreutils-8.32.tar.xz
cd coreutils-8.32

./configure --prefix=/usr                     \
            --host=$LFS_TGT                   \
            --build=$(build-aux/config.guess) \
            --enable-install-program=hostname \
            --enable-no-install-program=kill,uptime

make -j
make DESTDIR=$LFS install

mv -v $LFS/usr/bin/{cat,chgrp,chmod,chown,cp,date,dd,df,echo} $LFS/bin
mv -v $LFS/usr/bin/{false,ln,ls,mkdir,mknod,mv,pwd,rm}        $LFS/bin
mv -v $LFS/usr/bin/{rmdir,stty,sync,true,uname}               $LFS/bin
mv -v $LFS/usr/bin/{head,nice,sleep,touch}                    $LFS/bin
mv -v $LFS/usr/bin/chroot                                     $LFS/usr/sbin
mkdir -pv $LFS/usr/share/man/man8
mv -v $LFS/usr/share/man/man1/chroot.1                        $LFS/usr/share/man/man8/chroot.8
sed -i 's/"1"/"8"/'                                           $LFS/usr/share/man/man8/chroot.8

cd ..
rm -rf coreutils-8.32
```

## diffutils

```sh
tar xvf diffutils-3.7.tar.xz
cd diffutils-3.7

./configure --prefix=/usr --host=$LFS_TGT
make -j
make DESTDIR=$LFS install

cd ..
rm -rf diffutils-3.7
```

## file

```sh
tar xzvf file-5.39.tar.gz
cd file-5.39

mkdir -v build
cd build

../configure --disable-bzlib      \
             --disable-libseccomp \
             --disable-xzlib      \
             --disable-zlib

make -j
cd ..

./configure --prefix=/usr --host=$LFS_TGT --build=$(./config.guess)
make -j FILE_COMPILE=$(pwd)/build/src/file
make DESTDIR=$LFS install

cd ..
rm -rf file-5.39
```

## findutils

```sh
tar xvf findutils-4.8.0.tar.xz
cd findutils-4.8.0

./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)

make -j
make DESTDIR=$LFS install

mv -v $LFS/usr/bin/find $LFS/bin
sed -i 's|find:=${BINDIR}|find:=/bin|' $LFS/usr/bin/updatedb

cd ..
rm -rf findutils-4.8.0
```

## gawk

```sh
tar xvf gawk-5.1.0.tar.xz
cd gawk-5.1.0

sed -i 's/extras//' Makefile.in

./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(./config.guess)

make -j
make DESTDIR=$LFS install

cd ..
rm -rf gawk-5.1.0
```

## grep

```sh
tar xvf grep-3.6.tar.xz
cd grep-3.6

./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --bindir=/bin
make -j
make DESTDIR=$LFS install

cd ..
rm -rf grep-3.6
```

## gzip

```sh
tar xvf gzip-1.10.tar.xz
cd gzip-1.10

./configure --prefix=/usr --host=$LFS_TGT
make -j
make DESTDIR=$LFS install

mv -v $LFS/usr/bin/gzip $LFS/bin/gzip

cd ..
rm -rg gzip-1.10
```

## make

```sh
tar xzvf make-4.3.tar.gz
cd make-4.3

./configure --prefix=/usr   \
            --without-guile \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)
make -j
make DESTDIR=$LFS install

cd ..
rm -rf make-4.3
```

## patch

```sh
tar xvf patch-2.7.6.tar.xz
cd patch-2.7.6

./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --build=$(build-aux/config.guess)
make -j
make DESTDIR=$LFS install

cd ..
rm -rf patch-2.7.6
```

## sed

```sh
tar xvf sed-4.8.tar.xz
cd sed-4.8

./configure --prefix=/usr   \
            --host=$LFS_TGT \
            --bindir=/bin
make -j
make DESTDIR=$LFS install

cd ..
rm -rf sed-4.8
```

## tar

```sh
tar xvf tar-1.34.tar.xz
cd tar-1.34

./configure --prefix=/usr                     \
            --host=$LFS_TGT                   \
            --build=$(build-aux/config.guess) \
            --bindir=/bin
make -j
make DESTDIR=$LFS install

cd ..
rm -rf tar-1.34
```

## xz

```sh
tar xvf xz-5.2.5.tar.xz
cd xz-5.2.5

./configure --prefix=/usr                     \
            --host=$LFS_TGT                   \
            --build=$(build-aux/config.guess) \
            --disable-static                  \
            --docdir=/usr/share/doc/xz-5.2.5
make -j
make DESTDIR=$LFS install

mv -v $LFS/usr/bin/{lzma,unlzma,lzcat,xz,unxz,xzcat}  $LFS/bin
mv -v $LFS/usr/lib/liblzma.so.*                       $LFS/lib
ln -svf ../../lib/$(readlink $LFS/usr/lib/liblzma.so) $LFS/usr/lib/liblzma.so

cd ..
rm -rf xz-5.2.5
```