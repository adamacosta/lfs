# Build the LFS system

We are now ready to use our self-contained temporary toolchain to build the final LFS system. First, re-enter the chroot environment, either using the previously saved off script, or manually:

```sh
mount -v --bind /dev        $LFS/dev     &&
mount -v --bind /dev/pts    $LFS/dev/pts &&
mount -vt       proc proc   $LFS/proc    &&
mount -vt       sysfs sysfs $LFS/sys     &&
mount -vt       tmpfs tmpfs $LFS/run

chroot "$LFS" /usr/bin/env -i          \
    HOME=/root                         \
    TERM="$TERM"                       \
    PS1='(lfs chroot) \u:\w\$ '        \
    PATH=/bin:/usr/bin:/sbin:/usr/sbin \
    MAKEFLAGS="-j"                     \
    /bin/bash --login +h

cd /sources
```

If you created a separate build tree: `cd build_dir`.

## Install basic system software

### man-pages

```sh
tar xvf /sources/man-pages-5.11.tar.xz &&
cd       man-pages-5.11                &&

make install &&

cd .. &&
rm -rf man-pages-5.11
```

### iana-etc

```sh
tar xzvf /sources/iana-etc-20210407.tar.gz &&
cd        iana-etc-20210407                &&

cp services protocols /etc &&

cd .. &&
rm -rf iana-etc-20210407
```

### glibc

#### Build glibc

First is the initial compile and installation of the Name Service Cache Daemon and locales. Beware that at least a few tests are likely to fail here. You can check the LFS book for expected failures. Generally at least some of cpu feature tests are going to fail depending upon architecture. The install step will complain if `/etc/ld.so.conf` does not exist, so we will create it even though it will be empty for now.

```sh
tar xvf /sources/glibc-2.33.tar.xz &&
cd       glibc-2.33                &&

patch -Np1 -i ../../glibc-2.33-fhs-1.patch &&
sed -e '402a\      *result = local->data.services[database_index];' \
    -i nss/nss_database.c &&

mkdir -v build &&
cd       build &&

../configure --prefix=/usr                            \
             --disable-werror                         \
             --enable-kernel=3.2                      \
             --enable-stack-protector=strong          \
             --with-headers=/usr/include              \
             libc_cv_slibdir=/lib &&

make -j 4                                                           &&
make -j 4 check                                                     &&
touch /etc/ld.so.conf                                               &&
sed '/test-installation/s@$(PERL)@echo not running@' -i ../Makefile &&
make install                                                        &&

cp    -v ../nscd/nscd.conf /etc/nscd.conf &&
mkdir -pv /var/cache/nscd                 &&

install -v -Dm644 ../nscd/nscd.tmpfiles /usr/lib/tmpfiles.d/nscd.conf   &&
install -v -Dm644 ../nscd/nscd.service /lib/systemd/system/nscd.service &&

mkdir -pv /usr/lib/locale &&
make localedata/install-locales
```

#### Configure glibc

Set a configuration for the namespace switch service and add timezone data.

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

tar -xf ../../../tzdata2021a.tar.gz &&

ZONEINFO=/usr/share/zoneinfo
mkdir -pv $ZONEINFO/{posix,right} &&

for tz in etcetera southamerica northamerica europe africa antarctica  \
          asia australasia backward; do
    zic -L /dev/null   -d $ZONEINFO       ${tz} &&
    zic -L /dev/null   -d $ZONEINFO/posix ${tz} &&
    zic -L leapseconds -d $ZONEINFO/right ${tz}
done

cp  -v zone.tab zone1970.tab iso3166.tab $ZONEINFO &&
zic -d $ZONEINFO -p America/New_York               &&
unset ZONEINFO                                     &&

ln -sfv /usr/share/zoneinfo/America/Chicago /etc/localtime
```

#### Configure ld.so

Now we'll add the real loading linker search paths we want in addition to the defaults.

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

When complete, we will remove the build tree.

```sh
cd ../.. &&
rm -rf glibc-2.33
```

### zlib

```sh
tar xvf /sources/zlib-1.2.11.tar.xz &&
cd       zlib-1.2.11                &&

./configure --prefix=/usr &&

make         &&
make check   &&
make install &&

rm -fv /usr/lib/libz.a &&

cd .. &&
rm -rf zlib-1.2.11
```

### bzip2

```sh
tar xzvf /sources/bzip2-1.0.8.tar.gz &&
cd        bzip2-1.0.8                &&

patch -Np1 -i ../../bzip2-1.0.8-install_docs-1.patch     &&
sed -i 's@\(ln -s -f \)$(PREFIX)/bin/@\1@'   Makefile    &&
sed -i "s@(PREFIX)/man@(PREFIX)/share/man@g" Makefile    &&

make -f Makefile-libbz2_so &&
make clean                 &&
make                       &&
make PREFIX=/usr install   &&

cp -v  bzip2-shared /usr/bin/bzip2      &&
cp -av libbz2.so*   /usr/lib            &&
ln -sv libbz2.so.1.0 /usr/lib/libbz2.so &&
rm -v  /usr/bin/{bunzip2,bzcat}         &&
ln -sv bzip2 /usr/bin/bunzip2           &&
ln -sv bzip2 /usr/bin/bzcat             &&
rm -fv /usr/lib/libbz2.a                &&

cd .. &&
rm -rf bzip2-1.0.8
```

### xz

```sh
tar xvf /sources/xz-5.2.5.tar.xz &&
cd       xz-5.2.5                &&

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/xz-5.2.5 &&

make         &&
make install &&

cd .. &&
rm -rf xz-5.2.5
```

### zstd

Beware that at least one test of first block run length encoding fails. I'm not sure why this is.

```sh
tar xzvf $LFS/sources/zstd-1.4.9.tar.gz &&
cd        zstd-1.4.9                    &&

make                     &&
make check               &&
make prefix=/usr install &&

rm -v /usr/lib/libzstd.a &&

cd .. &&
rm -rf zstd-1.4.9
```

### file

```sh
tar xzvf $LFS/sources/file-5.40.tar.gz &&
cd        file-5.40                    &&

./configure --prefix=/usr &&

make         &&
make check   &&
make install &&

cd .. &&
rm -rf file-5.40
```

### readline

```sh
tar xzvf $LFS/sources/readline-8.1.tar.gz &&
cd        readline-8.1                    &&

sed -i '/MV.*old/d' Makefile.in              &&
sed -i '/{OLDSUFF}/c:' support/shlib-install &&

./configure --prefix=/usr    \
            --disable-static \
            --with-curses    \
            --docdir=/usr/share/doc/readline-8.1 &&

make SHLIB_LIBS="-lncursesw"         &&
make SHLIB_LIBS="-lncursesw" install &&

