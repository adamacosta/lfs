# Additional system utilities

## UnRar

Extract files from the Windows RAR archive.

```sh
curl https://www.rarlab.com/rar/unrarsrc-6.0.6.tar.gz -o /sources/unrarsrc-6.0.6.tar.gz &&

tar xzvf /sources/unrarsrc-6.0.6.tar.gz &&
cd        unrar                         &&

make -f makefile                   &&
sudo install -vm755 unrar /usr/bin &&

cd .. &&
rm -rf unrarsrc-6.0.6
```

## openldap (client only)

```sh
curl https://www.openldap.org/software/download/OpenLDAP/openldap-release/openldap-2.4.57.tgz -o openldap-2.4.57.tgz
curl https://www.linuxfromscratch.org/patches/blfs/10.1/openldap-2.4.57-consolidated-1.patch -o openldap-2.4.57-consolidated-1.patch
tar xzvf openldap-2.4.57.tgz
cd openldap-2.4.57

patch -Np1 -i ../openldap-2.4.57-consolidated-1.patch &&
autoconf &&
./configure --prefix=/usr     \
            --sysconfdir=/etc \
            --disable-static  \
            --enable-dynamic  \
            --disable-debug   \
            --disable-slapd
make depend &&
make -j
sudo make install

cd ..
rm -rf openldap-2.4.57
```

## sendmail

