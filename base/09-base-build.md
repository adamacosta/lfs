# Build the LFS system

Remount virtual filesystems and enter chroot environment

```sh
mount -v --bind /dev $LFS/dev
mount -v --bind /dev/pts $LFS/dev/pts
mount -vt proc proc $LFS/proc
mount -vt sysfs sysfs $LFS/sys
mount -vt tmpfs tmpfs $LFS/run

chroot "$LFS" /usr/bin/env -i   \
    HOME=/root                  \
    TERM="$TERM"                \
    PS1='(lfs chroot) \u:\w\$ ' \
    PATH=/bin:/usr/bin:/sbin:/usr/sbin \
    /bin/bash --login +h

cd /souces
```

## Install basic system software

### man-pages

```sh
tar xvf man-pages-5.10.tar.xz
cd man-pages-5.10

make -j install

cd ..
rm -rf man-pages-5.10
```

### iana-etc

```sh
tar xzvf iana-etc-20210202.tar.gz
cd iana-etc-20210202

cp services protocols /etc

cd ..
rm -rf iana-etc-20210202
```

### glibc

#### Build glibc

```sh
tar xvf glibc-2.33.tar.xz
cd glibc-2.33

patch -Np1 -i ../glibc-2.33-fhs-1.patch
sed -e '402a\      *result = local->data.services[database_index];' \
    -i nss/nss_database.c

mkdir -v build
cd build

../configure --prefix=/usr                            \
             --disable-werror                         \
             --enable-kernel=3.2                      \
             --enable-stack-protector=strong          \
             --with-headers=/usr/include              \
             libc_cv_slibdir=/lib

make -j 4
make -j 4 check
touch /etc/ld.so.conf
sed '/test-installation/s@$(PERL)@echo not running@' -i ../Makefile
make -j install

cp -v ../nscd/nscd.conf /etc/nscd.conf
mkdir -pv /var/cache/nscd

install -v -Dm644 ../nscd/nscd.tmpfiles /usr/lib/tmpfiles.d/nscd.conf
install -v -Dm644 ../nscd/nscd.service /lib/systemd/system/nscd.service

mkdir -pv /usr/lib/locale
make localedata/install-locales
```

#### Configure glibc

```sh
cat > /etc/nsswitch.conf << EOF
# Begin /etc/nsswitch.conf

passwd: files
group: files
shadow: files

hosts: files dns
networks: files

protocols: files
services: files
ethers: files
rpc: files

# End /etc/nsswitch.conf
EOF

tar -xf ../../tzdata2021a.tar.gz

ZONEINFO=/usr/share/zoneinfo
mkdir -pv $ZONEINFO/{posix,right}

for tz in etcetera southamerica northamerica europe africa antarctica  \
          asia australasia backward; do
    zic -L /dev/null   -d $ZONEINFO       ${tz}
    zic -L /dev/null   -d $ZONEINFO/posix ${tz}
    zic -L leapseconds -d $ZONEINFO/right ${tz}
done

cp -v zone.tab zone1970.tab iso3166.tab $ZONEINFO
zic -d $ZONEINFO -p America/New_York
unset ZONEINFO

ln -sfv /usr/share/zoneinfo/America/Chicago /etc/localtime
```

#### Configure ld.so

```sh
cat > /etc/ld.so.conf << EOF
# Begin /etc/ld.so.conf
/usr/local/lib
/opt/lib

# Add an include directory
include /etc/ld.so.conf.d/*.conf

EOF

mkdir -pv /etc/ld.so.conf.d
```

#### Cleanup

```sh
cd ../..
rm -rf glibc-2.33
```

### zlib

```sh
tar xvf zlib-1.2.11.tar.xz
cd zlib-1.2.11

./configure --prefix=/usr
make -j

make -j check
make -j install

mv -v /usr/lib/libz.so.* /lib
ln -sfv ../../lib/$(readlink /usr/lib/libz.so) /usr/lib/libz.so
rm -fv /usr/lib/libz.a

cd ..
rm -rf zlib-1.2.11
```

### bzip2

```sh
tar xzvf bzip2-1.0.8.tar.gz
cd bzip2-1.0.8

patch -Np1 -i ../bzip2-1.0.8-install_docs-1.patch
sed -i 's@\(ln -s -f \)$(PREFIX)/bin/@\1@' Makefile
sed -i "s@(PREFIX)/man@(PREFIX)/share/man@g" Makefile

make -f Makefile-libbz2_so
make clean
make -j
make PREFIX=/usr install

cp -v bzip2-shared /bin/bzip2
cp -av libbz2.so* /lib
ln -sv ../../lib/libbz2.so.1.0 /usr/lib/libbz2.so
rm -v /usr/bin/{bunzip2,bzcat,bzip2}
ln -sv bzip2 /bin/bunzip2
ln -sv bzip2 /bin/bzcat
rm -fv /usr/lib/libbz2.a
```

### xz

```sh
tar xvf xz-5.2.5.tar.xz
cd xz-5.2.5

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/xz-5.2.5
make -j
make -j install
mv -v   /usr/bin/{lzma,unlzma,lzcat,xz,unxz,xzcat} /bin
mv -v /usr/lib/liblzma.so.* /lib
ln -svf ../../lib/$(readlink /usr/lib/liblzma.so) /usr/lib/liblzma.so

cd ..
rm -rf xz-5.2.5
```

### zstd

```sh
tar xzvf zstd-1.4.8.tar.gz
cd zstd-1.4.8

make -j
make -j check
make prefix=/usr install

rm -v /usr/lib/libzstd.a
mv -v /usr/lib/libzstd.so.* /lib
ln -sfv ../../lib/$(readlink /usr/lib/libzstd.so) /usr/lib/libzstd.so

cd ..
rm -rf zstd-1.4.8
```

### file

```sh
tar xvf file-5.39.tar.xz
cd file-5.39

./configure --prefix=/usr
make -j
make -j check
make -j install

cd ..
rm -rf file-5.39
```

### readline

```sh
tar xzvf readline-8.1.tar.gz
cd readline-8.1

sed -i '/MV.*old/d' Makefile.in
sed -i '/{OLDSUFF}/c:' support/shlib-install

./configure --prefix=/usr    \
            --disable-static \
            --with-curses    \
            --docdir=/usr/share/doc/readline-8.1
make SHLIB_LIBS="-lncursesw" -j
make SHLIB_LIBS="-lncursesw" -j install

mv -v /usr/lib/lib{readline,history}.so.* /lib
ln -sfv ../../lib/$(readlink /usr/lib/libreadline.so) /usr/lib/libreadline.so
ln -sfv ../../lib/$(readlink /usr/lib/libhistory.so ) /usr/lib/libhistory.so
install -v -m644 doc/*.{ps,pdf,html,dvi} /usr/share/doc/readline-8.1

cd ..
rm -rf readline-8.1
```

### m4

```sh
tar xvf m4-1.4.18.tar.xz
cd m4-1.4.18

sed -i 's/IO_ftrylockfile/IO_EOF_SEEN/' lib/*.c
echo "#define _IO_IN_BACKUP 0x100" >> lib/stdio-impl.h

./configure --prefix=/usr
make -j
make -j check
make -j install

cd ..
rm -rf m4-1.4.18
```

### bc