install -v -m644 doc/*.{ps,pdf,html,dvi} /usr/share/doc/readline-8.1 &&

cd .. &&
rm -rf readline-8.1
```

### m4

```sh
tar xvf $LFS/sources/m4-1.4.18.tar.xz &&
cd       m4-1.4.18                    &&

sed -i 's/IO_ftrylockfile/IO_EOF_SEEN/' lib/*.c        &&
echo "#define _IO_IN_BACKUP 0x100" >> lib/stdio-impl.h &&

./configure --prefix=/usr &&

make         &&
make check   &&
make install &&

cd .. &&
rm -rf m4-1.4.18
```

### bc

```sh
tar xvf $LFS/sources/bc-4.0.1.tar.xz &&
cd       bc-4.0.1                    &&

PREFIX=/usr CC=gcc ./configure.sh -G -O3 &&

make         &&
make test    &&
make install &&

cd .. &&
rm -rf bc-4.0.1
```

### flex

```sh
tar xzvf $LFS/sources/flex-2.6.4.tar.gz &&
cd        flex-2.6.4                    &&

./configure --prefix=/usr \
            --docdir=/usr/share/doc/flex-2.6.4 \
            --disable-static &&

make         &&
make check   &&
make install &&

ln -sv flex /usr/bin/lex &&

cd .. &&
rm -rf flex-2.6.4
```

### tcl

```sh
tar xzvf $LFS/sources/tcl8.6.11-src.tar.gz            &&
cd        tcl8.6.11                                   &&
tar xzvf ../tcl8.6.11-html.tar.gz --strip-component=1 &&

SRCDIR=$(pwd)
cd unix &&
./configure --prefix=/usr           \
            --mandir=/usr/share/man \
            --enable-64bit &&

make &&

sed -e "s|$SRCDIR/unix|/usr/lib|" \
    -e "s|$SRCDIR|/usr/include|"  \
    -i tclConfig.sh                 &&

sed -e "s|$SRCDIR/unix/pkgs/tdbc1.1.2|/usr/lib/tdbc1.1.2|" \
    -e "s|$SRCDIR/pkgs/tdbc1.1.2/generic|/usr/include|"    \
    -e "s|$SRCDIR/pkgs/tdbc1.1.2/library|/usr/lib/tcl8.6|" \
    -e "s|$SRCDIR/pkgs/tdbc1.1.2|/usr/include|"            \
    -i pkgs/tdbc1.1.2/tdbcConfig.sh &&

sed -e "s|$SRCDIR/unix/pkgs/itcl4.2.1|/usr/lib/itcl4.2.1|" \
    -e "s|$SRCDIR/pkgs/itcl4.2.1/generic|/usr/include|"    \
    -e "s|$SRCDIR/pkgs/itcl4.2.1|/usr/include|"            \
    -i pkgs/itcl4.2.1/itclConfig.sh &&

unset SRCDIR

make test                          &&
make install                       &&
chmod -v u+w /usr/lib/libtcl8.6.so &&
make install-private-headers       &&
ln -sfv tclsh8.6 /usr/bin/tclsh    &&

mv /usr/share/man/man3/{Thread,Tcl_Thread}.3 &&

cd ../.. &&
rm -rf tcl8.6.11
```

### expect

```sh
tar xzvf $LFS/sources/expect5.45.4.tar.gz &&
cd        expect5.45.4                    &&

./configure --prefix=/usr           \
            --with-tcl=/usr/lib     \
            --enable-shared         \
            --mandir=/usr/share/man \
            --with-tclinclude=/usr/include &&

make         &&
make test    &&
make install &&

ln -svf expect5.45.4/libexpect5.45.4.so /usr/lib &&

cd .. &&
rm -rf expect5.45.4
```

### DejaGNU

If tests fail due to "your system has no more ptys," check that:

1. The `/dev/pts` virtual filesystem has been mounted
2. Your kernel is compiled with the option `CONFIG_DEVPTS_FS=y`
3. If running Arch Linux, you may need to restart since kernel modules get pulled out from underneath you during a system upgrade

If this happens, it is critical to fix as it will prevent any tests run by `expect` from working and these are needed to validate many of the critical system utilities.

```sh
tar xzvf $LFS/sources/dejagnu-1.6.2.tar.gz &&
cd        dejagnu-1.6.2                    &&

./configure --prefix=/usr &&

makeinfo --html --no-split -o doc/dejagnu.html doc/dejagnu.texi         &&
makeinfo --plaintext       -o doc/dejagnu.txt  doc/dejagnu.texi         &&
make     install                                                        &&

install  -v -dm755  /usr/share/doc/dejagnu-1.6.2                        &&
install  -v -m644   doc/dejagnu.{html,txt} /usr/share/doc/dejagnu-1.6.2 &&

make     check &&

cd .. &&
rm -rf dejagnu-1.6.2
```

### binutils

`--enable-targets=x86_64-pep` enables support for EFI executables and is a dependency of `gnu-efi`.

Beware that four tests are known to fail, in the suite for `ld`, all saying some variety of "Run property."

```sh
tar xvf $LFS/sources/binutils-2.36.1.tar.xz &&
cd       binutils-2.36.1                    &&

sed -i '/@\tincremental_copy/d' gold/testsuite/Makefile.in &&

mkdir -v build &&
cd       build &&

../configure --prefix=/usr       \
             --enable-gold       \
             --enable-ld=default \
             --enable-plugins    \
             --enable-shared     \
             --disable-werror    \
             --enable-64-bit-bfd \
             --with-system-zlib  \
             --enable-targets=x86_64-pep &&

make tooldir=/usr         &&
make -k check             &&
make tooldir=/usr install &&

rm -fv /usr/lib/lib{bfd,ctf,ctf-nobfd,opcodes}.a &&

cd ../.. &&
rm -rf binutils-2.36.1
```

### gmp

```sh
tar xvf $LFS/sources/gmp-6.2.1.tar.xz &&
cd       gmp-6.2.1                    &&

./configure --prefix=/usr    \
            --enable-cxx     \
            --disable-static \
            --docdir=/usr/share/doc/gmp-6.2.1 &&

make                                &&
make html                           &&
make check 2>&1 | tee gmp-check-log &&

awk '/# PASS:/{total+=$3} ; END{print total}' gmp-check-log &&

make install      &&
make install-html &&

cd .. &&
rm -rf gmp-6.2.1
```

### mpfr

```sh
tar xvf $LFS/sources/mpfr-4.1.0.tar.xz &&
cd       mpfr-4.1.0                    &&

./configure --prefix=/usr        \
            --disable-static     \
            --enable-thread-safe \
            --docdir=/usr/share/doc/mpfr-4.1.0 &&

make              &&
make html         &&
make check        &&
make install      &&
make install-html &&

cd .. &&
rm -rf mpfr-4.1.0
```

### mpc

```sh
tar xzvf $LFS/sources/mpc-1.2.1.tar.gz &&
cd        mpc-1.2.1                    &&

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/mpc-1.2.1 &&

make              &&
make html         &&
make check        &&
make install      &&
make install-html &&

cd .. &&
rm -rf mpc-1.2.1
```

### isl

```sh
tar xvf $LFS/sources/isl-0.24.tar.xz &&
cd       isl-0.24                    &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
make check        &&
make install      &&

cd .. &&
rm -rf isl-0.24
```

### attr

```sh
tar xzvf $LFS/sources/attr-2.5.1.tar.gz &&
cd        attr-2.5.1                    &&

./configure --prefix=/usr     \
            --disable-static  \
            --sysconfdir=/etc \
            --docdir=/usr/share/doc/attr-2.5.1 &&

make         &&
make check   &&
make install &&

cd .. &&
rm -rf attr-2.5.1
```

### acl

```sh
tar xvf $LFS/sources/acl-2.3.1.tar.xz &&
cd        acl-2.3.1                   &&

./configure --prefix=/usr         \
            --disable-static      \
            --libexecdir=/usr/lib \
            --docdir=/usr/share/doc/acl-2.2.1 &&

make         &&
make install &&

cd .. &&
rm -rf acl-2.3.1
```

### libcap

```sh
tar xvf $LFS/sources/libcap-2.49.tar.xz &&
cd       libcap-2.49                    &&

sed -i '/install -m.*STA/d' libcap/Makefile &&
make prefix=/usr lib=lib                    &&
make test                                   &&
make prefix=/usr lib=lib install            &&

cd .. &&
rm -rf libcap-2.49
```

### shadow

#### Installation

```sh
tar xvf $LFS/sources/shadow-4.8.1.tar.xz &&
cd       shadow-4.8.1                    &&

sed -i 's/groups$(EXEEXT) //' src/Makefile.in                     &&
find man -name Makefile.in -exec sed -i 's/groups\.1 / /'   {} \; &&
find man -name Makefile.in -exec sed -i 's/getspnam\.3 / /' {} \; &&
find man -name Makefile.in -exec sed -i 's/passwd\.5 / /'   {} \; &&
sed -e 's:#ENCRYPT_METHOD DES:ENCRYPT_METHOD SHA512:' \
    -e 's:/var/spool/mail:/var/mail:'                 \
    -i etc/login.defs                                   &&
sed -i 's/1000/999/' etc/useradd                        &&
touch /usr/bin/passwd                                   &&

./configure --sysconfdir=/etc \
            --with-group-name-max-length=32 &&

make         &&
make install &&

cd .. &&
rm -rf shadow-4.8.1
```

#### Configuration

This unsets default mailbox creation and sets a root password.

```sh
pwconv
grpconv
sed -i 's/yes/no/' /etc/default/useradd
passwd root
```

### gcc

```sh
tar xvf $LFS/sources/gcc-11.1.0.tar.xz &&
cd       gcc-11.1.0                    &&

sed -e '/m64=/s/lib64/lib/' \
    -i.orig gcc/config/i386/t-linux64 &&

mkdir -v build &&
cd       build &&

../configure --prefix=/usr            \
             LD=ld                    \
             --enable-languages=c,c++ \
             --disable-multilib       \
             --disable-bootstrap      \
             --with-system-zlib &&

make -j 4 &&

ulimit -s 32768    &&
chown -Rv tester . &&
su tester -c "PATH=$PATH make -k check"
```

To check the results, run:

```sh
../contrib/test_summary | grep -A7 Summ
```

This will produce output that looks like this:

```
        === g++ Summary ===

# of expected passes        208074
# of unexpected failures    3
# of unexpected successes   3
# of expected failures      1054
# of unsupported tests      8962
/sources/build_dir/gcc-11.1.0/build/gcc/xg++  version 11.1.0 (GCC)
--
        === gcc Summary ===

# of expected passes        160277
# of expected failures      860
# of unsupported tests      2389
/sources/build_dir/gcc-11.1.0/build/gcc/xgcc  version 11.1.0 (GCC)

        === libatomic tests ===
--
        === libatomic Summary ===

# of expected passes        54
        === libgomp tests ===


Running target unix

        === libgomp Summary ===

# of expected passes        2894
# of expected failures      6
# of unsupported tests      308
        === libitm tests ===


--
        === libitm Summary ===

# of expected passes        42
# of expected failures      3
# of unsupported tests      1
        === libstdc++ tests ===


--
        === libstdc++ Summary ===

# of expected passes        14769
# of unexpected failures    7
# of expected failures      104
# of unsupported tests      352

Compiler version: 11.1.0 (GCC)
```

You can compare to results found at [gcc test results archive](https://gcc.gnu.org/pipermail/gcc-testresults/). Many tests will fail, but just be sure you're not seeing disproportionately more failures than the developers.

To install:

```sh
make install &&

rm    -rf   /usr/lib/gcc/$(gcc -dumpmachine)/11.1.0/include-fixed/bits/ &&
chown -v -R root:root /usr/lib/gcc/*linux-gnu/11.1.0/include{,-fixed}   &&
ln    -sv   ../usr/bin/cpp /lib/cpp                                     &&
ln    -sfv  ../../libexec/gcc/$(gcc -dumpmachine)/11.1.0/liblto_plugin.so \
            /usr/lib/bfd-plugins/                                       &&
mkdir -pv   /usr/share/gdb/auto-load/usr/lib                            &&
mv    -v    /usr/lib/*gdb.py /usr/share/gdb/auto-load/usr/lib           &&

cd ../.. &&
rm -rf gcc-11.1.0
```

#### Sanity check gcc

Linux From Scratch recommends making a simple test program. It is an empty program because we are only interested in the build log.

```sh
echo 'int main(){}' > dummy.c
cc dummy.c -v -Wl,--verbose &> dummy.log
readelf -l a.out | grep ': /lib'
```

This should print out:

```
[Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
```

Then ensure we're setup to use the correct start files:

```sh
grep -o '/usr/lib.*/crt[1in].*succeeded' dummy.log
```

Should print:

```
/usr/lib/gcc/x86_64-pc-linux-gnu/11.1.0/../../../../lib/crt1.o succeeded
/usr/lib/gcc/x86_64-pc-linux-gnu/11.1.0/../../../../lib/crti.o succeeded
/usr/lib/gcc/x86_64-pc-linux-gnu/11.1.0/../../../../lib/crtn.o succeeded
```

Note that it should say differently if you're not running on `x86_64` architecture.

### pkgconfig

```sh
tar xzvf $LFS/sources/pkg-config-0.29.2.tar.gz &&
cd        pkg-config-0.29.2                    &&

