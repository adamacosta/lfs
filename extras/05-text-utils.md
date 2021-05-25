# Structured data and text processing utilities

## jq

First, install the `oniguruma` regex library this depends on:

```sh
curl -L https://github.com/kkos/oniguruma/releases/download/v6.9.7.1/onig-6.9.7.1.tar.gz -o onig-6.9.7.1.tar.gz
tar xzvf onig-6.9.7.1.tar.gz
cd onig-6.9.7.1

./configure --prefix=/usr
make -j
sudo make install

cd ..
rm -rf onig-6.9.7.1
```

```sh
curl -L https://github.com/stedolan/jq/releases/download/jq-1.6/jq-1.6.tar.gz -o jq-1.6.tar.gz
tar zavf jq-1.6.tar.gz
cd jq-1.6

./configure --prefix=/usr
make -j
sudo make install

cd ..
rm -rf jq-1.6
```

## yq

This is intended to be a fully portable binary installation, given it is a statically-linked golang binary. In lieu of having to install golang, just download a binary release:

```sh
sudo wget https://github.com/mikefarah/yq/releases/download/v4.2.0/yq_linux_amd64 -O /usr/bin/yq &&\
    sudo chmod +x /usr/bin/yq
```

## icu

```sh
curl -L http://github.com/unicode-org/icu/releases/download/release-68-2/icu4c-68_2-src.tgz -o icu4c-68_2-src.tgz
tar xzvf icu4c-68_2-src.tgz
cd icu/source

./configure --prefix=/usr
make -j
sudo make install

cd ..
rm -rf icu
```

## libxml2

```sh
curl http://xmlsoft.org/sources/libxml2-2.9.10.tar.gz -o libxml2-2.9.10.tar.gz
curl -L http://www.linuxfromscratch.org/patches/blfs/10.1/libxml2-2.9.10-security_fixes-1.patch -o libxml2-2.9.10-security_fixes-1.patch
tar xzvf libxml2-2.9.10.tar.gz

patch -p1 -i ../libxml2-2.9.10-security_fixes-1.patch
sed -i '/if Py/{s/Py/(Py/;s/)/))/}' python/{types.c,libxml.c}
sed -i 's/test.test/#&/' python/tests/tstLastError.py
sed -i 's/ TRUE/ true/' encoding.c
./configure --prefix=/usr    \
            --disable-static \
            --with-history   \
            --with-python=/usr/bin/python3
make -j
sudo make install

cd ..
rm -rf libxml2-2.9.10
```

## Zip

```sh
curl https://downloads.sourceforge.net/infozip/zip30.tar.gz -o zip30.tar.gz
tar xzvf zip30.tar.gz
cd zip30

make -f unix/Makefile generic_gcc
sudo make prefix=/usr MANDIR=/usr/share/man/man1 -f unix/Makefile install

cd ..
rm -rf zip30
```

## Unzip

```sh
curl https://downloads.sourceforge.net/infozip/unzip60.tar.gz -o unzip60.tar.gz
curl http://www.linuxfromscratch.org/patches/blfs/10.1/unzip-6.0-consolidated_fixes-1.patch -o unzip-6.0-consolidated_fixes-1.patch
tar xzvf unzip60.tar.gz
cd unzip60

patch -Np1 -i ../unzip-6.0-consolidated_fixes-1.patch
make -f unix/Makefile generic
sudo make prefix=/usr MANDIR=/usr/share/man/man1 \
 -f unix/Makefile install

cd ..
rm -rf unzip60
```

## sgml

```sh
curl https://sourceware.org/ftp/docbook-tools/new-trials/SOURCES/sgml-common-0.6.3.tgz -o sgml-common-0.6.3.tgz
curl http://www.linuxfromscratch.org/patches/blfs/10.1/sgml-common-0.6.3-manpage-1.patch -o sgml-common-0.6.3-manpage-1.patch
tar xzvf sgml-common-0.6.3.tgz
cd sgml-common-0.6.3

patch -Np1 -i ../sgml-common-0.6.3-manpage-1.patch &&
autoreconf -f -i
./configure --prefix=/usr --sysconfdir=/etc
make
sudo make docdir=/usr/share/doc install &&
sudo install-catalog --add /etc/sgml/sgml-ent.cat \
    /usr/share/sgml/sgml-iso-entities-8879.1986/catalog &&
sudo install-catalog --add /etc/sgml/sgml-docbook.cat \
    /etc/sgml/sgml-ent.cat

cd ..
rm -rf sgml-common-0.6.3
```

