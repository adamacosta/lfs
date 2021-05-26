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
curl https://sqlite.org/2021/sqlite-autoconf-3340100.tar.gz -o sqlite-autoconf-3340100.tar.gz
curl https://sqlite.org/2021/sqlite-doc-3340100.zip -o sqlite-doc-3340100.zip
tar xzvf sqlite-autoconf-3340100.tar.gz
cd sqlite-autoconf-3340100

unzip -q ../sqlite-doc-3340100.zip
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
            -DSQLITE_ENABLE_FTS3_TOKENIZER=1"
make
sudo make install
sudo install -v -m755 -d /usr/share/doc/sqlite-3.34.1 &&
sudo cp -v -R sqlite-doc-3340100/* /usr/share/doc/sqlite-3.34.1

cd ..
rm -rf sqlite-autoconf-3340100
```

## Aspell

```sh
curl https://ftp.gnu.org/gnu/aspell/aspell-0.60.8.tar.gz -o aspell-0.60.8.tar.gz
tar xzvf aspell-0.60.8.tar.gz
cd aspell-0.60.8

./configure --prefix=/usr
make
sudo make install
sudo ln -svfn aspell-0.60 /usr/lib/aspell &&
sudo install -v -m755 -d /usr/share/doc/aspell-0.60.8/aspell{,-dev}.html &&
sudo install -v -m644 manual/aspell.html/* \
    /usr/share/doc/aspell-0.60.8/aspell.html &&
sudo install -v -m644 manual/aspell-dev.html/* \
    /usr/share/doc/aspell-0.60.8/aspell-dev.html &&
sudo install -v -m 755 scripts/ispell /usr/bin/ &&
sudo install -v -m 755 scripts/spell /usr/bin/

cd ..
rm -rf aspell-0.60.8
```

Now install a dictionary. Note that I am choosing English because I am an American English speaker, but you can go to the GNU Aspell ftp server and find a different dictionary if you prefer, or even install many.

```sh
curl https://ftp.gnu.org/gnu/aspell/dict/en/aspell6-en-2020.12.07-0.tar.bz2 -o aspell6-en-2020.12.07-0.tar.bz2
tar xvf aspell6-en-2020.12.07.0.tar.bz2
cd aspell6-en-2020.12.07

./configure
make
sudo make install

cd ..
rm -rf aspell6-2020.12.07
```

You can also dump the dictionary to a text file if you wish:

```sh
sudo sh -c 'aspell -d en dump master | aspell -l en expand >> /usr/share/dict/words'
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
curl https://ftp.gnu.org/gnu/parallel/parallel-20210522.tar.bz2 -o parallel-20210522.tar.bz2
tar xvf parallel-20210522.tar.bz2
cd parallel-20210522

./configure --prefix=/usr
make
sudo make install

cd ..
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