```sh
tar xvf bc-3.3.0.tar.xz
cd bc-3.3.0

PREFIX=/usr CC=gcc ./configure.sh -G -O3
make -j
make -j test
make -j install

cd ..
rm -rf bc-3.3.0
```

### flex

```sh
tar xzvf flex-2.6.4.tar.gz
cd flex-2.6.4

./configure --prefix=/usr \
            --docdir=/usr/share/doc/flex-2.6.4 \
            --disable-static
make -j
make -j check
make -j install
ln -sv flex /usr/bin/lex

cd ..
rm -rf flex-2.6.4
```

### tcl

```sh
tar xzvf tcl8.6.11-src.tar.gz
cd tcl8.6.11
tar xzvf ../tcl8.6.11-html.tar.gz --strip-component

SRCDIR=$(pwd)
cd unix
./configure --prefix=/usr           \
            --mandir=/usr/share/man \
            --enable-64bit
make -j


sed -e "s|$SRCDIR/unix|/usr/lib|" \
    -e "s|$SRCDIR|/usr/include|"  \
    -i tclConfig.sh

sed -e "s|$SRCDIR/unix/pkgs/tdbc1.1.2|/usr/lib/tdbc1.1.2|" \
    -e "s|$SRCDIR/pkgs/tdbc1.1.2/generic|/usr/include|"    \
    -e "s|$SRCDIR/pkgs/tdbc1.1.2/library|/usr/lib/tcl8.6|" \
    -e "s|$SRCDIR/pkgs/tdbc1.1.2|/usr/include|"            \
    -i pkgs/tdbc1.1.2/tdbcConfig.sh

sed -e "s|$SRCDIR/unix/pkgs/itcl4.2.1|/usr/lib/itcl4.2.1|" \
    -e "s|$SRCDIR/pkgs/itcl4.2.1/generic|/usr/include|"    \
    -e "s|$SRCDIR/pkgs/itcl4.2.1|/usr/include|"            \
    -i pkgs/itcl4.2.1/itclConfig.sh

unset SRCDIR

make -j test
make -j install
chmod -v u+w /usr/lib/libtcl8.6.so
make -j install-private-headers
ln -sfv tclsh8.6 /usr/bin/tclsh
mv /usr/share/man/man3/{Thread,Tcl_Thread}.3

cd ../..
rm -rf tcl8.6.11
```

### expect

```sh
tar xzvf expect5.45.4.tar.gz
cd expect5.45.4

./configure --prefix=/usr           \
            --with-tcl=/usr/lib     \
            --enable-shared         \
            --mandir=/usr/share/man \
            --with-tclinclude=/usr/include
make -j
make -j test
make -j install
ln -svf expect5.45.4/libexpect5.45.4.so /usr/lib

cd ..
rm -rf expect5.45.4
```

### DejaGNU

```sh
tar xzvf dejagnu-1.6.2.tar.gz
cd dejagnu-1.6.2

./configure --prefix=/usr
makeinfo --html --no-split -o doc/dejagnu.html doc/dejagnu.texi
makeinfo --plaintext       -o doc/dejagnu.txt  doc/dejagnu.texi
make -j install
install -v -dm755  /usr/share/doc/dejagnu-1.6.2
install -v -m644   doc/dejagnu.{html,txt} /usr/share/doc/dejagnu-1.6.2
make -j check
```

### binutils

`--enable-targets=x86_64-pep` enables support for EFI executables and is a dependency of `gnu-efi`.

```sh
tar xvf binutils-2.36.1
cd binutils-2.36.1

sed -i '/@\tincremental_copy/d' gold/testsuite/Makefile.in

mkdir -v build
cd build

../configure --prefix=/usr       \
             --enable-gold       \
             --enable-ld=default \
             --enable-plugins    \
             --enable-shared     \
             --disable-werror    \
             --enable-64-bit-bfd \
             --with-system-zlib  \
             --enable-targets=x86_64-pep
make tooldir=/usr
make -k check
make tooldir=/usr install
rm -fv /usr/lib/lib{bfd,ctf,ctf-nobfd,opcodes}.a

cd ../..
rm -rf binutils-2.36.1
```

### gmp

```sh
tar xvf gmp-6.2.1.tar.xz
cd gmp-6.2.1

./configure --prefix=/usr    \
            --enable-cxx     \
            --disable-static \
            --docdir=/usr/share/doc/gmp-6.2.1
make -j
make -j html
make -j check 2>&1 | tee gmp-check-log
awk '/# PASS:/{total+=$3} ; END{print total}' gmp-check-log
make -j install
make -j install-html

cd ..
rm -rf gmp-6.2.1
```

### mpfr

```sh
tar xvf mpfr-4.1.0.tar.xz
cd mpfr-4.1.0

./configure --prefix=/usr        \
            --disable-static     \
            --enable-thread-safe \
            --docdir=/usr/share/doc/mpfr-4.1.0
make -j
make -j html
make -j check
make -j install
make -j install-html

cd ..
rm -rf mpfr-4.1.0
```

### mpc

```sh
tar xzvf mpc-1.2.1.tar.gz
cd mpc-1.2.1

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/mpc-1.2.1
make -j
make -j html
make -j check
make -j install
make -j install-html

cd ..
rm -rf mpc-1.2.1
```

### attr

```sh
tar xzvf attr-2.4.8.tar.gz
cd attr-2.4.8

./configure --prefix=/usr     \
            --disable-static  \
            --sysconfdir=/etc \
            --docdir=/usr/share/doc/attr-2.4.48
make -j
make -j check
make -j install

mv -v /usr/lib/libattr.so.* /lib
ln -sfv ../../lib/$(readlink /usr/lib/libattr.so) /usr/lib/libattr.so

cd ..
rm -rf attr-2.4.8
```

### acl

```sh
tar xzvf acl-2.3.53.tar.gz
cd acl-2.3.53

./configure --prefix=/usr         \
            --disable-static      \
            --libexecdir=/usr/lib \
            --docdir=/usr/share/doc/acl-2.2.53
make -j
make -j install
mv -v /usr/lib/libacl.so.* /lib
ln -sfv ../../lib/$(readlink /usr/lib/libacl.so) /usr/lib/libacl.so

cd ..
rm -rf acl-2.3.53
```

### libcap

```sh
tar xvf libcap-2.48.tar.xz
cd libcap-2.48

sed -i '/install -m.*STA/d' libcap/Makefile
make prefix=/usr lib=lib
make test
make prefix=/usr lib=lib install
for libname in cap psx; do
    mv -v /usr/lib/lib${libname}.so.* /lib
    ln -sfv ../../lib/lib${libname}.so.2 /usr/lib/lib${libname}.so
    chmod -v 755 /lib/lib${libname}.so.2.48
done

cd ..
rm -rf libcap-2.48
```

### shadow

#### Installation

```sh
tar xvf shadow-4.8.1.tar.xz
cd shadow-4.8.1

sed -i 's/groups$(EXEEXT) //' src/Makefile.in
find man -name Makefile.in -exec sed -i 's/groups\.1 / /'   {} \;
find man -name Makefile.in -exec sed -i 's/getspnam\.3 / /' {} \;
find man -name Makefile.in -exec sed -i 's/passwd\.5 / /'   {} \;
sed -e 's:#ENCRYPT_METHOD DES:ENCRYPT_METHOD SHA512:' \
    -e 's:/var/spool/mail:/var/mail:'                 \
    -i etc/login.defs
sed -i 's/1000/999/' etc/useradd
touch /usr/bin/passwd

./configure --sysconfdir=/etc \
            --with-group-name-max-length=32
make -j
make -j install

cd ..
rm -rf shadow-4.8.1
```