./configure --prefix=/usr              \
            --with-internal-glib       \
            --disable-host-tool        \
            --docdir=/usr/share/doc/pkg-config-0.29.2 &&

make         &&
make check   &&
make install &&

cd .. &&
rm -rf pkg-config-0.29.2
```

### ncurses

```sh
tar xzvf $LFS/sources/ncurses-6.2.tar.gz &&
cd        ncurses-6.2                    &&

./configure --prefix=/usr           \
            --mandir=/usr/share/man \
            --with-shared           \
            --without-debug         \
            --without-normal        \
            --enable-pc-files       \
            --enable-widec &&

make         &&
make install &&

for lib in ncurses form panel menu tinfo ; do
    rm -vf                    /usr/lib/lib${lib}.so        &&
    echo "INPUT(-l${lib}w)" > /usr/lib/lib${lib}.so        &&
    ln -sfv ${lib}w.pc        /usr/lib/pkgconfig/${lib}.pc
done

rm -vf                     /usr/lib/libcursesw.so &&
echo "INPUT(-lncursesw)" > /usr/lib/libcursesw.so &&
ln -sfv libncurses.so      /usr/lib/libcurses.so  &&
rm -fv /usr/lib/libncurses++w.a                   &&
mkdir -v       /usr/share/doc/ncurses-6.2         &&
cp -v -R doc/* /usr/share/doc/ncurses-6.2         &&

cd .. &&
rm -rf ncurses-6.2
```

### sed

```sh
tar xvf $LFS/sources/sed-4.8.tar.xz &&
cd       sed-4.8                    &&

./configure --prefix=/usr \
            --bindir=/bin &&

make      &&
make html &&

chown -Rv tester .                      &&
su tester -c "PATH=$PATH make -k check" &&

make install                                      &&
install -d -m755           /usr/share/doc/sed-4.8 &&
install -m644 doc/sed.html /usr/share/doc/sed-4.8 &&

cd .. &&
rm -rf sed-4.8
```

### psmisc

```sh
tar xvf $LFS/sources/psmisc-23.4.tar.xz &&
cd       psmisc-23.4                    &&

./configure --prefix=/usr &&

make         &&
make install &&

cd .. &&
rm -rf psmisc-23.4
```

### gettext

```sh
tar xvf $LFS/sources/gettext-0.21.tar.xz &&
cd       gettext-0.21                    &&

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/gettext-0.21 &&

make         &&
make check   &&
make install &&

chmod -v 0755 /usr/lib/preloadable_libintl.so &&

cd .. &&
rm -rf gettext-0.21
```

### bison

```sh
tar xvf $LFS/sources/bison-3.7.6.tar.xz &&
cd       bison-3.7.6                    &&

./configure --prefix=/usr \
            --docdir=/usr/share/doc/bison-3.7.6 &&

make         &&
make check   &&
make install &&

cd .. &&
rm -rf bison-3.7.6
```

### grep

```sh
tar xvf $LFS/sources/grep-3.6.tar.xz &&
cd       grep-3.6                    &&

./configure --prefix=/usr \
            --bindir=/bin &&

make         &&
make check   &&
make install &&

cd .. &&
rm -rf grep-3.6
```

### bash

```sh
tar xzvf $LFS/sources/bash-5.1.tar.gz &&
cd        bash-5.1                    &&

sed -i  '/^bashline.o:.*shmbchar.h/a bashline.o: ${DEFDIR}/builtext.h' Makefile.in &&

./configure --prefix=/usr                    \
            --docdir=/usr/share/doc/bash-5.1 \
            --without-bash-malloc            \
            --with-installed-readline &&

make               &&
chown -Rv tester . &&
su tester << EOF   &&
PATH=$PATH make tests < $(tty)
EOF

make install
```

Be sure to log back in now that you have a new installation.

```sh
exec /bin/bash --login +h
```

Then delete the build tree:

```sh
cd .. &&
rm -rf bash-5.1
```

### libtool

As noted in Linux From Scratch, five tests are expected to fail when run before `automake` is built due to circular dependencies.

```sh
tar xvf $LFS/sources/libtool-2.4.6.tar.xz &&
cd       libtool-2.4.6                    &&

./configure --prefix=/usr &&

make         &&
make check   &&
make install &&

rm -fv /usr/lib/libltdl.a &&

cd .. &&
rm -rf libtool-2.4.6
```

### gdbm

The "version" test is known to fail.

```sh
tar xzvf $LFS/sources/gdbm-1.19.tar.gz &&
cd        gdbm-1.19                    &&

./configure --prefix=/usr    \
            --disable-static \
            --enable-libgdbm-compat &&

make         &&
make check   &&
make install &&

cd .. &&
rm -rf gdbm-1.19
```

### gperf

```sh
tar xzvf $LFS/sources/gperf-3.1.tar.gz &&
cd        gperf-3.1                    &&

./configure --prefix=/usr \
            --docdir=/usr/share/doc/gperf-3.1 &&

make             &&
make -j1 check   &&
make     install &&

cd .. &&
rm -rf gperf-3.1
```

### expat

```sh
tar xvf $LFS/sources/expat-2.4.1.tar.xz &&
cd       expat-2.4.1                    &&

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/expat-2.4.1 &&

make         &&
make check   &&
make install &&

install -v -m644 doc/*.{html,png,css} /usr/share/doc/expat-2.4.1 &&

cd .. &&
rm -rf expat-2.4.1
```

### inetutils

```sh
tar xvf $LFS/sources/inetutils-2.0.tar.xz &&
cd       inetutils-2.0                    &&

./configure --prefix=/usr        \
            --localstatedir=/var \
            --disable-logger     \
            --disable-whois      \
            --disable-rcp        \
            --disable-rexec      \
            --disable-rlogin     \
            --disable-rsh        \
            --disable-servers    \
            --disable-talk &&

make         &&
make check   &&
make install &&

cd .. &&
rm -rf inetutils-2.0
```

### perl

```sh
tar xvf $LFS/sources/perl-5.32.1.tar.xz &&
cd       perl-5.32.1                    &&

export BUILD_ZLIB=False &&
export BUILD_BZIP2=0    &&

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
             -Dusethreads &&

make         &&
make test    &&
make install &&
unset BUILD_ZLIB BUILD_BZIP2 &&

cd .. &&
rm -rf perl-5.32.1
```

### XML::Parser

This is an additional Perl module used by other tools.

```sh
tar xzvf $LFS/sources/XML-Parser-2.46.tar.gz &&
cd        XML-Parser-2.46                    &&

perl Makefile.PL &&
make             &&
make test        &&
make install     &&

cd .. &&
rm -rf XML-Parser-2.46
```

### intltool

```sh
tar xzvf $LFS/sources/intltool-0.51.0.tar.gz &&
cd        intltool-0.51.0                    &&

sed -i 's:\\\${:\\\$\\{:' intltool-update.in &&
./configure --prefix=/usr

make         &&
make check   &&
make install &&

install -v -Dm644 doc/I18N-HOWTO /usr/share/doc/intltool-0.51.0/I18N-HOWTO &&

cd .. &&
rm -rf intltool-0.51.0
```

### autoconf

```sh
tar xvf $LFS/sources/autoconf-2.71.tar.xz &&
cd       autoconf-2.71                    &&

./configure --prefix=/usr &&

make         &&
make check   &&
make install &&

cd .. &&
rm -rf autoconf-2.71
```

### automake

Several `automake` tests are expected to fail.

```sh
tar xvf /sources/automake-1.16.3.tar.xz &&
cd       automake-1.16.3                &&

sed -i "s/''/etags/" t/tags-lisp-space.sh &&
./configure --prefix=/usr \
            --docdir=/usr/share/doc/automake-1.16.3 &&

make         &&
make check   &&
make install &&

cd .. &&
rm -rf automake-1.16.3
```

At this point, you should be able to rebuild `libtool` if you wish to make all of the tests pass.

### kmod

```sh
tar xvf /sources/kmod-28.tar.xz &&
cd       kmod-28                &&

./configure --prefix=/usr          \
            --bindir=/bin          \
            --sysconfdir=/etc      \
            --with-rootlibdir=/lib \
            --with-xz              \
            --with-zstd            \
            --with-zlib &&

make         &&
make install &&

for target in depmod insmod lsmod modinfo modprobe rmmod; do
  sudo ln -sfv kmod /usr/bin/$target
done

cd .. &&
rm -rf kmod-28
```

### libelf

```sh
tar xvf /sources/elfutils-0.183.tar.bz2 &&
cd       elfutils-0.183                 &&

./configure --prefix=/usr                \
            --disable-debuginfod         \
            --enable-libdebuginfod=dummy \
            --libdir=/lib &&

make                                               &&
make check                                         &&
make -C libelf install                             &&
install -vm644 config/libelf.pc /usr/lib/pkgconfig &&
rm /lib/libelf.a                                   &&

cd .. &&
rm -rf elfutils-0.183
```

### libffi

```sh
tar xzvf /sources/libffi-3.3.tar.gz &&
cd        libffi-3.3                &&

./configure --prefix=/usr    \
            --disable-static \
            --with-gcc-arch=native &&

make         &&
make check   &&
make install &&

cd .. &&
rm -rf libffi-3.3
```

### openssl

```sh
tar xzvf $LFS/sources/openssl-1.1.1k.tar.gz &&
cd        openssl-1.1.1k                    &&

./config --prefix=/usr         \
         --openssldir=/etc/ssl \
         --libdir=lib          \
         shared                \
         zlib-dynamic &&

make                                                       &&
make test                                                  &&
sed -i '/INSTALL_LIBS/s/libcrypto.a libssl.a//' Makefile   &&
make MANSUFFIX=ssl install                                 &&
mv -v /usr/share/doc/openssl /usr/share/doc/openssl-1.1.1k &&
cp -vfr doc/* /usr/share/doc/openssl-1.1.1k                &&

cd .. &&
rm -rf openssl-1.1.1k
```

### Python

We will provide links from `/usr/bin/python3` to `/usr/bin/python` and `/usr/bin/pip3` to `/usr/bin/pip` as has been standard on many distros since Python 2 went past its EOL.

Python can optionally include C modules for `uuid` and `sqlite` if these are available on the system. Though presented as optional extras, you may wish to install them now. If you add `sqlite`, also add the `--enable-loadable-sqlite-extensions` flag to the `configure` script.

#### libuuid

If you wish to install this:

```sh
tar xvf /sources/libuuid-1.0.3.tar.gz &&
cd       libuuid-1.0.3                &&

./configure --prefix=/usr \
            --disable-static &&

make         &&
make check   &&
make install &&

cd ..
rm -rf libuuid-1.0.3
```

It is not recommended to run tests in the chroot environment as the socket tests will hang indefinitely. When the new system is booted into and a network is available, `Python` can be rebuilt and tests run. This is a good chance to add `sqlite` support.

```sh
tar xvf /sources/Python-3.9.5.tar.xz &&
cd       Python-3.9.5                &&

./configure --prefix=/usr          \
            --enable-shared        \
            --enable-optimizations \
            --with-system-expat    \
            --with-system-ffi      \
            --with-ensurepip=yes &&

make         &&
make install &&

install -v -dm755 /usr/share/doc/python-3.9.5/html &&

tar --strip-components=1  \
    --no-same-owner       \
    --no-same-permissions \
    -C /usr/share/doc/python-3.9.5/html \
    -xvf ../../python-3.9.5-docs-html.tar.bz2 &&
ln -sfv /usr/bin/python3 /usr/bin/python      &&
ln -sfv /usr/bin/pip3 /usr/bin/pip            &&

cd .. &&
rm -rf Python-3.9.5
```

### ninja

```sh
tar xzvf $LFS/sources/ninja-1.10.2.tar.gz &&
cd        ninja-1.10.2                    &&

sed -i '/int Guess/a \
  int   j = 0;\
  char* jobs = getenv( "NINJAJOBS" );\
  if ( jobs != NULL ) j = atoi( jobs );\
  if ( j > 0 ) return j;\
' src/ninja.cc                   &&
python3 configure.py --bootstrap &&

./ninja ninja_test                                      &&
./ninja_test --gtest_filter=-SubprocessTest.SetWithLots &&

install -vm755 ninja /usr/bin/                                                    &&
install -vDm644 misc/bash-completion /usr/share/bash-completion/completions/ninja &&
install -vDm644 misc/zsh-completion  /usr/share/zsh/site-functions/_ninja         &&

cd .. &&
rm -rf ninja-1.10.2
```

### meson

```sh
tar xzvf $LFS/sources/meson-0.58.0.tar.gz &&
cd        meson-0.58.0                    &&

python3 setup.py build               &&
python3 setup.py install --root=dest &&
cp -rv dest/* /                      &&

cd .. &&
rm -rf meson-0.58.0
```

### coreutils

```sh
tar xvf $LFS/sources/coreutils-8.32.tar.xz &&
cd       coreutils-8.32                    &&

patch -Np1 -i ../../coreutils-8.32-i18n-1.patch   &&
sed -i '/test.lock/s/^/#/' gnulib-tests/gnulib.mk &&
autoreconf -fiv                                   &&
FORCE_UNSAFE_CONFIGURE=1 ./configure \
            --prefix=/usr            \
            --enable-no-install-program=kill,uptime &&

make                                                         &&
make NON_ROOT_USERNAME=tester check-root                     &&
echo "dummy:x:102:tester" >> /etc/group                      &&
chown -Rv tester .                                           &&
su tester -c "PATH=$PATH make RUN_EXPENSIVE_TESTS=yes check" &&
sed -i '/dummy/d' /etc/group                                 &&
make install                                                 &&

mv -v /usr/share/man/man1/chroot.1 /usr/share/man/man8/chroot.8 &&
sed -i 's/"1"/"8"/' /usr/share/man/man8/chroot.8                &&

cd .. &&
rm -rf coreutils-8.32
```

### check

```sh
tar xzvf $LFS/sources/check-0.15.2.tar.gz &&
cd        check-0.15.2                    &&

./configure --prefix=/usr \
            --disable-static &&

make                                            &&
make check                                      &&
make docdir=/usr/share/doc/check-0.15.2 install &&

cd .. &&
rm -rf check-0.15.2
```

### diffutils

```sh
tar xvf $LFS/sources/diffutils-3.7.tar.xz &&
cd       diffutils-3.7                    &&

./configure --prefix=/usr &&

make         &&
make check   &&
make install &&

cd .. &&
rm -rf diffutils-3.7
```

### gawk

```sh
tar xvf $LFS/sources/gawk-5.1.0.tar.xz &&
cd       gawk-5.1.0                    &&

sed -i 's/extras//' Makefile.in &&
./configure --prefix=/usr       &&

make                               &&
make check                         &&
make install                       &&
mkdir -v /usr/share/doc/gawk-5.1.0 &&
cp    -v doc/{awkforai.txt,*.{eps,pdf,jpg}} /usr/share/doc/gawk-5.1.0 &&

cd .. &&
rm -rf gawk-5.1.0
```

### findutils

```sh
tar xvf $LFS/sources/findutils-4.8.0.tar.xz &&
cd       findutils-4.8.0                    &&

./configure --prefix=/usr \
            --localstatedir=/var/lib/locate &&

make                                    &&
chown -Rv tester .                      &&
su tester -c "PATH=$PATH make -j check" &&
make install                            &&

sed -i 's|find:=${BINDIR}|find:=/bin|' /usr/bin/updatedb &&

cd .. &&
rm -rf findutils-4.8.0
```

### groff

```sh
tar xzvf /sources/groff-1.22.4.tar.gz &&
cd        groff-1.22.4                &&

PAGE=letter ./configure --prefix=/usr &&

make -j1         &&
make -j1 install &&

cd .. &&
rm -rf groff-1.22.4
```

### less

```sh
tar xzvf /sources/less-581.tar.gz &&
cd        less-581                &&

./configure --prefix=/usr \
            --sysconfdir=/etc &&

make         &&
make install &&

cd .. &&
rm -rf less-581
```

### gzip

```sh
tar xvf /sources/gzip-1.10.tar.xz &&
cd       gzip-1.10                &&

./configure --prefix=/usr &&

make         &&
make check   &&
make install &&

cd .. &&
rm -rf gzip-1.10
```

### iproute2

```sh
tar xvf /sources/iproute2-5.12.0.tar.xz &&
cd       iproute2-5.12.0                &&

sed -i  /ARPD/d Makefile          &&
rm  -fv man/man8/arpd.8           &&
sed -i 's/.m_ipt.0//' tc/Makefile &&

make                                               &&
make DOCDIR=/usr/share/doc/iproute2-5.12.0 install &&

cd .. &&
rm -rf iproute2-5.12.0
```

### kbd

```sh
tar xvf /sources/kbd-2.4.0.tar.xz &&
cd       kbd-2.4.0                &&

patch -Np1 -i ../../kbd-2.4.0-backspace-1.patch      &&
sed -i '/RESIZECONS_PROGS=/s/yes/no/' configure      &&
sed -i 's/resizecons.8 //' docs/man/man8/Makefile.in &&
./configure --prefix=/usr \
            --disable-vlock &&

make                                         &&
make check                                   &&
make install                                 &&
mkdir -v            /usr/share/doc/kbd-2.4.0 &&
cp -R -v docs/doc/* /usr/share/doc/kbd-2.4.0 &&

cd .. &&
rm -rf kbd-2.4.0
```

### libpipeline

```sh
tar xzvf /sources/libpipeline-1.5.3.tar.gz &&
cd        libpipeline-1.5.3                &&

./configure --prefix=/usr &&

make         &&
make check   &&
make install &&

cd .. &&
rm -rf libpipeline-1.5.3
```

### make

```sh
tar xzvf /sources/make-4.3.tar.gz &&
cd        make-4.3                &&

./configure --prefix=/usr &&

make         &&
make check   &&
make install &&

cd .. &&
rm -rf make-4.3
```

### patch

```sh
tar xvf /sources/patch-2.7.6.tar.xz &&
cd       patch-2.7.6                &&

./configure --prefix=/usr &&

make         &&
make check   &&
make install &&

cd .. &&
rm -rf patch-2.7.6
```

### man-db

```sh
tar xvf /sources/man-db-2.9.4.tar.xz &&
cd       man-db-2.9.4                &&

sed -i '/find/s@/usr@@' init/systemd/man-db.service.in &&
./configure --prefix=/usr                        \
            --docdir=/usr/share/doc/man-db-2.9.4 \
            --sysconfdir=/etc                    \
            --disable-setuid                     \
            --enable-cache-owner=bin             \
            --with-browser=/usr/bin/lynx         \
            --with-vgrind=/usr/bin/vgrind        \
            --with-grap=/usr/bin/grap &&

make         &&
make check   &&
make install &&

cd .. &&
rm -rf man-db-2.9.4
```

### tar

Test 227 'binary store/restore' is known to fail.

```sh
tar xvf /sources/tar-1.34.tar.xz &&
cd       tar-1.34                &&

FORCE_UNSAFE_CONFIGURE=1  \
./configure --prefix=/usr \
            --bindir=/bin &&

make         &&
make check   &&
make install &&
make -C doc install-html docdir=/usr/share/doc/tar-1.34 &&

cd .. &&
rm -rf tar-1.34
```

### texinfo

```sh
tar xvf /sources/texinfo-6.7.tar.xz &&
cd       texinfo-6.7                &&

./configure --prefix=/usr &&

make         &&
make check   &&
make install &&
make TEXMF=/usr/share/texmf install-tex &&

pushd /usr/share/info
  rm -v dir
  for f in *
    do install-info $f dir 2>/dev/null
  done
popd

cd .. &&
rm -rf texinfo-6.7
```

### vim

```sh
tar xzvf /sources/vim-8.2.2813.tar.gz &&
cd        vim-8.2.2813                &&

echo '#define SYS_VIMRC_FILE "/etc/vimrc"' >> src/feature.h &&
./configure --prefix=/usr                                   &&

make                                                          &&
chown -Rv tester .                                            &&
su tester -c "LANG=en_US.UTF-8 make -j1 test" &> vim-test.log &&
make install                                                  &&

ln -sv vim /usr/bin/vi &&

for L in  /usr/share/man/{,*/}man1/vim.1; do
    ln -sv vim.1 $(dirname $L)/vi.1