## docbook-xml

```sh
curl http://www.docbook.org/xml/4.5/docbook-xml-4.5.zip -o docbook-xml-4.5.zip
unzip docbook-xml-4.5.zip -d docbook-xml-4.5
cd docbook-xml-4.5
```

Build the DTD catalog:

```sh
sudo install -v -d -m755 /usr/share/xml/docbook/xml-dtd-4.5 &&
sudo install -v -d -m755 /etc/xml &&
sudo chown -R root:root . &&
sudo cp -v -af docbook.cat *.dtd ent/ *.mod \
    /usr/share/xml/docbook/xml-dtd-4.5
```

Build the docbook catalog:

```sh
if [ ! -e /etc/xml/docbook ]; then
    sudo xmlcatalog --noout --create /etc/xml/docbook
fi &&
sudo xmlcatalog --noout --add "public" \
    "-//OASIS//DTD DocBook XML V4.5//EN" \
    "http://www.oasis-open.org/docbook/xml/4.5/docbookx.dtd" \
    /etc/xml/docbook &&
sudo xmlcatalog --noout --add "public" \
    "-//OASIS//DTD DocBook XML CALS Table Model V4.5//EN" \
    "file:///usr/share/xml/docbook/xml-dtd-4.5/calstblx.dtd" \
    /etc/xml/docbook &&
sudo xmlcatalog --noout --add "public" \
    "-//OASIS//DTD XML Exchange Table Model 19990315//EN" \
    "file:///usr/share/xml/docbook/xml-dtd-4.5/soextblx.dtd" \
    /etc/xml/docbook &&
sudo xmlcatalog --noout --add "public" \
    "-//OASIS//ELEMENTS DocBook XML Information Pool V4.5//EN" \
    "file:///usr/share/xml/docbook/xml-dtd-4.5/dbpoolx.mod" \
    /etc/xml/docbook &&
sudo xmlcatalog --noout --add "public" \
    "-//OASIS//ELEMENTS DocBook XML Document Hierarchy V4.5//EN" \
    "file:///usr/share/xml/docbook/xml-dtd-4.5/dbhierx.mod" \
    /etc/xml/docbook &&
sudo xmlcatalog --noout --add "public" \
    "-//OASIS//ELEMENTS DocBook XML HTML Tables V4.5//EN" \
    "file:///usr/share/xml/docbook/xml-dtd-4.5/htmltblx.mod" \
    /etc/xml/docbook &&
sudo xmlcatalog --noout --add "public" \
    "-//OASIS//ENTITIES DocBook XML Notations V4.5//EN" \
    "file:///usr/share/xml/docbook/xml-dtd-4.5/dbnotnx.mod" \
    /etc/xml/docbook &&
sudo xmlcatalog --noout --add "public" \
    "-//OASIS//ENTITIES DocBook XML Character Entities V4.5//EN" \
    "file:///usr/share/xml/docbook/xml-dtd-4.5/dbcentx.mod" \
    /etc/xml/docbook &&
sudo xmlcatalog --noout --add "public" \
    "-//OASIS//ENTITIES DocBook XML Additional General Entities V4.5//EN" \
    "file:///usr/share/xml/docbook/xml-dtd-4.5/dbgenent.mod" \
    /etc/xml/docbook &&
sudo xmlcatalog --noout --add "rewriteSystem" \
    "http://www.oasis-open.org/docbook/xml/4.5" \
    "file:///usr/share/xml/docbook/xml-dtd-4.5" \
    /etc/xml/docbook &&
sudo xmlcatalog --noout --add "rewriteURI" \
    "http://www.oasis-open.org/docbook/xml/4.5" \
    "file:///usr/share/xml/docbook/xml-dtd-4.5" \
    /etc/xml/docbook
```

Build the xml catalog:

```sh
if [ ! -e /etc/xml/catalog ]; then
    sudo xmlcatalog --noout --create /etc/xml/catalog
fi &&
sudo xmlcatalog --noout --add "delegatePublic" \
    "-//OASIS//ENTITIES DocBook XML" \
    "file:///etc/xml/docbook" \
    /etc/xml/catalog &&
sudo xmlcatalog --noout --add "delegatePublic" \
    "-//OASIS//DTD DocBook XML" \
    "file:///etc/xml/docbook" \
    /etc/xml/catalog &&
sudo xmlcatalog --noout --add "delegateSystem" \
    "http://www.oasis-open.org/docbook/" \
    "file:///etc/xml/docbook" \
    /etc/xml/catalog &&
sudo xmlcatalog --noout --add "delegateURI" \
    "http://www.oasis-open.org/docbook/" \
    "file:///etc/xml/docbook" \
    /etc/xml/catalog
```

Configure `docbook` to utilize DTD 4.5 when any version 4.x is called for:

```sh
for DTDVERSION in 4.1.2 4.2 4.3 4.4
do
  sudo xmlcatalog --noout --add "public" \
    "-//OASIS//DTD DocBook XML V$DTDVERSION//EN" \
    "http://www.oasis-open.org/docbook/xml/$DTDVERSION/docbookx.dtd" \
    /etc/xml/docbook
  sudo xmlcatalog --noout --add "rewriteSystem" \
    "http://www.oasis-open.org/docbook/xml/$DTDVERSION" \
    "file:///usr/share/xml/docbook/xml-dtd-4.5" \
    /etc/xml/docbook
  sudo xmlcatalog --noout --add "rewriteURI" \
    "http://www.oasis-open.org/docbook/xml/$DTDVERSION" \
    "file:///usr/share/xml/docbook/xml-dtd-4.5" \
    /etc/xml/docbook
  sudo xmlcatalog --noout --add "delegateSystem" \
    "http://www.oasis-open.org/docbook/xml/$DTDVERSION/" \
    "file:///etc/xml/docbook" \
    /etc/xml/catalog
  sudo xmlcatalog --noout --add "delegateURI" \
    "http://www.oasis-open.org/docbook/xml/$DTDVERSION/" \
    "file:///etc/xml/docbook" \
    /etc/xml/catalog
done
```

Clean up:

```sh
cd ..
rm -rf docbook-xml-4.5
```

## docbook-xslt

```sh
curl https://github.com/docbook/xslt10-stylesheets/releases/download/release/1.79.2/docbook-xsl-nons-1.79.2.tar.bz2 -o docbook-xsl-nons-1.79.2.tar.bz2
curl http://www.linuxfromscratch.org/patches/blfs/10.1/docbook-xsl-nons-1.79.2-stack_fix-1.patch -o docbook-xsl-nons-1.79.2-stack_fix-1.patch
tar xvf docbook-xsl-nons-1.79.2.tar.bz2
cd docbook-xsl-nons-1.79.2

patch -Np1 -i ../docbook-xsl-nons-1.79.2-stack_fix-1.patch
sudo install -v -m755 -d /usr/share/xml/docbook/xsl-stylesheets-nons-1.79.2 &&
sudo cp -v -R VERSION assembly common eclipse epub epub3 extensions fo        \
         highlighting html htmlhelp images javahelp lib manpages params  \
         profiling roundtrip slides template tests tools webhelp website \
         xhtml xhtml-1_1 xhtml5                                          \
    /usr/share/xml/docbook/xsl-stylesheets-nons-1.79.2 &&
sudo ln -s VERSION /usr/share/xml/docbook/xsl-stylesheets-nons-1.79.2/VERSION.xsl &&
sudo install -v -m644 -D README \
                    /usr/share/doc/docbook-xsl-nons-1.79.2/README.txt &&
sudo install -v -m644    RELEASE-NOTES* NEWS* \
                    /usr/share/doc/docbook-xsl-nons-1.79.2
```

Configure the catalog:

```sh
if [ ! -d /etc/xml ]; then sudo install -v -m755 -d /etc/xml; fi &&
if [ ! -f /etc/xml/catalog ]; then
    sudo xmlcatalog --noout --create /etc/xml/catalog
fi &&
sudo xmlcatalog --noout --add "rewriteSystem" \
           "https://cdn.docbook.org/release/xsl-nons/1.79.2" \
           "/usr/share/xml/docbook/xsl-stylesheets-nons-1.79.2" \
    /etc/xml/catalog &&
sudo xmlcatalog --noout --add "rewriteURI" \
           "https://cdn.docbook.org/release/xsl-nons/1.79.2" \
           "/usr/share/xml/docbook/xsl-stylesheets-nons-1.79.2" \
    /etc/xml/catalog &&
sudo xmlcatalog --noout --add "rewriteSystem" \
           "https://cdn.docbook.org/release/xsl-nons/current" \
           "/usr/share/xml/docbook/xsl-stylesheets-nons-1.79.2" \
    /etc/xml/catalog &&
sudo xmlcatalog --noout --add "rewriteURI" \
           "https://cdn.docbook.org/release/xsl-nons/current" \
           "/usr/share/xml/docbook/xsl-stylesheets-nons-1.79.2" \
    /etc/xml/catalog &&
sudo xmlcatalog --noout --add "rewriteSystem" \
           "http://docbook.sourceforge.net/release/xsl/current" \
           "/usr/share/xml/docbook/xsl-stylesheets-nons-1.79.2" \
    /etc/xml/catalog &&
sudo xmlcatalog --noout --add "rewriteURI" \
           "http://docbook.sourceforge.net/release/xsl/current" \
           "/usr/share/xml/docbook/xsl-stylesheets-nons-1.79.2" \
    /etc/xml/catalog
```

## libxslt

```sh
curl http://xmlsoft.org/sources/libxslt-1.1.34.tar.gz -o libxslt-1.1.34.tar.gz
tar xzvf libxslt-1.1.34.tar.gz
cd libxslt-1.1.34

sed -i s/3000/5000/ libxslt/transform.c doc/xsltproc.{1,xml} &&
./configure --prefix=/usr --disable-static --without-python
make
sed -e 's@http://cdn.docbook.org/release/xsl@https://cdn.docbook.org/release/xsl-nons@' \
    -e 's@\$Date\$@31 October 2019@' -i doc/xsltproc.xml &&
xsltproc/xsltproc --nonet doc/xsltproc.xml -o doc/xsltproc.1
sudo make install

cd ..
rm -rf libxslt-1.1.34
```

## GLib

```sh
curl https://download.gnome.org/sources/glib/2.66/glib-2.66.7.tar.xz -o glib-2.66.7.tar.xz
curl http://www.linuxfromscratch.org/patches/blfs/10.1/glib-2.66.7-skip_warnings-1.patch -o glib-2.66.7-skip_warnings-1.patch
tar xvf glib-2.66.7.tar.xz
cd glib-2.66.7

patch -Np1 -i ../glib-2.66.7-skip_warnings-1.patch
mkdir build &&
cd    build &&
meson --prefix=/usr      \
      -Dman=true         \
      -Dselinux=disabled \
      ..
ninja
sudo ninja install &&
sudo mkdir -p /usr/share/doc/glib-2.66.7 &&
sudo cp -r ../docs/reference/{NEWS,gio,glib,gobject} /usr/share/doc/glib-2.66.7

cd ..
rm -rf glib-2.66.7
```

## itstool

```sh
curl http://files.itstool.org/itstool/itstool-2.0.6.tar.bz2 -o itstool-2.0.6.tar.bz2
tar xvf itstool-2.0.6.tar.bz2
cd itstool-2.0.6

PYTHON=/usr/bin/python3 ./configure --prefix=/usr
make
sudo make install

cd ..
rm -rf itstool-2.0.6
```

## xmlto

```sh
curl https://releases.pagure.org/xmlto/xmlto-0.0.28.tar.bz2 -o xmlto-0.0.28.tar.bz2
tar xvf xmlto-0.0.28.tar.bz2
cd xmlto-0.0.28

LINKS="/usr/bin/links" ./configure --prefix=/usr
make
sudo make install

cd ..
rm -rf xmlto-0.0.28
```

## shared-mime-info

```sh
curl https://gitlab.freedesktop.org/xdg/shared-mime-info/uploads/0ee50652091363ab0d17e335e5e74fbe/shared-mime-info-2.1.tar.xz -o shared-mime-info-2.1.tar.xz
tar xvf shared-mime-info-2.1.tar.xz
cd shared-mime-info-2.1

mkdir build &&
cd    build &&
meson --prefix=/usr -Dupdate-mimedb=true ..
ninja

sudo ninja install

cd ../..
rm -rf shared-mime-info-2.1
```