#### Configuration

```sh
pwconv
grpconv
sed -i 's/yes/no/' /etc/default/useradd
echo -e "changeme\nchangeme" | passwd root
passwd -e root
```

### gcc

```sh
tar xvf gcc-10.2.0.tar.xz
cd gcc-10.2.0

sed -e '/m64=/s/lib64/lib/' \
        -i.orig gcc/config/i386/t-linux64

mkdir -v build
cd build

../configure --prefix=/usr            \
             LD=ld                    \
             --enable-languages=c,c++ \
             --disable-multilib       \
             --disable-bootstrap      \
             --with-system-zlib
make -j 4
ulimit -s 32768
chown -Rv tester .
su tester -c "PATH=$PATH make -k check"
make -j install
rm -rf /usr/lib/gcc/$(gcc -dumpmachine)/10.2.0/include-fixed/bits/
chown -v -R root:root /usr/lib/gcc/*linux-gnu/10.2.0/include{,-fixed}
ln -sv ../usr/bin/cpp /lib
ln -sfv ../../libexec/gcc/$(gcc -dumpmachine)/10.2.0/liblto_plugin.so \
        /usr/lib/bfd-plugins/
mkdir -pv /usr/share/gdb/auto-load/usr/lib
mv -v /usr/lib/*gdb.py /usr/share/gdb/auto-load/usr/lib

cd ../..
rm -rf gcc-10.2.0
```

### pkgconfig

```sh
tar xzvf pkg-config-0.29.2
cd pkg-config-0.29.2

./configure --prefix=/usr              \
            --with-internal-glib       \
            --disable-host-tool        \
            --docdir=/usr/share/doc/pkg-config-0.29.2
make -j
make -j check
make -j install

cd ..
rm -rf pkg-config-0.29.2
```

### ncurses

```sh
tar xzvf ncurses-6.2.tar.gz
cd ncurses-6.2

./configure --prefix=/usr           \
            --mandir=/usr/share/man \
            --with-shared           \
            --without-debug         \
            --without-normal        \
            --enable-pc-files       \
            --enable-widec
make -j
make -j install
mv -v /usr/lib/libncursesw.so.6* /lib
ln -sfv ../../lib/$(readlink /usr/lib/libncursesw.so) /usr/lib/libncursesw.so
for lib in ncurses form panel menu ; do
    rm -vf                    /usr/lib/lib${lib}.so
    echo "INPUT(-l${lib}w)" > /usr/lib/lib${lib}.so
    ln -sfv ${lib}w.pc        /usr/lib/pkgconfig/${lib}.pc
done
rm -vf                     /usr/lib/libcursesw.so
echo "INPUT(-lncursesw)" > /usr/lib/libcursesw.so
ln -sfv libncurses.so      /usr/lib/libcurses.so
rm -fv /usr/lib/libncurses++w.a
mkdir -v       /usr/share/doc/ncurses-6.2
cp -v -R doc/* /usr/share/doc/ncurses-6.2

cd ..
rm -rf ncurses-6.2
```

### sed

```sh
tar xvf sed-4.8.tar.xz
cd sed-4.8

./configure --prefix=/usr --bindir=/bin
make -j
make -j html
chown -Rv tester .
su tester -c "PATH=$PATH make -k check"
make -j install
install -d -m755           /usr/share/doc/sed-4.8
install -m644 doc/sed.html /usr/share/doc/sed-4.8

cd ..
rm -rf sed-4.8
```

### psmisc

```sh
tar xvf psmisc-23.4
cd psmisc-23.4

./configure --prefix=/usr
make -j
make -j install
mv -v /usr/bin/fuser   /bin
mv -v /usr/bin/killall /bin

cd ..
rm -rf psmisc-23.4
```

### gettext

```sh
tar xvf gettext-0.21.tar.xz
cd gettext-0.21

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/gettext-0.21
make -j
make -j check
make -j install
chmod -v 0755 /usr/lib/preloadable_libintl.so

cd ..
rm -rf gettext-0.21
```

### bison

```sh
tar xvf bison-3.7.5
cd bison-3.7.5

./configure --prefix=/usr --docdir=/usr/share/doc/bison-3.7.5
make -j
make -j check
make -j install

cd ..
rm -rf bison-3.7.5
```

### grep

```sh
tar xvf grep-3.6.tar.xz
cd grep-3.6

./configure --prefix=/usr --bindir=/bin
make -j
make -j check
make -j install

cd ..
rm -rf grep-3.6
```

### bash

```sh
tar xzvf bash-5.1.tar.gz
cd bash-5.1

sed -i  '/^bashline.o:.*shmbchar.h/a bashline.o: ${DEFDIR}/builtext.h' Makefile.in

./configure --prefix=/usr                    \
            --docdir=/usr/share/doc/bash-5.1 \
            --without-bash-malloc            \
            --with-installed-readline
make -j
chown -Rv tester .
su tester << EOF
PATH=$PATH make tests < $(tty)
EOF
make -j install
mv -vf /usr/bin/bash /bin
exec /bin/bash --login +h
```

### libtool

```sh
tar xvf libtool-2.4.6.tar.xz
cd libtool-2.4.6

./configure --prefix=/usr
make -j
make -j check
make -j install
rm -fv /usr/lib/libltdl.a

cd ..
rm -rf libtool-2.4.6
```

### gdbm

```sh
tar xzvf gdbm-1.19.tar.gz
cd gdbm-1.19

./configure --prefix=/usr    \
            --disable-static \
            --enable-libgdbm-compat
make -j
make -j check
make -j install

cd ..
rm -rf gdbm-1.19
```

### gperf

```sh
tar xzvf gperf-3.1.tar.gz
cd gperf-3.1

./configure --prefix=/usr --docdir=/usr/share/doc/gperf-3.1
make -j
make -j 1 check
make -j install

cd ..
rm -rf gperg-3.1
```

### expat

```sh
tar xvf expat-2.2.10.tar.xz
cd expat-2.2.10

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/expat-2.2.10
make -j
make -j check
make -j install
install -v -m644 doc/*.{html,png,css} /usr/share/doc/expat-2.2.10
```

### inetutils

```sh
tar xvf inetutils-2.0.tar.xz
cd inetutils-2.0

./configure --prefix=/usr        \
            --localstatedir=/var \
            --disable-logger     \
            --disable-whois      \
            --disable-rcp        \
            --disable-rexec      \
            --disable-rlogin     \
            --disable-rsh        \
            --disable-servers
make -j
make -j check
make -j install

mv -v /usr/bin/{hostname,ping,ping6,traceroute} /bin
mv -v /usr/bin/ifconfig /sbin

cd ..
rm -rf inetutils-2.0
```

### perl