done
ln -sv ../vim/vim82/doc /usr/share/doc/vim-8.2.2813 &&

cat > /etc/vimrc << EOF &&
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

cd .. &&
rm -rf vim-8.2.2813
```

### gnu-efi

This is not part of base Linux From Scratch, but required for using `systemd-boot`.

```sh
tar xzvf /sources/gnu-efi-3.0.12.tar.gz &&
cd        gnu-efi-3.0.12                &&

make                     &&
make PREFIX=/usr install &&

cd .. &&
rm -rf gnu-efi-3.0.12
```

### PAM

This is not part of base Linux From Scratch, but enables better and more secure authentication rules.

```sh
tar xvf /sources/Linux-PAM-1.5.1.tar.xz &&
cd       Linux-PAM-1.5.1                &&

./configure --prefix=/usr                    \
            --sysconfdir=/etc                \
            --libdir=/usr/lib                \
            --enable-securedir=/lib/security \
            --docdir=/usr/share/doc/Linux-PAM-1.5.1 &&

make
```

If this is the first installation, tests require the presence of an `/etc/pam.d/other` file, so create it:

```sh
install -v -m755 -d /etc/pam.d &&
cat > /etc/pam.d/other << EOF
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
make install                    &&
chmod -v 4755 /sbin/unix_chkpwd &&

