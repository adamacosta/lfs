# Additional system utilities

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
curl https://sqlite.org/2021/sqlite-autoconf-3340100.tar.gz -o /sources/sqlite-autoconf-3340100.tar.gz &&
curl https://sqlite.org/2021/sqlite-doc-3340100.zip -o /sources/sqlite-doc-3340100.zip &&

tar xzvf /sources/sqlite-autoconf-3340100.tar.gz &&
cd        sqlite-autoconf-3340100                &&

unzip -q /sources/sqlite-doc-3340100.zip &&

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
sudo install -v -m755 -d /usr/share/doc/sqlite-3.34.1           &&
sudo cp -v -R sqlite-doc-3340100/* /usr/share/doc/sqlite-3.34.1 &&

cd .. &&
rm -rf sqlite-autoconf-3340100
```

At this point, you may wish to rebuild `Python` to enable the optional loadable extension modules.

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

## enchant

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

## cpufrequtils

```sh
curl https://mirrors.edge.kernel.org/pub/linux/utils/kernel/cpufreq/cpufrequtils-008.tar.xz -o /sources/cpufrequtils-008.tar.xz &&

tar xvf /sources/cpufrequtils-008.tar.xz &&
cd       cpufrequtils-008                &&

make              &&
sudo make install &&

cd .. &&
rm -rf cpufrequtils-008
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