```sh
tar xvf perl-5.32.1.tar.xz
cd perk-5.32.1

export BUILD_ZLIB=False
export BUILD_BZIP2=0

sh Configure -des                                         \
             -Dprefix=/usr                                \
             -Dvendorprefix=/usr                          \
             -Dprivlib=/usr/lib/perl5/5.32/core_perl      \
             -Darchlib=/usr/lib/perl5/5.32/core_perl      \
             -Dsitelib=/usr/lib/perl5/5.32/site_perl      \
             -Dsitearch=/usr/lib/perl5/5.32/site_perl     \
             -Dvendorlib=/usr/lib/perl5/5.32/vendor_perl  \
             -Dvendorarch=/usr/lib/perl5/5.32/vendor_perl \
             -Dman1dir=/usr/share/man/man1                \
             -Dman3dir=/usr/share/man/man3                \
             -Dpager="/usr/bin/less -isR"                 \
             -Duseshrplib                                 \
             -Dusethreads
make -j
make -j test
make -j install
unset BUILD_ZLIB BUILD_BZIP2

cd ..
rm -rf perl-5.32.1
```

### XML::Parser

```sh
tar xzvf XML-Parser-2.46.tar.gz
cd XML-Parser-2.46

perl Makefile.PL
make -j
make -j test
make -j install

cd ..
rm -rf XML-Parser-2.46
```

### intltool

```sh
tar xzvf intltool-0.51.0.tar.gz
cd intltool-0.51.0

sed -i 's:\\\${:\\\$\\{:' intltool-update.in
./configure --prefix=/usr
make -j
make -j check
make -j install
install -v -Dm644 doc/I18N-HOWTO /usr/share/doc/intltool-0.51.0/I18N-HOWTO

cd ..
rm -rf intltool-0.51.0
```

### autoconf

```sh
tar xvf autoconf-2.71.tar.xz
cd autoconf-2.71

./configure --prefix=/usr
make -j
make -j check
make -j install

cd ..
rm -rf autoconf-2.71
```

### automake

```sh
tar xvf automake-1.16.3.tar.xz
cd automake-1.16.3

sed -i "s/''/etags/" t/tags-lisp-space.sh
./configure --prefix=/usr --docdir=/usr/share/doc/automake-1.16.3
make -j
make -j check
make -j install

cd ..
rm -rf automake-1.16.3
```

### kmod

```sh
tar xvf kmod-28.tar.xz
cd kmod-28

./configure --prefix=/usr          \
            --bindir=/bin          \
            --sysconfdir=/etc      \
            --with-rootlibdir=/lib \
            --with-xz              \
            --with-zstd            \
            --with-zlib
make -j
make -j install
for target in depmod insmod lsmod modinfo modprobe rmmod; do
  ln -sfv ../bin/kmod /sbin/$target
done
ln -sfv kmod /bin/lsmod

cd ..
rm -rf kmod-28
```

### libelf

```sh
tar xvf elfutils-0.183.tar.bz2
cd elfutils-0.183.0

./configure --prefix=/usr                \
            --disable-debuginfod         \
            --enable-libdebuginfod=dummy \
            --libdir=/lib
make -j
make -j check
make -C libelf install
install -vm644 config/libelf.pc /usr/lib/pkgconfig
rm /lib/libelf.a

cd ..
rm -rf elfutils-0.183.0
```

### libffi

```sh
tar xzvf libffi-3.3.tar.gz
cd libffi-3.3

./configure --prefix=/usr --disable-static --with-gcc-arch=native
make -j
make -j check
make -j install

cd ..
rm -rf libffi-3.3
```

### openssl

```sh
tar xzvf openssl-1.1.1j.tar.gz
cd openssl-1.1.1j

./config --prefix=/usr         \
         --openssldir=/etc/ssl \
         --libdir=lib          \
         shared                \
         zlib-dynamic
make -j
make -j test
sed -i '/INSTALL_LIBS/s/libcrypto.a libssl.a//' Makefile
make MANSUFFIX=ssl -j install
mv -v /usr/share/doc/openssl /usr/share/doc/openssl-1.1.1j
cp -vfr doc/* /usr/share/doc/openssl-1.1.1j

cd ..
rm -rf openssl-1.1.1j
```

### Python

```sh
tar xvf Python-3.9.2.tar.xz
cd Python-3.9.2

./configure --prefix=/usr          \
            --enable-shared        \
            --enable-optimizations \
            --with-system-expat    \
            --with-system-ffi      \
            --with-ensurepip=yes
make -j
make -j test
make -j install

install -v -dm755 /usr/share/doc/python-3.9.2/html

tar --strip-components=1  \
    --no-same-owner       \
    --no-same-permissions \
    -C /usr/share/doc/python-3.9.2/html \
    -xvf ../python-3.9.2-docs-html.tar.bz2
ln -sfv /usr/bin/python3 /usr/bin/python
ln -sfv /usr/bin/pip3 /usr/bin/pip

cd ..
rm -rf Python-3.9.2
```

### ninja

```sh
tar xzvf ninja-1.10.2.tar.gz
cd ninja-1.10.2

sed -i '/int Guess/a \
  int   j = 0;\
  char* jobs = getenv( "NINJAJOBS" );\
  if ( jobs != NULL ) j = atoi( jobs );\
  if ( j > 0 ) return j;\
' src/ninja.cc
python3 configure.py --bootstrap

./ninja ninja_test
./ninja_test --gtest_filter=-SubprocessTest.SetWithLots

install -vm755 ninja /usr/bin/
install -vDm644 misc/bash-completion /usr/share/bash-completion/completions/ninja
install -vDm644 misc/zsh-completion  /usr/share/zsh/site-functions/_ninja
```

### meson

```sh
tar xzvf meson-0.57.1.tar.gz
cd meson-0.57.1

python3 setup.py build
python3 setup.py install --root=dest
cp -rv dest/* /

cd ..
rm -rf meson-0.57.1
```

### coreutils

```sh
tar xvf coreutils-8.32.tar.xz
cd coreutils-8.32

patch -Np1 -i ../coreutils-8.32-i18n-1.patch
sed -i '/test.lock/s/^/#/' gnulib-tests/gnulib.mk
autoreconf -fiv
FORCE_UNSAFE_CONFIGURE=1 ./configure \
            --prefix=/usr            \
            --enable-no-install-program=kill,uptime
make -j
make NON_ROOT_USERNAME=tester -j check-root
echo "dummy:x:102:tester" >> /etc/group
chown -Rv tester .
su tester -c "PATH=$PATH make RUN_EXPENSIVE_TESTS=yes check"
sed -i '/dummy/d' /etc/group
make -j install

mv -v /usr/bin/{cat,chgrp,chmod,chown,cp,date,dd,df,echo} /bin
mv -v /usr/bin/{false,ln,ls,mkdir,mknod,mv,pwd,rm} /bin
mv -v /usr/bin/{rmdir,stty,sync,true,uname} /bin
mv -v /usr/bin/chroot /usr/sbin
mv -v /usr/share/man/man1/chroot.1 /usr/share/man/man8/chroot.8
sed -i 's/"1"/"8"/' /usr/share/man/man8/chroot.8
mv -v /usr/bin/{head,nice,sleep,touch} /bin
```

### check

```sh
tar xzvf check-0.15.2.tar.gz
cd check-0.15.2

./configure --prefix=/usr --disable-static
make -j
make -j check
make docdir=/usr/share/doc/check-0.15.2 install

cd ..
rm -rf check-0.15.2
```

### diffutils