cd .. &&
rm -rf Linux-PAM-1.5.1
```

Next, create some configuration files:

```sh
install -vdm755 /etc/pam.d             &&
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
tar xvf /sources/shadow-4.8.1.tar.xz &&
cd       shadow-4.8.1                &&

sed -i 's/groups$(EXEEXT) //' src/Makefile.in &&

find man -name Makefile.in -exec sed -i 's/groups\.1 / /'   {} \; &&
find man -name Makefile.in -exec sed -i 's/getspnam\.3 / /' {} \; &&
find man -name Makefile.in -exec sed -i 's/passwd\.5 / /'   {} \; &&

sed -e 's@#ENCRYPT_METHOD DES@ENCRYPT_METHOD SHA512@' \
    -e 's@/var/spool/mail@/var/mail@'                 \
    -i etc/login.defs                                 &&

sed -i 's/1000/999/' etc/useradd                      &&

./configure --sysconfdir=/etc \
            --with-group-name-max-length=32 &&

make         &&
make install &&

cd .. &&
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
    install -v -m644 /etc/pam.d/chage /etc/pam.d/${PROGRAM} &&
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
tar xzvf /sources/sudo-1.9.7.tar.gz &&
cd        sudo-1.9.7                &&

./configure --prefix=/usr              \
            --libexecdir=/usr/lib      \
            --with-secure-path         \
            --with-all-insults         \
            --with-env-editor          \
            --docdir=/usr/share/doc/sudo-1.9.5p2 \
            --with-passprompt="[sudo] password for %p: " &&