```sh
curl ftp://ftp.sendmail.org/pub/sendmail/sendmail.8.16.1.tar.gz -o sendmail.8.16.1.tar.gz
tar xzvf sendmail.8.16.1.tar.gz
cd sendmail-8.16.1

sudo groupadd -g 26 smmsp                               &&
sudo useradd -c "Sendmail Daemon" -g smmsp -d /dev/null \
        -s /bin/false -u 26 smmsp                  &&
sudo chmod -v 1777 /var/mail                            &&
sudo install -v -m700 -d /var/spool/mqueue

cat >> devtools/Site/site.config.m4 << "EOF"
APPENDDEF(`confENVDEF',`-DSTARTTLS -DLDAPMAP')
APPENDDEF(`confLIBS', `-lssl -lcrypto -lldap -llber')
EOF

cat >> devtools/Site/site.config.m4 << "EOF"
define(`confMANGRP',`root')
define(`confMANOWN',`root')
define(`confSBINGRP',`root')
define(`confUBINGRP',`root')
define(`confUBINOWN',`root')
EOF

sed -i 's|/usr/man/man|/usr/share/man/man|' \
    devtools/OS/Linux           &&
cd sendmail                     &&
sh Build                        &&
cd ../cf/cf                     &&
cp generic-linux.mc sendmail.mc &&
sh Build sendmail.cf

sudo install -v -d -m755 /etc/mail &&
sudo sh Build install-cf &&
cd ../..            &&
sudo sh Build install    &&
sudo install -v -m644 cf/cf/{submit,sendmail}.mc /etc/mail &&
sudo cp -v -R cf/* /etc/mail                               &&
sudo install -v -m755 -d /usr/share/doc/sendmail-8.16.1/{cf,sendmail} &&
sudo install -v -m644 CACerts FAQ KNOWNBUGS LICENSE PGPKEYS README RELEASE_NOTES \
        /usr/share/doc/sendmail-8.16.1 &&
sudo install -v -m644 sendmail/{README,SECURITY,TRACEFLAGS,TUNING} \
        /usr/share/doc/sendmail-8.16.1/sendmail &&
sudo install -v -m644 cf/README /usr/share/doc/sendmail-8.16.1/cf &&
for manpage in sendmail editmap mailstats makemap praliases smrsh
do
    sudo install -v -m644 $manpage/$manpage.8 /usr/share/man/man8
done &&
sudo install -v -m644 sendmail/aliases.5    /usr/share/man/man5 &&
sudo install -v -m644 sendmail/mailq.1      /usr/share/man/man1 &&
sudo install -v -m644 sendmail/newaliases.1 /usr/share/man/man1 &&
sudo install -v -m644 vacation/vacation.1   /usr/share/man/man1

cd doc/op                                       &&
sed -i 's/groff/GROFF_NO_SGR=1 groff/' Makefile &&
make op.txt

sudo install -v -d -m755 /usr/share/doc/sendmail-8.16.1 &&
sudo install -v -m644 op.ps op.txt /usr/share/doc/sendmail-8.16.1

cd ../../..
rm -rf sendmail-8.16.1
```

## SQLite

```sh
curl https://sqlite.org/2021/sqlite-autoconf-3350500.tar.gz -o /sources/sqlite-autoconf-3350500.tar.gz &&
curl https://sqlite.org/2021/sqlite-doc-3350500.zip -o /sources/sqlite-doc-3350500.zip &&

tar xzvf /sources/sqlite-autoconf-3350500.tar.gz &&
cd        sqlite-autoconf-3350500                &&

unzip -q /sources/sqlite-doc-3350500.zip &&

./configure --prefix=/usr     \
            --disable-static  \
            --enable-fts5     \
            CFLAGS="-g -O2                    \
            -DSQLITE_ENABLE_FTS3=1            \
            -DSQLITE_ENABLE_FTS4=1            \
            -DSQLITE_ENABLE_COLUMN_METADATA=1 \
            -DSQLITE_ENABLE_UNLOCK_NOTIFY=1   \
            -DSQLITE_ENABLE_DBSTAT_VTAB=1     \
            -DSQLITE_SECURE_DELETE=1          \
            -DSQLITE_ENABLE_FTS3_TOKENIZER=1" &&

make                                                            &&
sudo make install                                               &&
sudo install -v -m755 -d /usr/share/doc/sqlite-3.35.5           &&
sudo cp -v -R sqlite-doc-3350500/* /usr/share/doc/sqlite-3.35.5 &&

cd .. &&
rm -rf sqlite-autoconf-3350500
```

At this point, you may wish to rebuild `Python` to enable the optional loadable extension modules.

## sharutils

Utilities for creating shell archives.

```sh
curl https://ftp.gnu.org/gnu/sharutils/sharutils-4.15.2.tar.xz -o /sources/sharutils-4.15.2.tar.xz &&

tar xvf /sources/sharutils-4.15.2.tar.xz &&
cd       sharutils-4.15.2                &&

sed -i 's/BUFSIZ/rw_base_size/' src/unshar.c    &&
sed -i '/program_name/s/^/extern /' src/*opts.h &&

sed -i 's/IO_ftrylockfile/IO_EOF_SEEN/' lib/*.c        &&
echo "#define _IO_IN_BACKUP 0x100" >> lib/stdio-impl.h &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf sharutils-4.15.2
```

## BerkeleyDB

```sh
curl https://anduin.linuxfromscratch.org/BLFS/bdb/db-5.3.28.tar.gz -o /sources/db-5.3.28.tar.gz &&

tar xzvf /sources/db-5.3.28.tar.gz &&
cd        db-5.3.28                &&

sed -i 's/\(__atomic_compare_exchange\)/\1_db/' src/dbinc/atomic.h &&

cd build_unix                        &&
../dist/configure --prefix=/usr      \
                  --enable-compat185 \
                  --enable-dbm       \
                  --disable-static   \
                  --enable-cxx       &&

make                                              &&
sudo make docdir=/usr/share/doc/db-5.3.28 install &&

sudo chown -v -R root:root                   \
      /usr/bin/db_*                          \
      /usr/include/db{,_185,_cxx}.h          \
      /usr/lib/libdb*.{so,la}                \
      /usr/share/doc/db-5.3.28 &&

cd ../.. &&
rm -rf db-5.3.28
```

## Aspell

```sh
curl https://ftp.gnu.org/gnu/aspell/aspell-0.60.8.tar.gz -o /sources/aspell-0.60.8.tar.gz &&

tar xzvf /sources/aspell-0.60.8.tar.gz &&
cd        aspell-0.60.8                &&

./configure --prefix=/usr &&

make                                                                     &&
sudo make install                                                        &&
sudo ln -svfn aspell-0.60 /usr/lib/aspell                                &&
sudo install -v -m755 -d /usr/share/doc/aspell-0.60.8/aspell{,-dev}.html &&
sudo install -v -m644 manual/aspell.html/* \
    /usr/share/doc/aspell-0.60.8/aspell.html                             &&
sudo install -v -m644 manual/aspell-dev.html/* \
    /usr/share/doc/aspell-0.60.8/aspell-dev.html                         &&
sudo install -v -m 755 scripts/ispell /usr/bin/                          &&
sudo install -v -m 755 scripts/spell /usr/bin/                           &&

cd .. &&
rm -rf aspell-0.60.8
```

Now install a dictionary. Note that I am choosing English because I am an American English speaker, but you can go to the GNU Aspell ftp server and find a different dictionary if you prefer, or even install many.

```sh
curl https://ftp.gnu.org/gnu/aspell/dict/en/aspell6-en-2020.12.07-0.tar.bz2 -o /sources/aspell6-en-2020.12.07-0.tar.bz2 &&

tar xvf /sources/aspell6-en-2020.12.07-0.tar.bz2 &&
cd       aspell6-en-2020.12.07-0                 &&

./configure &&

make              &&
sudo make install &&

cd .. &&
rm -rf aspell6-2020.12.07-0
```

You can also dump the dictionary to a text file if you wish:

```sh
sudo sh -c 'aspell -d en dump master | aspell -l en expand >> /usr/share/dict/words'
```

## Enchant

Interface to spell checking libraries.

```sh
curl https://github.com/AbiWord/enchant/releases/download/v2.2.15/enchant-2.2.15.tar.gz -o /sources/enchant-2.2.15.tar.gz &&

tar xzvf /sources/enchant-2.2.15.tar.gz &&
cd        enchant-2.2.15                &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf enchant-2.2.15
```

## Hunspell

Additional spell checking libraries used by various desktop programs.

```sh
curl https://github.com/hunspell/hunspell/files/2573619/hunspell-1.7.0.tar.gz -o /sources/hunspell-1.7.0.tar.gz &&

tar xzvf /sources/hunspell-1.7.0.tar.gz &&
cd        hunspell-1.7.0                &&

autoreconf  -vfi &&

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/hunspell-1.7.0 &&

make              &&
sudo make install &&

cd .. &&
rm -rf hunspell-1.7.0
```

## mutt

```sh
curl https://bitbucket.org/mutt/mutt/downloads/mutt-2.0.5.tar.gz -o mutt-2.0.5.tar.gz
tar xzvf mutt-2.0.5.tar.gz
cd mutt-2.0.5

sed -i -e 's/ -with_backspaces//' -e 's/elinks/links/' \
  -e 's/-no-numbering -no-references//' doc/Makefile.in
./configure --prefix=/usr                           \
            --sysconfdir=/etc                       \
            --with-docdir=/usr/share/doc/mutt-2.0.5 \
            --with-ssl                              \
            --enable-external-dotlock               \
            --enable-pop                            \
            --enable-imap                           \
            --enable-hcache                         \
            --enable-sidebar
make
sudo make install

cd ..
rm -rf mutt-2.0.5
```

To utilize `gpg` for mail:

```sh
cat /usr/share/doc/mutt-2.0.5/samples/gpg.rc >> ~/.muttrc
```

## at

```sh
curl http://software.calhariz.com/at/at_3.2.1.orig.tar.gz -o at_3.2.1.orig.tar.gz
tar xzvf at_3.2.1.orig.tar.gz
cd at-3.2.1

sudo groupadd -g 17 atd &&
sudo useradd -d /dev/null -c "atd daemon" -g atd -s /bin/false -u 17 atd

./configure --with-daemon_username=atd        \
            --with-daemon_groupname=atd       \
            SENDMAIL=/usr/sbin/sendmail       \
            --with-jobdir=/var/spool/atjobs   \
            --with-atspool=/var/spool/atspool \
            --with-systemdsystemunitdir=/lib/systemd/system
make
sudo make install docdir=/usr/share/doc/at-3.2.1 \
                  atdocdir=/usr/share/doc/at-3.2.1

cd ..
rm -rf at-3.2.1
```

## parallel

```sh
curl https://ftp.gnu.org/gnu/parallel/parallel-20210522.tar.bz2 -o /sources/parallel-20210522.tar.bz2 &&

tar xvf /sources/parallel-20210522.tar.bz2 &&
cd       parallel-20210522                 &&

./configure --prefix=/usr \
            --docdir=/usr/share/doc/parallel-20210522 &&

make              &&
sudo make install &&

cd .. &&
rm -rf parallel-20210522
```

## cpio

```sh
curl https://ftp.gnu.org/gnu/cpio/cpio-2.13.tar.bz2 -o cpio-2.13.tar.bz2
tar xvf cpio-2.13.tar.bz2
cd cpio-2.13

sed -i '/The name/,+2 d' src/global.c
./configure --prefix=/usr \
            --bindir=/bin \
            --enable-mt   \
            --with-rmt=/usr/libexec/rmt
make -j
makeinfo --html            -o doc/html      doc/cpio.texi &&
makeinfo --html --no-split -o doc/cpio.html doc/cpio.texi &&
makeinfo --plaintext       -o doc/cpio.txt  doc/cpio.texi
sudo make install &&
sudo install -v -m755 -d /usr/share/doc/cpio-2.13/html &&
sudo install -v -m644    doc/html/* \
                    /usr/share/doc/cpio-2.13/html &&
sudo install -v -m644    doc/cpio.{html,txt} \
                    /usr/share/doc/cpio-2.13

cd ..
rm -rf cpio-2.13
```

## fcron

```sh
curl http://fcron.free.fr/archives/fcron-3.2.1.src.tar.gz -o fcron-3.2.1.src.tar.gz
tar xzvf fcron-3.2.1.src.tar.gz
cd fcron-3.2.1

sudo groupadd -g 22 fcron &&
sudo useradd -d /dev/null -c "Fcron User" -g fcron -s /bin/false -u 22 fcron

./configure --prefix=/usr          \
            --sysconfdir=/etc      \
            --localstatedir=/var   \
            --without-sendmail     \
            --with-piddir=/run     \
            --with-boot-install=no
make -j
sudo make install

cd ..
rm -rf fcron-3.2.1
```

## PCRE2

```sh
curl https://ftp.pcre.org/pub/pcre/pcre2-10.37.tar.bz2 -o /sources/pcre2-10.37.tar.bz2 &&

tar xvf /sources/pcre2-10.37.tar.bz2 &&
cd       pcre2-10.37                 &&

./configure --prefix=/usr                       \
            --docdir=/usr/share/doc/pcre2-10.37 \
            --enable-unicode                    \
            --enable-jit                        \
            --enable-pcre2-16                   \
            --enable-pcre2-32                   \
            --enable-pcre2grep-libz             \
            --enable-pcre2grep-libbz2           \
            --enable-pcre2test-libreadline      \
            --disable-static                    &&

make              &&
sudo make install &&

cd .. &&
rm -rf pcre2-10.37
```

## pciutils

```sh
curl https://www.kernel.org/pub/software/utils/pciutils/pciutils-3.7.0.tar.xz -o /sources/pciutils-3.7.0.tar.xz &&

tar xvf /sources/pciutils-3.7.0.tar.xz &&
cd       pciutils-3.7.0                &&

make PREFIX=/usr                \
     SHAREDIR=/usr/share/hwdata \
     SHARED=yes &&

sudo make PREFIX=/usr                \
          SHAREDIR=/usr/share/hwdata \
          SHARED=yes                 \
          install install-lib &&

sudo chmod -v 755 /usr/lib/libpci.so &&

cd .. &&
rm -rf pciutils-3.7.0
```

## libusb

```sh
curl https://github.com/libusb/libusb/releases/download/v1.0.24/libusb-1.0.24.tar.bz2 -o /sources/libusb-1.0.24.tar.bz2 &&

tar xvf /sources/libusb-1.0.24.tar.bz2 &&
cd       libusb-1.0.24                 &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libusb-1.0.24
```

## usbutils

```sh
curl https://www.kernel.org/pub/linux/utils/usb/usbutils/usbutils-013.tar.xz -o /sources/usbutils-013.tar.xz &&

tar xvf /sources/usbutils-013.tar.xz &&
cd       usbutils-013                &&

./autogen.sh --prefix=/usr \
            --datadir=/usr/share/hwdata &&

make                                                                    &&
sudo make install                                                       &&
sudo install -dm755 /usr/share/hwdata/                                  &&
sudo wget http://www.linux-usb.org/usb.ids -O /usr/share/hwdata/usb.ids &&

cd .. &&
rm -rf usbutils-013
```

### Configuring usbutils

Create a timed service to update the ids weekly:

```sh
sudo su -
cat > /usr/lib/systemd/system/update-usbids.service << "EOF" &&
[Unit]
Description=Update usb.ids file
Documentation=man:lsusb(8)
DefaultDependencies=no
After=local-fs.target network-online.target
Before=shutdown.target

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/bin/wget http://www.linux-usb.org/usb.ids -O /usr/share/hwdata/usb.ids
EOF
cat > /usr/lib/systemd/system/update-usbids.timer << "EOF" &&
[Unit]
Description=Update usb.ids file weekly

[Timer]
OnCalendar=Sun 03:00:00
Persistent=true

[Install]
WantedBy=timers.target
EOF
systemctl enable update-usbids.timer
exit
```

## libburn

```sh
curl https://files.libburnia-project.org/releases/libburn-1.5.4.tar.gz -o /sources/libburn-1.5.4.tar.gz &&

tar xzvf /sources/libburn-1.5.4.tar.gz &&
cd        libburn-1.5.4                &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libburn-1.5.4
```

## libisofs

```sh
curl https://files.libburnia-project.org/releases/libisofs-1.5.4.tar.gz -o /sources/libisofs-1.5.4.tar.gz &&

tar xzvf /sources/libisofs-1.5.4.tar.gz &&
cd        libisofs-1.5.4                &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libisofs-1.5.4
```

## libisoburn

```sh
curl https://files.libburnia-project.org/releases/libisoburn-1.5.4.tar.gz -o /sources/libisoburn-1.5.4.tar.gz &&

tar xzvf /sources/libisoburn-1.5.4.tar.gz &&
cd        libisoburn-1.5.4                &&

./configure --prefix=/usr              \
            --disable-static           \
            --enable-pkg-check-modules &&

make              &&
sudo make install &&

cd .. &&
rm -rf libisoburn-1.5.4
```

## libcdio

```sh
curl https://ftp.gnu.org/gnu/libcdio/libcdio-2.1.0.tar.bz2 -o /sources/libcdio-2.1.0.tar.bz2 &&
curl https://ftp.gnu.org/gnu/libcdio/libcdio-paranoia-10.2+2.0.1.tar.bz2 -o /sources/libcdio-paranoia-10.2+2.0.1.tar.bz2 &&

tar xvf /sources/libcdio-2.1.0.tar.bz2 &&
cd       libcdio-2.1.0                 &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd ..                &&
rm -rf libcdio-2.1.0 &&

tar xvf /sources/libcdio-paranoia-10.2+2.0.1.tar.bz2 &&
cd       libcdio-paranoia-10.2+2.0.1                 &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libcdio-paranoia-10.2+2.0.1
```

## libdvdread

```sh
curl https://get.videolan.org/libdvdread/6.1.2/libdvdread-6.1.2.tar.bz2 -o /sources/libdvdread-6.1.2.tar.bz2 &&

tar xvf /sources/libdvdread-6.1.2.tar.bz2 &&
cd       libdvdread-6.1.2                 &&

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/libdvdread-6.1.2 &&

make              &&
sudo make install &&

cd .. &&
rm -rf libdvdread-6.1.2
```

## parted

Use the `--disable-device-mapper` flag to `configure` if you did not install `LVM2`.

```sh
curl https://ftp.gnu.org/gnu/parted/parted-3.4.tar.xz -o /sources/parted-3.4.tar.xz &&

tar xvf /sources/parted-3.4.tar.xz &&
cd       parted-3.4                &&

./configure --prefix=/usr \
            --disable-static &&

make                                                   &&
make -C doc html                                       &&
makeinfo --html      -o doc/html       doc/parted.texi &&
makeinfo --plaintext -o doc/parted.txt doc/parted.texi &&
sudo make -k check &&
sudo make install  &&

sudo install -v -m755 -d /usr/share/doc/parted-3.4/html &&
sudo install -v -m644    doc/html/* \
                        /usr/share/doc/parted-3.4/html  &&
sudo install -v -m644    doc/{FAT,API,parted.{txt,html}} \
                        /usr/share/doc/parted-3.4       &&

cd .. &&
sudo rm -rf parted-3.4
```

## sysstat

```sh
curl http://sebastien.godard.pagesperso-orange.fr/sysstat-12.5.4.tar.xz -o /sources/sysstat-12.5.4.tar.xz &&

tar xvf /sources/sysstat-12.5.4.tar.xz &&
cd       sysstat-12.5.4                &&

sa_lib_dir=/usr/lib/sa          \
sa_dir=/var/log/sa              \
conf_dir=/etc/sysconfig         \
./configure --prefix=/usr       \
            --disable-file-attr \
            --docdir=/usr/share/doc/sysstat-12.5 &&

make              &&
sudo make install &&

sudo install -v -m644 sysstat.service /usr/lib/systemd/system/sysstat.service &&
sudo sed -i "/^Also=/d" /usr/lib/systemd/system/sysstat.service               &&

cd .. &&
rm -rf sysstat-12.5.4
```

## libtirpc

```sh
curl https://downloads.sourceforge.net/libtirpc/libtirpc-1.3.2.tar.bz2 -o /sources/libtirpc-1.3.2.tar.bz2 &&

tar xvf /sources/libtirpc-1.3.2.tar.bz2 &&
cd       libtirpc-1.3.2                 &&

./configure --prefix=/usr     \
            --sysconfdir=/etc \
            --disable-static  \
            --disable-gssapi &&

make              &&
sudo make install &&

cd .. &&
rm -rf libtirpc-1.3.2
```

## lsof

```sh
curl https://www.mirrorservice.org/sites/lsof.itap.purdue.edu/pub/tools/unix/lsof/lsof_4.91.tar.gz -o /sources/lsof_4.91.tar.gz &&

tar xzvf /sources/lsof_4.91.tar.gz &&
cd        lsof_4.91                &&

tar -xf lsof_4.91_src.tar  &&
cd lsof_4.91_src           &&
./Configure -n linux       &&

make CFGL="-L./lib -ltirpc" &&

sudo install -v -m0755 -o root -g root lsof /usr/bin &&
sudo install -v lsof.8 /usr/share/man/man8           &&

cd .. &&
rm -rf lsof_4.91
```

## bubblewrap

```sh
curl https://github.com/projectatomic/bubblewrap/releases/download/v0.4.1/bubblewrap-0.4.1.tar.xz -o /sources/bubblewrap-0.4.1.tar.xz &&

tar xvf /sources/bubblewrap-0.4.1.tar.xz &&
cd       bubblewrap-0.4.1                &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf bubblewrap-0.4.1
```

## FUSE

The filesystem in userspace. Requires settings in the kernel.

```
File systems  --->
  <*/M> FUSE (Filesystem in Userspace) support  [CONFIG_FUSE_FS]
  <*/M>   Character device in Userspace support [CONFIG_CUSE]
```

```sh
curl https://github.com/libfuse/libfuse/releases/download/fuse-3.10.4/fuse-3.10.4.tar.xz -o /sources/fuse-3.10.4.tar.xz &&

tar xvf /sources/fuse-3.10.4.tar.xz &&
cd       fuse-3.10.4                &&

sed -i '/^udev/,$ s/^/#/' util/meson.build &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

sudo chmod u+s /usr/bin/fusermount3                 &&
sudo install -v -m755 -d /usr/share/doc/fuse-3.10.4 &&
sudo install -v -m644    ../doc/{README.NFS,kernel.txt} \
                         /usr/share/doc/fuse-3.10.4 &&
sudo cp -Rv ../doc/html  /usr/share/doc/fuse-3.10.4 &&

cd ../.. &&
rm -rf fuse-3.10.4
```

## libatasmart

```sh
curl http://0pointer.de/public/libatasmart-0.19.tar.xz -o /sources/libatasmart-0.19.tar.xz &&

tar xvf /sources/libatasmart-0.19.tar.xz &&
cd       libatasmart-0.19                &&

./configure --prefix=/usr \
            --disable-static &&

make                                                     &&
sudo make docdir=/usr/share/doc/libatasmart-0.19 install &&

cd .. &&
rm -rf libatasmart-0.19
```

## libbytesize

```sh
curl https://github.com/storaged-project/libbytesize/releases/download/2.5/libbytesize-2.5.tar.gz -o /sources/libbytesize-2.5.tar.gz &&

tar xzvf /sources/libbytesize-2.5.tar.gz &&
cd        libbytesize-2.5                &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf libbytesize-2.5
```

## popt

Command-line option parser. If `doxygen` is not installed, remove the calls to build and install the API documentation.

```sh
curl http://ftp.rpm.org/popt/releases/popt-1.x/popt-1.18.tar.gz -o /sources/popt-1.18.tar.gz &&

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

## cryptsetup

Uses the kernel crypto API to encrypt block devices. Requires kernel settings.

```
Device Drivers  --->
  [*] Multiple devices driver support (RAID and LVM) ---> [CONFIG_MD]
       <*/M> Device mapper support                        [CONFIG_BLK_DEV_DM]
       <*/M> Crypt target support                         [CONFIG_DM_CRYPT]

Cryptographic API  --->
  <*/M> XTS support                                       [CONFIG_CRYPTO_XTS]
  <*/M> SHA224 and SHA256 digest algorithm                [CONFIG_CRYPTO_SHA256]
  <*/M> AES cipher algorithms                             [CONFIG_CRYPTO_AES]
  <*/M> User-space interface for symmetric key cipher algorithms
                                                          [CONFIG_CRYPTO_USER_API_SKCIPHER]
  For tests:
  <*/M> Twofish cipher algorithm                          [CONFIG_CRYPTO_TWOFISH]
```

```sh
curl https://www.kernel.org/pub/linux/utils/cryptsetup/v2.3/cryptsetup-2.3.6.tar.xz -o /sources/cryptsetup-2.3.6.tar.xz &&

tar xvf /sources/cryptsetup-2.3.6.tar.xz &&
cd       cryptsetup-2.3.6                &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf cryptsetup-2.3.6
```

## VolumeKey

Library for handling volume encryption keys. `SWIG` is needed to build the Python bindings. Otherwise, pass the `--without-python3` flag to `configure`.

```sh
curl https://github.com/felixonmars/volume_key/archive/volume_key-0.3.12.tar.gz -o /sources/volume_key-0.3.12.tar.gz &&

tar xzvf /sources/volume_key-0.3.12.tar.gz &&
cd        volume_key-volume_key-0.3.12     &&

autoreconf  -fiv             &&
./configure --prefix=/usr    \
            --without-python &&

make              &&
sudo make install &&

cd .. &&
rm -rf volume_key-volume_key-0.3.12
```

## libblockdev

```sh
curl https://github.com/storaged-project/libblockdev/releases/download/2.25-1/libblockdev-2.25.tar.gz -o /sources/libblockdev-2.25.tar.gz &&

tar xzvf /sources/libblockdev-2.25.tar.gz &&
cd        libblockdev-2.25                &&

sed 's/g_memdup/&2/' -i             \
    src/lib/plugin_apis/vdo.{c,api} \
    src/plugins/vdo.c &&

./configure --prefix=/usr     \
            --sysconfdir=/etc \
            --with-python3    \
            --without-gtk-doc \
            --without-nvdimm  \
            --without-dm      &&

make              &&
sudo make install &&

cd .. &&
rm -rf libblockdev-2.25
```

## gptfdisk

Utility for reading and manipulating GUID partition tables.

```sh
curl https://downloads.sourceforge.net/gptfdisk/gptfdisk-1.0.8.tar.gz -o /sources/gptfdisk-1.0.8.tar.gz &&
curl https://www.linuxfromscratch.org/patches/blfs/svn/gptfdisk-1.0.8-convenience-1.patch -o /sources/patches/gptfdisk-1.0.8-convenience-1.patch &&

tar xzvf /sources/gptfdisk-1.0.8.tar.gz &&
cd        gptfdisk-1.0.8                &&

patch -Np1 -i /sources/patches/gptfdisk-1.0.8-convenience-1.patch &&
sed   -i 's|ncursesw/||' gptcurses.cc                             &&

make              &&
sudo make install &&

cd .. &&
rm -rf gptfdisk-1.0.8
```

## dosfstools

Be sure to enable the dos filesystems in the kernel if you need them. EFI boot partitions require VFAT.

```
File systems --->
  <DOS/FAT/EXFAT/NT Filesystems --->
    <*/M> MSDOS fs support             [CONFIG_MSDOS_FS]
    <*/M> VFAT (Windows-95) fs support [CONFIG_VFAT_FS]
```

```sh
curl https://github.com/dosfstools/dosfstools/releases/download/v4.2/dosfstools-4.2.tar.gz -o /sources/dosfstools-4.2.tar.gz &&

tar xzvf /sources/dosfstools-4.2.tar.gz &&
cd        dosfstools-4.2                &&

./configure --prefix=/usr            \
            --enable-compat-symlinks \
            --mandir=/usr/share/man  \
            --docdir=/usr/share/doc/dosfstools-4.2 &&

make              &&
sudo make install &&

cd .. &&
rm -rf dosfstools-4.2
```

## xfsprogs

Programs for the xfs filesystem. To enable the kernel drivers.

```
File systems --->
  <*/M> XFS filesystem support [CONFIG_XFS_FS]
```

```sh
curl https://www.kernel.org/pub/linux/utils/fs/xfs/xfsprogs/xfsprogs-5.12.0.tar.xz -o /sources/xfsprogs-5.12.0.tar.xz &&

tar xvf /sources/xfsprogs-5.12.0.tar.xz &&
cd       xfsprogs-5.12.0                &&

make DEBUG=-DNDEBUG     \
     INSTALL_USER=root  \
     INSTALL_GROUP=root &&

sudo make PKG_DOC_DIR=/usr/share/doc/xfsprogs-5.12.0 install     &&
sudo make PKG_DOC_DIR=/usr/share/doc/xfsprogs-5.12.0 install-dev &&

sudo rm -rfv /usr/lib/libhandle.{a,la} &&

cd .. &&
rm -rf xfsprogs-5.12.0
```

## udisks

```sh
curl https://github.com/storaged-project/udisks/releases/download/udisks-2.9.2/udisks-2.9.2.tar.bz2 -o /sources/udisks-2.9.2.tar.bz2 &&

tar xvf /sources/udisks-2.9.2.tar.bz2 &&
cd       udisks-2.9.2                 &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static     &&

make              &&
sudo make install &&

cd .. &&
rm -rf udisks-2.9.2
```

## CrackLib

```sh
curl https://github.com/cracklib/cracklib/releases/download/v2.9.7/cracklib-2.9.7.tar.bz2 -o /sources/cracklib-2.9.7.tar.bz2 &&
curl https://github.com/cracklib/cracklib/releases/download/v2.9.7/cracklib-words-2.9.7.bz2 -o /sources/cracklib-words-2.9.7.bz2 &&

tar xvf /sources/cracklib-2.9.7.tar.bz2 &&
cd       cracklib-2.9.7                 &&

sed -i '/skipping/d' util/packer.c &&

PYTHON=python3 CPPFLAGS=-I/usr/include/python3.9 \
./configure --prefix=/usr    \
            --disable-static \
            --with-default-dict=/usr/lib/cracklib/pw_dict &&

make              &&
sudo make install &&

sudo install -v -m644 -D      /sources/cracklib-words-2.9.7.bz2 \
                              /usr/share/dict/cracklib-words.bz2    &&

sudo bunzip2 -v               /usr/share/dict/cracklib-words.bz2    &&
sudo ln -v -sf cracklib-words /usr/share/dict/words                 &&
sudo install -v -m755 -d      /usr/lib/cracklib                     &&

sudo create-cracklib-dict     /usr/share/dict/cracklib-words        &&

cd .. &&
rm -rf cracklib-2.9.7
```

## libpwquality

```sh
curl https://github.com/libpwquality/libpwquality/releases/download/libpwquality-1.4.4/libpwquality-1.4.4.tar.bz2 -o /sources/libpwquality-1.4.4.tar.bz2 &&

tar xvf /sources/libpwquality-1.4.4.tar.bz2 &&
cd       libpwquality-1.4.4                 &&

./configure --prefix=/usr                      \
            --disable-static                   \
            --with-securedir=/usr/lib/security \
            --with-python-binary=python3 &&

make              &&
sudo make install &&

cd .. &&
rm -rf libpwquality-1.4.4
```

## AccountsService

```sh
curl https://www.freedesktop.org/software/accountsservice/accountsservice-0.6.55.tar.xz -o /sources/accountsservice-0.6.55.tar.xz &&

tar xvf /sources/accountsservice-0.6.55.tar.xz &&
cd       accountsservice-0.6.55                &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Dadmin_group=adm   \
      -Dsystemd=true      \
      .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf accountsservice-0.6.55
```

## Kerberos

```sh
curl https://kerberos.org/dist/krb5/1.19/krb5-1.19.1.tar.gz -o /sources/krb5-1.19.1.tar.gz &&

tar xzvf /sources/krb5-1.19.1.tar.gz &&
cd        krb5-1.19.1                &&

cd src &&

sed -i -e 's@\^u}@^u cols 300}@' tests/dejagnu/config/default.exp     &&
sed -i -e '/eq 0/{N;s/12 //}'    plugins/kdb/db2/libdb2/test/run.test &&
sed -i '/t_iprop.py/d'           tests/Makefile.in                    &&

./configure --prefix=/usr            \
            --sysconfdir=/etc        \
            --localstatedir=/var/lib \
            --runstatedir=/run       \
            --with-system-et         \
            --with-system-ss         \
            --with-system-verto=no   \
            --enable-dns-for-realm &&

make              &&
sudo make install &&

sudo install -v -dm755 /usr/share/doc/krb5-1.19.1 &&
sudo cp -vfr ../doc/*  /usr/share/doc/krb5-1.19.1 &&

cd .. &&
rm -rf krb5-1.19.1
```

`Kerberos` requires some configuration if you are actually in a network domain, but the libraries alone are also required by GNOME Control Center.

## sshfs

Allows you to mount a remote directory tree as if it were local from any host you have `ssh` access to. Not particularly performant or robuts, but very handy for quickly using files on a remote host without needing to manually `ssh` and `scp` back and forth.

```sh
curl https://github.com/libfuse/sshfs/releases/download/sshfs-3.7.2/sshfs-3.7.2.tar.xz -o /sources/sshfs-3.7.2.tar.xz &&

tar xvf /sources/sshfs-3.7.2.tar.xz &&
cd       sshfs-3.7.2                &&

mkdir build &&
cd    build &&

meson --prefix=/usr .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf sshfs-3.7.2
```

To mount an ssh server you need to be able to log into the server. For example, to mount your remote home folder to the local `~/examplepath` (the directory must exist and you must have permissions to write to it):

```sh
sshfs example.com:/home/userid ~/examplepath
```
When you've finished work and want to unmount it again:
```sh
fusermount3 -u ~/example`
```
You can also mount an sshfs filesystem at boot by adding an entry similar to the following in the /etc/fstab file:

```
userid@example.com:/path /media/path fuse.sshfs _netdev,IdentityFile=/home/userid/.ssh/id_rsa 0 0
```

## cpufrequtils

Interact with the CPU governors from userspace.

```sh
curl https://mirrors.edge.kernel.org/pub/linux/utils/kernel/cpufreq/cpufrequtils-008.tar.xz -o /sources/cpufrequtils-008.tar.xz &&

tar xvf /sources/cpufrequtils-008.tar.xz &&
cd       cpufrequtils-008                &&

make                                    &&
sudo make install mandir=/usr/share/man &&

cd .. &&
rm -rf cpufrequtils-008
```

## lm-sensors

Library and utilities for exporting hardware monitoring information from the Linux kernel to userspace.

```sh
curl https://github.com/lm-sensors/lm-sensors/archive/refs/tags/V3-6-0.tar.gz -o /sources/lm-sensors-3.6.0.tar.gz &&

tar xzvf /sources/lm-sensors-3.6.0.tar.gz &&
cd        lm-sensors-3-6-0                &&

make BUILD_STATIC_LIB=0                 &&
sudo make PREFIX=/usr        \
          BUILD_STATIC_LIB=0 \
          MANDIR=/usr/share/man install &&

cd .. &&
rm -rf lm-sensors-3-6-0
```

## htop

An enhanced, interactive version of `top`.

```sh
curl https://github.com/htop-dev/htop/archive/refs/tags/3.0.5.tar.gz -o /sources/htop-3.0.5.tar.gz &&

tar xzvf /sources/htop-3.0.5.tar.gz &&
cd        htop-3.0.5                &&

./autogen.sh &&

./configure --prefix=/usr    \
            --enable-unicode \
            --enable-sensors \
            --enable-hwloc &&

make              &&
sudo make install &&

cd .. &&
rm -rf htop-3.0.5
```