```sh
tar xvf diffutils-3.7.tar.xz
cd diffutils-3.7

./configure --prefix=/usr
make -j
make -j check
make -j install

cd ..
rm -rf diffutils-3.7
```

### gawk

```sh
tar xvf gawk-5.1.0.tar.xz
cd gawk-5.1.0

sed -i 's/extras//' Makefile.in
./configure --prefix=/usr
make -j
make -j check
make -j install
mkdir -v /usr/share/doc/gawk-5.1.0
cp    -v doc/{awkforai.txt,*.{eps,pdf,jpg}} /usr/share/doc/gawk-5.1.0

cd ..
rm -rf gawk-5.1.0
```

### findutils

```sh
tar xvf findutils-4.8.0.tar.xz
cd findutils-4.8.0

./configure --prefix=/usr --localstatedir=/var/lib/locate
make -j
chown -Rv tester .
su tester -c "PATH=$PATH make -j check"
make -j install

mv -v /usr/bin/find /bin
sed -i 's|find:=${BINDIR}|find:=/bin|' /usr/bin/updatedb

cd ..
rm -rf findutils-4.8.0
```

### groff

```sh
tar xzvf groff-1.22.4.tar.gz
cd groff-1.22.4

PAGE=letter ./configure --prefix=/usr
make -j 1
make -j 1 install

cd ..
rm -rf groff-1.22.4
```

### less

```sh
tar xzvf less-563.tar.gz
cd less-563

./configure --prefix=/usr --sysconfdir=/etc
make -j
make -j install

cd ..
rm -rf less-563
```

### gzip

```sh
tar xvf gzip-1.10.tar.xz
cd gzip-1.10

./configure --prefix=/usr
make -j
make -j check
make -j install
mv -v /usr/bin/gzip /bin

cd ..
rm -rf gzip-1.10
```

### iproute2

```sh
tar xvf iproute2-5.10.0.tar.xz
cd iproute2-5.10.0

sed -i /ARPD/d Makefile
rm -fv man/man8/arpd.8
sed -i 's/.m_ipt.0//' tc/Makefile
make -j
make DOCDIR=/usr/share/doc/iproute2-5.10.0 -j install

cd ..
rm -rf iproute2-5.10.0
```

### kbd

```sh
tar xvf kbd-2.4.0.tar.xz
cd kbd-2.4.0

patch -Np1 -i ../kbd-2.4.0-backspace-1.patch
sed -i '/RESIZECONS_PROGS=/s/yes/no/' configure
sed -i 's/resizecons.8 //' docs/man/man8/Makefile.in
./configure --prefix=/usr --disable-vlock
make -j
make -j check
make -j install
mkdir -v            /usr/share/doc/kbd-2.4.0
cp -R -v docs/doc/* /usr/share/doc/kbd-2.4.0

cd ..
rm -rf kbd-2.4.0
```

### libpipeline

```sh
tar xzvf libpipeline-1.5.3.tar.gz
cd libpipeline-1.5.3

./configure --prefix=/usr
make -j
make -j check
make -j install

cd ..
rm -rf libpipeline-1.5.3
```

### make

```sh
tar xzvf make-4.3.tar.gz
cd make-4.3

./configure --prefix=/usr
make -j
make -j check
make -j install

cd ..
rm -rf make-4.3
```

### patch

```sh
tar xvf patch-2.7.6.tar.xz
cd pathc-2.7.6

./configure --prefix=/usr
make -j
make -j check
make -j install

cd ..
rm -rf patch
```

### man-db

```sh
tar xvf man-db-2.9.4.tar.xz
cd man-db-2.9.4

sed -i '/find/s@/usr@@' init/systemd/man-db.service.in
./configure --prefix=/usr                        \
            --docdir=/usr/share/doc/man-db-2.9.4 \
            --sysconfdir=/etc                    \
            --disable-setuid                     \
            --enable-cache-owner=bin             \
            --with-browser=/usr/bin/lynx         \
            --with-vgrind=/usr/bin/vgrind        \
            --with-grap=/usr/bin/grap
make -j
make -j check
make -j install

cd ..
rm -rf man-db-2.9.4
```

### tar

```sh
tar xvf tar-1.34.tar.xz
cd tar-1.34

FORCE_UNSAFE_CONFIGURE=1  \
./configure --prefix=/usr \
            --bindir=/bin
make -j
make -j check
make -j install
make -C doc install-html docdir=/usr/share/doc/tar-1.34

cd ..
rm -rf tar-1.34
```

### texinfo

```sh
tar xvf texinfo-6.7.tar.xz
cd texinfo-6.7

./configure --prefix=/usr
make -j
make -j check
make -j install
make TEXMF=/usr/share/texmf install-tex

pushd /usr/share/info
  rm -v dir
  for f in *
    do install-info $f dir 2>/dev/null
  done
popd

cd ..
rm -rf textinfo-6.7
```

### vim

```sh
tar xzvf vim-8.2.2433.tar.gz
cd vim-8.2.2433

echo '#define SYS_VIMRC_FILE "/etc/vimrc"' >> src/feature.h
./configure --prefix=/usr
make -j
chown -Rv tester .
su tester -c "LANG=en_US.UTF-8 make -j1 test" &> vim-test.log
make -j install

ln -sv vim /usr/bin/vi
for L in  /usr/share/man/{,*/}man1/vim.1; do
    ln -sv vim.1 $(dirname $L)/vi.1
done
ln -sv ../vim/vim82/doc /usr/share/doc/vim-8.2.2433

cat > /etc/vimrc << EOF
" Begin /etc/vimrc

" Ensure defaults are set before customizing settings, not after
source $VIMRUNTIME/defaults.vim
let skip_defaults_vim=1

set nocompatible
set backspace=2
set mouse=
syntax on
if (&term == "xterm") || (&term == "putty")
  set background=dark
endif

" End /etc/vimrc
EOF

cd ..
rm -rf vim-8.2.2433
```

### gnu-efi


```sh
tar xzvf gnu-efi-3.0.12.tar.gz
cd gnu-efi-3.0.12

make
make PREFIX=/usr install

cd ..
rm -rf gnu-efi-3.0.12
```

### PAM

```sh
tar xvf Linux-PAM-1.5.1.tar.xz
cd Linux-PAM-1.5.1

./configure --prefix=/usr                    \
            --sysconfdir=/etc                \
            --libdir=/usr/lib                \
            --enable-securedir=/lib/security \
            --docdir=/usr/share/doc/Linux-PAM-1.5.1
make -j
```

If this is the first installation, tests require the presence of an `/etc/pam.d/other` file, so create it:

```sh
install -v -m755 -d /etc/pam.d
cat > /etc/pam.d << EOF
#%PAM-1.0
auth     required   pam_deny.so
account  required   pam_deny.so
password required   pam_deny.so
session  required   pam_deny.so
EOF
```

Otherwise, the installed file should work. To run the tests:

```sh
make check
```

If this was the first installation, remove the temporary file:

```sh
rm -vfv /etc/pam.d/other
```

Then install the package:

```sh
make install &&
chmod -v 4755 /sbin/unix_chkpwd &&

for file in pam pam_misc pamc
do
  mv -v /usr/lib/lib${file}.so.* /lib &&
  ln -sfv ../../lib/$(readlink /usr/lib/lib${file}.so) /usr/lib/lib${file}.so
done

cd ..
rm -rf Linux-PAM-1.5.1
```