make
```

Run and check test results:

```sh
env LC_ALL=C make check 2>&1 | tee make-check.log &&
grep failed make-check.log
```

If pass, install:

```sh
make install                                                  &&
ln -sfv libsudo_util.so.0.0.0 /usr/lib/sudo/libsudo_util.so.0 &&

cd .. &&
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
tar xvf /sources/libcap-2.49.tar.xz &&
cd       libcap-2.49                &&

make -C pam_cap &&

install -v -m755 pam_cap/pam_cap.so /lib/security      &&
install -v -m644 pam_cap/capability.conf /etc/security &&

mv -v /etc/pam.d/system-auth{,.bak} &&
cat > /etc/pam.d/system-auth << EOF &&
# Begin /etc/pam.d/system-auth

auth      optional    pam_cap.so
EOF
tail -n +3 /etc/pam.d/system-auth.bak >> /etc/pam.d/system-auth &&

cd .. &&
rm -rf libcap-2.49
```

### libtasn

```sh
tar xzvf /sources/libtasn1-4.16.0.tar.gz &&
cd        libtasn1-4.16.0                &&

./configure --prefix=/usr \
            --disable-static &&

make         &&
make install &&
make -C doc/reference install-data-local &&

cd .. &&
rm -rf libtasn1-4.16.0
```

### p11-kit

```sh
tar xvf /sources/p11-kit-0.23.22.tar.xz &&
cd       p11-kit-0.23.22                &&