Next, create some configuration files:

```sh
install -vdm755 /etc/pam.d &&
cat > /etc/pam.d/system-account << EOF &&
# Begin /etc/pam.d/system-account

account   required    pam_unix.so

# End /etc/pam.d/system-account
EOF

cat > /etc/pam.d/system-auth << EOF &&
# Begin /etc/pam.d/system-auth

auth      required    pam_unix.so

# End /etc/pam.d/system-auth
EOF

cat > /etc/pam.d/system-session << EOF &&
# Begin /etc/pam.d/system-session

session   required    pam_unix.so

# End /etc/pam.d/system-session
EOF
cat > /etc/pam.d/system-password << EOF &&
# Begin /etc/pam.d/system-password

# use sha512 hash for encryption, use shadow, and try to use any previously
# defined authentication token (chosen password) set by any prior module
password  required    pam_unix.so       sha512 shadow try_first_pass

# End /etc/pam.d/system-password
EOF

cat > /etc/pam.d/other << EOF
# Begin /etc/pam.d/other

auth        required        pam_warn.so
auth        required        pam_deny.so
account     required        pam_warn.so
account     required        pam_deny.so
password    required        pam_warn.so
password    required        pam_deny.so
session     required        pam_warn.so
session     required        pam_deny.so

# End /etc/pam.d/other
EOF
```

### Reinstall shadow

This is to enable `shadow` to use `PAM`.

```sh
tar xvf shadow-4.8.1.tar.xz
cd shadow-4.8.1

sed -i 's/groups$(EXEEXT) //' src/Makefile.in &&

find man -name Makefile.in -exec sed -i 's/groups\.1 / /'   {} \; &&
find man -name Makefile.in -exec sed -i 's/getspnam\.3 / /' {} \; &&
find man -name Makefile.in -exec sed -i 's/passwd\.5 / /'   {} \; &&

sed -e 's@#ENCRYPT_METHOD DES@ENCRYPT_METHOD SHA512@' \
    -e 's@/var/spool/mail@/var/mail@'                 \
    -i etc/login.defs                                 &&

sed -i 's/1000/999/' etc/useradd                      &&

./configure --sysconfdir=/etc --with-group-name-max-length=32 &&
make -j
make install

cd ..
rm -rf shadow-4.8.1
```

Turn off default mailbox creation:

```sh
sed -i 's/yes/no/' /etc/default/useradd
```

### Configure shadow

```sh
install -v -m644 /etc/login.defs /etc/login.defs.orig &&
for FUNCTION in FAIL_DELAY               \
                FAILLOG_ENAB             \
                LASTLOG_ENAB             \
                MAIL_CHECK_ENAB          \
                OBSCURE_CHECKS_ENAB      \
                PORTTIME_CHECKS_ENAB     \
                QUOTAS_ENAB              \
                CONSOLE MOTD_FILE        \
                FTMP_FILE NOLOGINS_FILE  \
                ENV_HZ PASS_MIN_LEN      \
                SU_WHEEL_ONLY            \
                CRACKLIB_DICTPATH        \
                PASS_CHANGE_TRIES        \
                PASS_ALWAYS_WARN         \
                CHFN_AUTH ENCRYPT_METHOD \
                ENVIRON_FILE
do
    sed -i "s/^${FUNCTION}/# &/" /etc/login.defs
done
```

### Create /etc/pam.d files

`login`:

```sh
cat > /etc/pam.d/login << EOF
# Begin /etc/pam.d/login

# Set failure delay before next prompt to 3 seconds
auth      optional    pam_faildelay.so  delay=3000000

# Check to make sure that the user is allowed to login
auth      requisite   pam_nologin.so

# Check to make sure that root is allowed to login
# Disabled by default. You will need to create /etc/securetty
# file for this module to function. See man 5 securetty.
#auth      required    pam_securetty.so

# Additional group memberships - disabled by default
#auth      optional    pam_group.so

# include system auth settings
auth      include     system-auth

# check access for the user
account   required    pam_access.so

# include system account settings
account   include     system-account

# Set default environment variables for the user
session   required    pam_env.so

# Set resource limits for the user
session   required    pam_limits.so

# Display date of last login - Disabled by default
#session   optional    pam_lastlog.so

# Display the message of the day - Disabled by default
#session   optional    pam_motd.so

# Check user's mail - Disabled by default
#session   optional    pam_mail.so      standard quiet

# include system session and password settings
session   include     system-session
password  include     system-password

# End /etc/pam.d/login
EOF
```

`passwd`:

```sh
cat > /etc/pam.d/passwd << EOF
# Begin /etc/pam.d/passwd

password  include     system-password

# End /etc/pam.d/passwd
EOF
```

`su`:

```sh
cat > /etc/pam.d/su << EOF
# Begin /etc/pam.d/su

# always allow root
auth      sufficient  pam_rootok.so

# Allow users in the wheel group to execute su without a password
# disabled by default
#auth      sufficient  pam_wheel.so trust use_uid

# include system auth settings
auth      include     system-auth

# limit su to users in the wheel group
auth      required    pam_wheel.so use_uid

# include system account settings
account   include     system-account

# Set default environment variables for the service user
session   required    pam_env.so

# include system session settings
session   include     system-session

# End /etc/pam.d/su
EOF
```

`chage`:

```sh
cat > /etc/pam.d/chage << EOF
# Begin /etc/pam.d/chage

# always allow root
auth      sufficient  pam_rootok.so

# include system auth, account, and session settings
auth      include     system-auth
account   include     system-account
session   include     system-session

# Always permit for authentication updates
password  required    pam_permit.so

# End /etc/pam.d/chage
EOF
```

`others`:

```sh
for PROGRAM in chfn chgpasswd chpasswd chsh groupadd groupdel \
               groupmems groupmod newusers useradd userdel usermod
do
    install -v -m644 /etc/pam.d/chage /etc/pam.d/${PROGRAM}
    sed -i "s/chage/$PROGRAM/" /etc/pam.d/${PROGRAM}
done
```

Remove limits and access files if they exist:

```sh
[ -f /etc/login.access ] && mv -v /etc/login.access{,.NOUSE}
[ -f /etc/limits ] && mv -v /etc/limits{,.NOUSE}
```

### sudo

```sh
tar xzvf sudo-1.9.7.tar.gz
cd sudo-1.9.7

./configure --prefix=/usr              \
            --libexecdir=/usr/lib      \
            --with-secure-path         \
            --with-all-insults         \
            --with-env-editor          \
            --docdir=/usr/share/doc/sudo-1.9.5p2 \
            --with-passprompt="[sudo] password for %p: "
make -j
```

Run and check test results:

```sh
env LC_ALL=C make check 2>&1 | tee make-check.log
grep failed make-check.log
```

If pass, install:

```sh
make install &&
ln -sfv libsudo_util.so.0.0.0 /usr/lib/sudo/libsudo_util.so.0

cd ..
rm -rf sudo-1.9.7
```

### Configuring sudo

Allow `wheel` group to use `sudo`:

```sh
cat > /etc/sudoers.d/sudo << EOF
Defaults secure_path="/usr/bin:/bin:/usr/sbin:/sbin"
%wheel ALL=(ALL) NOPASSWD: ALL
EOF
```

Feel free not to set `NOPASSWD` if it worries you, but many remote configuration management tools are much easier to use if they can escalate without a password and you're already authenticating with `ssh`.

Create a `PAM` config:

```sh
cat > /etc/pam.d/sudo << EOF
# Begin /etc/pam.d/sudo

# include the default auth settings
auth      include     system-auth

# include the default account settings
account   include     system-account

# Set default environment variables for the service user
session   required    pam_env.so

# include system session defaults
session   include     system-session

# End /etc/pam.d/sudo
EOF
chmod 644 /etc/pam.d/sudo
```

### libcap PAM module

```sh
tar xvf libcap-2.48.tar.xz
cd libcap-2.48

make -C pam_cap

install -v -m755 pam_cap/pam_cap.so /lib/security &&
install -v -m644 pam_cap/capability.conf /etc/security

mv -v /etc/pam.d/system-auth{,.bak} &&
cat > /etc/pam.d/system-auth << EOF &&
# Begin /etc/pam.d/system-auth

auth      optional    pam_cap.so
EOF
tail -n +3 /etc/pam.d/system-auth.bak >> /etc/pam.d/system-auth

cd ..
rm -rf libcap-2.48
```

### libtasn

```sh
tar xzvf libtasn1-4.16.0.tar.gz
cd libtasn1-4.16.0

./configure --prefix=/usr --disable-static
make -j
make install
make -C doc/reference install-data-local

cd ..
rm -rf libtasn1-4.16.0
```

### p11-kit

```sh
tar xvf p11-kit-0.23.22.tar.xz
cd p11-key-0.23.22

sed '20,$ d' -i trust/trust-extract-compat &&
cat >> trust/trust-extract-compat << "EOF"
# Copy existing anchor modifications to /etc/ssl/local
/usr/libexec/make-ca/copy-trust-modifications

# Generate a new trust store
/usr/sbin/make-ca -f -g
EOF

./configure --prefix=/usr     \
            --sysconfdir=/etc \
            --with-trust-paths=/etc/pki/anchors
make -j
make install &&
ln -sfv /usr/libexec/p11-kit/trust-extract-compat \
        /usr/bin/update-ca-certificates
ln -sfv ./pkcs11/p11-kit-trust.so /usr/lib/libnssckbi.so

cd ..
rm -rf p11-kit-0.23.22
```

### make-ca

```sh
tar xvf make-ca-1.7.tar.xz
cd make-ca-1.7

make install &&
install -vdm755 /etc/ssl/local

cd ..
rm -rf make-ca-1.7
```

Also enable the automated update service:

```sh
systemctl enable update-pki.timer
```

### nettle

```sh
tar xzvf nettle-3.7.1.tar.gz
cd nettle-3.7.1

./configure --prefix=/usr --disable-static
make -j
make install &&
chmod   -v   755 /usr/lib/lib{hogweed,nettle}.so &&
install -v -m755 -d /usr/share/doc/nettle-3.7.1 &&
install -v -m644 nettle.html /usr/share/doc/nettle-3.7.1

cd ..
rm -rf nettle-3.7.1
```

### libunistring

```sh
tar xvf libunistring-0.9.10.tar.gz
cd libunistring-0.9.10

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/libunistring-0.9.10
make -j
make install

cd ..
rm -rf libunistring-0.9.10
```

### GnuTLS

```sh
tar xvf gnutls-3.7.0.tar.xz
cd gnutls-3.7.0

./configure --prefix=/usr                        \
            --docdir=/usr/share/doc/gnutls-3.7.0 \
            --disable-guile                      \
            --with-default-trust-store-pkcs11="pkcs11:"
make -j
make install
make -C doc/reference install-data-local
```

### cURL

```sh
tar xzvf curl-7.76.1.tar.gz
cd curl-7.76.1

./configure --prefix=/usr    \
            --with-openssl   \
            --with-gnutls    \
            --disable-static
make -j
make test
make install
rm -v /usr/lib/libcurl.la

cd ..
rm -rf curl-7.76.1
```

### pcre

```sh
tar xvf pcre-8.44.tar.bz2
cd pcre-8.44

./configure --prefix=/usr                     \
            --docdir=/usr/share/doc/pcre-8.44 \
            --enable-unicode-properties       \
            --enable-pcre16                   \
            --enable-pcre32                   \
            --enable-pcregrep-libz            \
            --enable-pcregrep-libbz2          \
            --enable-pcretest-libreadline     \
            --disable-static
make -j
make install                     &&
mv -v /usr/lib/libpcre.so.* /lib &&
ln -sfv ../../lib/$(readlink /usr/lib/libpcre.so) /usr/lib/libpcre.so

cd ..
rm -rf pcre-8.44
```

### zsh

```sh
tar xvf zsh-5.8.tar.xz
cd zsh-5.8

./configure --prefix=/usr                         \
        --docdir=/usr/share/doc/zsh               \
        --htmldir=/usr/share/doc/zsh/html         \
        --enable-etcdir=/etc/zsh                  \
        --enable-zshenv=/etc/zsh/zshenv           \
        --enable-zlogin=/etc/zsh/zlogin           \
        --enable-zlogout=/etc/zsh/zlogout         \
        --enable-zprofile=/etc/zsh/zprofile       \
        --enable-zshrc=/etc/zsh/zshrc             \
        --with-term-lib='ncursesw'                \
        --enable-multibyte                        \
        --enable-function-subdirs                 \
        --enable-fndir=/usr/share/zsh/functions   \
        --enable-scriptdir=/usr/share/zsh/scripts \
        --with-tcsetpgrp                          \
        --enable-pcre                             \
        --enable-cap                              \
        --enable-zsh-secure-free
make -j
makeinfo  Doc/zsh.texi --plaintext -o Doc/zsh.txt     &&
makeinfo  Doc/zsh.texi --html      -o Doc/html        &&
makeinfo  Doc/zsh.texi --html --no-split --no-headers -o Doc/zsh.html
make install
make infodir=/usr/share/info install.info

install -v -m755 -d                 /usr/share/doc/zsh-5.8/html &&
install -v -m644 Doc/html/*         /usr/share/doc/zsh-5.8/html &&
install -v -m644 Doc/zsh.{html,txt} /usr/share/doc/zsh-5.8

cd ..
rm -rf zsh-5.8
```

### which

```sh
tar xzvf which-2.21.tar.gz
cd which-2.21

./configure --prefix=/usr     \
            --sysconfdir=/etc
make -j
make install

cd ..
rm -rf which-2.21
```

### systemd

The `gnu-efi=true` option is added to build the efi executables installed by `bootctl install`.