sed '20,$ d' -i trust/trust-extract-compat &&
cat >> trust/trust-extract-compat << "EOF" &&
# Copy existing anchor modifications to /etc/ssl/local
/usr/libexec/make-ca/copy-trust-modifications

# Generate a new trust store
/usr/sbin/make-ca -f -g
EOF

./configure --prefix=/usr     \
            --sysconfdir=/etc \
            --with-trust-paths=/etc/pki/anchors &&

make         &&
make install &&
ln -sfv /usr/libexec/p11-kit/trust-extract-compat \
        /usr/bin/update-ca-certificates                  &&
ln -sfv ./pkcs11/p11-kit-trust.so /usr/lib/libnssckbi.so &&

cd .. &&
rm -rf p11-kit-0.23.22
```

### make-ca

```sh
tar xvf /sources/make-ca-1.7.tar.xz &&
cd       make-ca-1.7                &&

make install                   &&
install -vdm755 /etc/ssl/local &&

cd .. &&
rm -rf make-ca-1.7
```

### nettle

```sh
tar xzvf /sources/nettle-3.7.1.tar.gz &&
cd        nettle-3.7.1                &&

./configure --prefix=/usr \
            --disable-static &&

make         &&
make install &&

chmod   -v   755 /usr/lib/lib{hogweed,nettle}.so         &&
install -v -m755 -d /usr/share/doc/nettle-3.7.1          &&
install -v -m644 nettle.html /usr/share/doc/nettle-3.7.1 &&

cd .. &&
rm -rf nettle-3.7.1
```

### libunistring

```sh
tar xvf /sources/libunistring-0.9.10.tar.gz &&
cd       libunistring-0.9.10                &&

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/libunistring-0.9.10 &&

make         &&
make install &&

cd .. &&
rm -rf libunistring-0.9.10
```

### GnuTLS

```sh
tar xvf /sources/gnutls-3.7.0.tar.xz &&
cd       gnutls-3.7.0                &&

./configure --prefix=/usr                        \
            --docdir=/usr/share/doc/gnutls-3.7.0 \
            --disable-guile                      \
            --with-default-trust-store-pkcs11="pkcs11:" &&

make                                     &&
make install                             &&
make -C doc/reference install-data-local &&

cd .. &&
rm -rf gnutls-3.7.0
```

### cURL

```sh
tar xzvf /sources/curl-7.76.1.tar.gz &&
cd        curl-7.76.1                &&

./configure --prefix=/usr    \
            --with-openssl   \
            --with-gnutls    \
            --disable-static

make                      &&
make test                 &&
make install              &&
rm -v /usr/lib/libcurl.la &&

cd .. &&
rm -rf curl-7.76.1
```

### pcre

```sh
tar xvf /sources/pcre-8.44.tar.bz2 &&
cd       pcre-8.44                 &&

./configure --prefix=/usr                     \
            --docdir=/usr/share/doc/pcre-8.44 \
            --enable-unicode-properties       \
            --enable-pcre16                   \
            --enable-pcre32                   \
            --enable-pcregrep-libz            \
            --enable-pcregrep-libbz2          \
            --enable-pcretest-libreadline     \
            --disable-static &&

make         &&
make install &&

cd .. &&
rm -rf pcre-8.44
```

### zsh

```sh
tar xvf /sources/zsh-5.8.tar.xz &&
cd       zsh-5.8                &&

./configure --prefix=/usr                             \
            --docdir=/usr/share/doc/zsh               \
            --htmldir=/usr/share/doc/zsh/html         \
            --enable-etcdir=/etc/zsh                  \
            --enable-zshenv=/etc/zsh/zshenv           \
            --enable-zlogin=/etc/zsh/zlogin           \
            --enable-zlogout=/etc/zsh/zlogout         \
            --enable-zprofile=/etc/zsh/zprofile       \
            --enable-zshrc=/etc/zsh/zshrc             \
            --enable-multibyte                        \
            --enable-function-subdirs                 \
            --enable-fndir=/usr/share/zsh/functions   \
            --enable-scriptdir=/usr/share/zsh/scripts \
            --with-tcsetpgrp                          \
            --enable-pcre                             \
            --enable-cap                              \
            --enable-zsh-secure-free &&

make                                                                  &&
makeinfo  Doc/zsh.texi --plaintext -o Doc/zsh.txt                     &&
makeinfo  Doc/zsh.texi --html      -o Doc/html                        &&
makeinfo  Doc/zsh.texi --html --no-split --no-headers -o Doc/zsh.html &&
make install                                                          &&
make infodir=/usr/share/info install.info                             &&