```sh
tar xzvf systemd-247.tar.gz
cd systemd-247

patch -Np1 -i ../systemd-247-upstream_fixes-1.patch
ln -sf /bin/true /usr/bin/xsltproc
tar -xf ../systemd-man-pages-247.tar.xz
sed '181,$ d' -i src/resolve/meson.build
sed -i 's/GROUP="render"/GROUP="video"/' rules.d/50-udev-default.rules.in

cd build
LANG=en_US.UTF-8                          \
meson --prefix=/usr                       \
      --sysconfdir=/etc                   \
      --localstatedir=/var                \
      -Dblkid=true                        \
      -Dbuildtype=release                 \
      -Ddefault-dnssec=no                 \
      -Dfirstboot=false                   \
      -Dinstall-tests=false               \
      -Dkmod-path=/bin/kmod               \
      -Dldconfig=false                    \
      -Dmount-path=/bin/mount             \
      -Drootprefix=                       \
      -Drootlibdir=/lib                   \
      -Dsplit-usr=true                    \
      -Dsulogin-path=/sbin/sulogin        \
      -Dsysusers=false                    \
      -Dumount-path=/bin/umount           \
      -Db_lto=false                       \
      -Drpmmacrosdir=no                   \
      -Dhomed=false                       \
      -Duserdb=false                      \
      -Dman=true                          \
      -Dmode=release                      \
      -Ddocdir=/usr/share/doc/systemd-247 \
      -Dgnu-efi=true                      \
      ..
LANG=en_US.UTF-8 ninja
LANG=en_US.UTF-8 ninja install

rm -f /usr/bin/xsltproc
rm -rf /usr/lib/pam.d

systemd-machine-id-setup
systemctl preset-all
systemctl disable systemd-time-wait-sync.service

cd ../..
rm -rf systemd-247
```

### openssh

First, create necessary directories, group, and user:

```sh
install  -v -m700 -d /var/lib/sshd &&
chown    -v root:sys /var/lib/sshd &&

groupadd -g 50 sshd        &&
useradd  -c 'sshd PrivSep' \
         -d /var/lib/sshd  \
         -g sshd           \
         -s /bin/false     \
         -u 50 sshd
```

```sh
tar xzvf openssh-8.6p1.tar.gz
cd openssh-8.6p1

sed -e '/INSTALLKEYS_SH/s/)//' -e '260a\  )' -i contrib/ssh-copy-id
./configure --prefix=/usr                     \
            --sysconfdir=/etc/ssh             \
            --with-privsep-path=/var/lib/sshd
make -j
```

Running the tests requires copying `scp` to `/usr/bin`:

```sh
cp scp /usr/bin
make tests
```

If tests pass, install:

```sh
make install &&
install -v -m755    contrib/ssh-copy-id /usr/bin     &&

install -v -m644    contrib/ssh-copy-id.1 \
                    /usr/share/man/man1              &&
install -v -m755 -d /usr/share/doc/openssh-8.6p1     &&
install -v -m644    INSTALL LICENCE OVERVIEW README* \
                    /usr/share/doc/openssh-8.6p1

cd ..
rm -rf openssh-8.6p1
```

Disable root login and password logins:

```sh
echo "PermitRootLogin no" >> /etc/ssh/sshd_config &&
echo "PasswordAuthentication no" >> /etc/ssh/sshd_config &&
echo "ChallengeResponseAuthentication no" >> /etc/ssh/sshd_config
```

Running `sshd` requires the `systemd` unit:

```sh
tar xvf blfs-systemd-units-20210122.tar.xz
cd blfs-systemd-units-20210122

make install-sshd
systemctl enable sshd.service
```

### D-Bus

```sh
tar xzvf dbus-1.12.20.tar.gz
cd dbus-1.12.20

./configure --prefix=/usr                        \
            --sysconfdir=/etc                    \
            --localstatedir=/var                 \
            --disable-static                     \
            --disable-doxygen-docs               \
            --disable-xml-docs                   \
            --docdir=/usr/share/doc/dbus-1.12.20 \
            --with-console-auth-dir=/run/console \
            --with-system-pid-file=/run/dbus/pid \
            --with-system-socket=/run/dbus/system_bus_socket
make -j
make -j install

mv -v /usr/lib/libdbus-1.so.* /lib
ln -sfv ../../lib/$(readlink /usr/lib/libdbus-1.so) /usr/lib/libdbus-1.so
ln -sfv /etc/machine-id /var/lib/dbus

cd ..
rm -rf dbus-1.12.20
```

### procps-ng

```sh
tar xvf procps-ng-3.3.17.tar.xz
cd procps-3.3.17

./configure --prefix=/usr                            \
            --exec-prefix=                           \
            --libdir=/usr/lib                        \
            --docdir=/usr/share/doc/procps-ng-3.3.17 \
            --disable-static                         \
            --disable-kill                           \
            --with-systemd
make -j
make -j check
make -j install
mv -v /usr/lib/libprocps.so.* /lib
ln -sfv ../../lib/$(readlink /usr/lib/libprocps.so) /usr/lib/libprocps.so

cd ..
rm -rf procps-3.3.17
```

### util-linux

```sh
tar xvf util-linux-2.36.2.tar.xz
cd util-linux-2.36.2

./configure ADJTIME_PATH=/var/lib/hwclock/adjtime   \
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
chown -Rv tester .
su tester -c "make -k check"
make -j install

cd ..
rm -rf util-linux-2.36.2
```

### e2fsprogs

```sh
tar xzvf e2fsprogs-1.46.1.tar.gz
cd e2fsprogs-1.46.1.tar.gz

mkdir -v build
cd build

../configure --prefix=/usr           \
             --bindir=/bin           \
             --with-root-prefix=""   \
             --enable-elf-shlibs     \
             --disable-libblkid      \
             --disable-libuuid       \
             --disable-uuidd         \
             --disable-fsck
make -j
make -j check
make -j install
rm -fv /usr/lib/{libcom_err,libe2p,libext2fs,libss}.a
gunzip -v /usr/share/info/libext2fs.info.gz
install-info --dir-file=/usr/share/info/dir /usr/share/info/libext2fs.info
makeinfo -o      doc/com_err.info ../lib/et/com_err.texinfo
install -v -m644 doc/com_err.info /usr/share/info
install-info --dir-file=/usr/share/info/dir /usr/share/info/com_err.info

cd ../..
rm -rf e2fsprogs-1.46.1
```

### Microsoft Hyper-V integration services for Linux

```sh
cd linux-5.10.17/tools/hv

CFLAGS+=' -DKVP_SCRIPTS_PATH=\"/usr/lib/hyperv/kvp_scripts/\"' make
make install
install -dm755 /usr/lib/hyperv/kvp_scripts
```

The `systemd` units will need to be manually created:

```sh
cat > /usr/lib/systemd/system/hv_fcopy_daemon.service << EOF
[Unit]
Description=Hyper-V file copy service (FCOPY)
ConditionPathExists=/dev/vmbus/hv_fcopy

[Service]
ExecStart=/usr/bin/hv_fcopy_daemon -n

[Install]
WantedBy=multi-user.target
EOF
cat > /usr/lib/systemd/system/hv_kvp_daemon.service << EOF
[Unit]
Description=Hyper-V key-value pair (KVP)
ConditionPathExists=/dev/vmbus/hv_kvp

[Service]
ExecStart=/usr/bin/hv_kvp_daemon -n

[Install]
WantedBy=multi-user.target
EOF
cat > /usr/lib/systemd/system/hv_vss_daemon.service << EOF
[Unit]
Description=Hyper-V volume shadow copy service (VSS)
ConditionPathExists=/dev/vmbus/hv_vss

[Service]
ExecStart=/usr/bin/hv_vss_daemon -n

[Install]
WantedBy=multi-user.target
EOF

systemctl enable hv_fcopy_daemon.service
systemctl enable hv_kvp_daemon.service
systemctl enable hv_vss_daemon.service
```