install -v -m755 -d                 /usr/share/doc/zsh-5.8/html &&
install -v -m644 Doc/html/*         /usr/share/doc/zsh-5.8/html &&
install -v -m644 Doc/zsh.{html,txt} /usr/share/doc/zsh-5.8      &&

cd .. &&
rm -rf zsh-5.8
```

### which

```sh
tar xzvf /sources/which-2.21.tar.gz &&
cd        which-2.21                &&

./configure --prefix=/usr     \
            --sysconfdir=/etc &&

make         &&
make install &&

cd .. &&
rm -rf which-2.21
```

### systemd

The `gnu-efi=true` option is added to build the efi executables installed by `bootctl install`.

```sh
tar xzvf /sources/systemd-248.tar.gz &&
cd        systemd-248                &&

patch -Np1 -i ../../systemd-248-upstream_fixes-1.patch                    &&
sed -i 's/GROUP="render"/GROUP="video"/' rules.d/50-udev-default.rules.in &&

mkdir -v build &&
cd       build &&
LANG=en_US.UTF-8                          \
meson --prefix=/usr                       \
      --sysconfdir=/etc                   \
      --localstatedir=/var                \
      -Dblkid=true                        \
      -Dbuildtype=release                 \
      -Ddefault-dnssec=no                 \
      -Dfirstboot=false                   \
      -Dinstall-tests=false               \
      -Dldconfig=false                    \
      -Dsysusers=false                    \
      -Db_lto=false                       \
      -Drpmmacrosdir=no                   \
      -Dhomed=false                       \
      -Duserdb=false                      \
      -Dman=false                         \
      -Dmode=release                      \
      -Ddocdir=/usr/share/doc/systemd-248 \
      -Dgnu-efi=true                      \
      .. &&

LANG=en_US.UTF-8 ninja         &&
LANG=en_US.UTF-8 ninja install &&

tar -xf ../../../systemd-man-pages-248.tar.xz --strip-components=1 -C /usr/share/man &&
rm  -rf /usr/lib/pam.d                                                               &&

systemd-machine-id-setup                         &&
systemctl preset-all                             &&
systemctl disable systemd-time-wait-sync.service &&

cd ../.. &&
rm -rf systemd-248
```

Also enable the automated update service for `make-ca`:

```sh
systemctl enable update-pki.timer
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

Then unpack and build:

```sh
tar xzvf /sources/openssh-8.6p1.tar.gz &&
cd        openssh-8.6p1                &&

sed -e '/INSTALLKEYS_SH/s/)//' -e '260a\  )' -i contrib/ssh-copy-id &&
./configure --prefix=/usr                     \
            --sysconfdir=/etc/ssh             \
            --with-privsep-path=/var/lib/sshd &&
make
```

Running the tests requires copying `scp` to `/usr/bin`:

```sh
cp scp /usr/bin &&
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
                    /usr/share/doc/openssh-8.6p1     &&

cd .. &&
rm -rf openssh-8.6p1
```

Disable root login and password logins:

```sh
echo "PermitRootLogin no" >> /etc/ssh/sshd_config        &&
echo "PasswordAuthentication no" >> /etc/ssh/sshd_config &&
echo "ChallengeResponseAuthentication no" >> /etc/ssh/sshd_config
```

Running `sshd` requires the `systemd` unit:

```sh
tar xvf blfs-systemd-units-20210122.tar.xz &&
cd      blfs-systemd-units-20210122        &&

make install-sshd                          &&
systemctl enable sshd.service
```

### D-Bus

```sh
tar xzvf /sources/dbus-1.12.20.tar.gz &&
cd        dbus-1.12.20                &&

./configure --prefix=/usr                        \
            --sysconfdir=/etc                    \
            --localstatedir=/var                 \
            --disable-static                     \
            --disable-doxygen-docs               \
            --disable-xml-docs                   \
            --docdir=/usr/share/doc/dbus-1.12.20 \
            --with-console-auth-dir=/run/console \
            --with-system-pid-file=/run/dbus/pid \
            --with-system-socket=/run/dbus/system_bus_socket &&

make         &&
make install &&

ln -sfv /etc/machine-id /var/lib/dbus &&

cd .. &&
rm -rf dbus-1.12.20
```

### procps-ng

```sh
tar xvf /sources/procps-ng-3.3.17.tar.xz &&
cd       procps-3.3.17                   &&

./configure --prefix=/usr                            \
            --exec-prefix=                           \
            --libdir=/usr/lib                        \
            --docdir=/usr/share/doc/procps-ng-3.3.17 \
            --disable-static                         \
            --disable-kill                           \
            --with-systemd &&

make         &&
make check   &&
make install &&

cd .. &&
rm -rf procps-ng-3.3.17
```

### util-linux

```sh
tar xvf /sources/util-linux-2.36.2.tar.xz &&
cd       util-linux-2.36.2                &&

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
            runstatedir=/run &&

make                         &&
chown -Rv tester .           &&
su tester -c "make -k check" &&
make install                 &&

cd .. &&
rm -rf util-linux-2.36.2
```

### Filesystem programs

Select whichever of these you need support for, depending upon the filesystem(s) you created on your LFS partition(s).

#### e2fsprogs

```sh
tar xzvf /sources/e2fsprogs-1.46.2.tar.gz &&
cd        e2fsprogs-1.46.2                &&

mkdir -v build &&
cd       build &&

../configure --prefix=/usr           \
             --bindir=/bin           \
             --with-root-prefix=""   \
             --enable-elf-shlibs     \
             --disable-libblkid      \
             --disable-libuuid       \
             --disable-uuidd         \
             --disable-fsck &&

make         &&
make check   &&
make install &&

rm -fv    /usr/lib/{libcom_err,libe2p,libext2fs,libss}.a                   &&
gunzip -v /usr/share/info/libext2fs.info.gz                                &&
install-info --dir-file=/usr/share/info/dir /usr/share/info/libext2fs.info &&
makeinfo -o      doc/com_err.info ../lib/et/com_err.texinfo                &&
install -v -m644 doc/com_err.info /usr/share/info                          &&
install-info --dir-file=/usr/share/info/dir /usr/share/info/com_err.info   &&

cd ../.. &&
rm -rf e2fsprogs-1.46.2
```

#### Additional filesystem tools

These are listed in dependency order to provide support for `btrfs`, `lvm`, and `raid`. These will also need to be enabled in the kernel if you installed your system on top of these technologies. Included are required libraries for compression and async IO, plus the controllers for software raid, logical volume management, and the `btrfs` filesystem itself.

#### LZO

```sh
tar xzvf /sources/lzo-2.10.tar.gz &&
cd        lzo-2.10                &&

./configure --prefix=/usr                    \
            --enable-shared                  \
            --disable-static                 \
            --docdir=/usr/share/doc/lzo-2.10 &&

make         &&
make install &&

cd .. &&
rm -rf lzo
```

#### libaio

```sh
tar xvf /sources/libaio_0.3.112.orig.tar.xz &&
cd       libaio-0.3.112                     &&

sed -i '/install.*libaio.a/s/^/#/' src/Makefile &&

make         &&
make install &&

cd .. &&
rm -rf libaio-0.3.112
```

#### mdadm

Optionally, `mdadm` has a test suite that be run with the following command:

```sh
./test --keep-going --logdir=test-logs --save-logs
```

But this test suite requires that the running kernel supports RAID and also that `mdadm` is not already running, so if you are building LFS on a system already using software RAID, you will not be able to run the tests.

```sh
tar xvf /sources/mdadm-4.1.tar.xz &&
cd       mdadm-4.1                &&

sed 's@-Werror@@' -i Makefile &&

make         &&
make install &&

cd .. &&
rm -rf mdadm-4.1
```

#### LVM2

```sh
tar xzvf /sources/LVM2.2.03.12.tgz &&
cd        LVM2.2.03.12             &&

SAVEPATH=$PATH                    &&
PATH=$PATH:/sbin:/usr/sbin        &&
./configure --prefix=/usr         \
            --exec-prefix=        \
            --enable-cmdlib       \
            --enable-pkgconfig    \
            --enable-udev_sync    \
            --with-thin-check=    \
            --with-thin-dump=     \
            --with-thin-repair=   \
            --with-thin-restore=  \
            --with-cache-check=   \
            --with-cache-dump=    \
            --with-cache-repair=  \
            --with-cache-restore= &&

make           &&
PATH=$SAVEPATH &&
unset SAVEPATH &&

make -C tools install_tools_dynamic     &&
make -C udev  install                   &&
make -C libdm install                   &&
rm test/shell/lvconvert-raid-reshape.sh &&
make S=shell/thin-flags.sh check_local  &&

make install &&

cd .. &&
rm -rf LVM2.2.03.12
```

#### btrfs

```sh
tar xvf /sources/btrfs-progs-v5.12.1.tar.xz &&
cd       btrfs-progs-v5.12.1                &&

./configure --prefix=/usr \
            --disable-documentation &&

make       &&
make fssum &&

pushd tests
   ./fsck-tests.sh
   ./mkfs-tests.sh
   ./cli-tests.sh
   ./convert-tests.sh
   ./misc-tests.sh
   ./fuzz-tests.sh
popd

make install &&

cd ..                                &&
umount btrfs-progs-v5.12.1/tests/mnt &&
rm -rf btrfs-progs-v5.12.1
```
