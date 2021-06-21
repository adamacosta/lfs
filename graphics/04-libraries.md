# Graphics libraries

Create a separate directory for GNOME:

```sh
mkdir -pv /sources/gnome
```

## libepoxy

```sh
wget https://github.com/anholt/libepoxy/releases/download/1.5.8/libepoxy-1.5.8.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/libepoxy-1.5.8.tar.xz &&
cd       libepoxy-1.5.8                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libepoxy-1.5.8
```

## ATK

```sh
wget https://download.gnome.org/sources/atk/2.36/atk-2.36.0.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/atk-2.36.0.tar.xz &&
cd       atk-2.36.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf atk-2.36.0
```

## at-spi2-core

```sh
wget https://download.gnome.org/sources/at-spi2-core/2.40/at-spi2-core-2.40.2.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/at-spi2-core-2.40.2.tar.xz &&
cd       at-spi2-core-2.40.2                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf at-spi2-core-2.40.2
```

## at-spi2-atk

```sh
wget https://download.gnome.org/sources/at-spi2-atk/2.38/at-spi2-atk-2.38.0.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/at-spi2-atk-2.38.0.tar.xz &&
cd       at-spi2-atk-2.38.0                      &&

mkdir build &&
cd build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf at-spi2-atk-2.38.0
```

## gtk-doc

```sh
wget https://download.gnome.org/sources/gtk-doc/1.33/gtk-doc-1.33.2.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/gtk-doc-1.33.2.tar.xz &&
cd       gtk-doc-1.33.2                      &&

autoreconf -fiv           &&
./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf gtk-doc-1.33.2
```

## JSON-Glib

```sh
wget https://download.gnome.org/sources/json-glib/1.6/json-glib-1.6.2.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/json-glib-1.6.2.tar.xz &&
cd       json-glib-1.6.2                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
sudo rm -rf json-glib-1.6.2
```

## ISO Codes

```sh
wget http://anduin.linuxfromscratch.org/BLFS/iso-codes/iso-codes-4.6.0.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/iso-codes-4.6.0.tar.xz &&
cd       iso-codes-4.6.0                      &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf iso-codes-4.6.0
```

### GTK+-2

```sh
wget https://download.gnome.org/sources/gtk+/2.24/gtk+-2.24.33.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/gtk+-2.24.33.tar.xz &&
cd       gtk+-2.24.33                      &&

sed -e 's#l \(gtk-.*\).sgml#& -P \1#' \
    -i docs/{faq,tutorial}/Makefile.in      &&

./configure --prefix=/usr \
            --sysconfdir=/etc &&

make              &&
sudo make install &&

cd .. &&
rm -rf gtk+-2.24.33
```

## GTK+-3

```sh
wget https://download.gnome.org/sources/gtk+/3.24/gtk+-3.24.29.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/gtk+-3.24.29.tar.xz &&
cd       gtk+-3.24.29                      &&

./configure --prefix=/usr              \
            --sysconfdir=/etc          \
            --enable-broadway-backend  \
            --enable-x11-backend       \
            --enable-wayland-backend   &&

make              &&
sudo make install &&

cd .. &&
rm -rf gtk+-3.24.29
```

## GTK4

```sh
wget https://download.gnome.org/sources/gtk/4.2/gtk-4.2.1.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/gtk-4.2.1.tar.xz &&
cd       gtk-4.2.1                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Dbroadway_backend=true .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gtk-4.2.1
```

## tk

After installing this, you can rebuild `Python` to include the `TKinter` graphics toolkit.

```sh
wget https://downloads.sourceforge.net/tcl/tk8.6.11.1-src.tar.gz -P /sources &&

tar xzvf /sources/tk8.6.11-1-src.tar.gz &&
cd        tk8.6.11                      &&

cd unix &&
./configure --prefix=/usr \
            --mandir=/usr/share/man \
            $([ $(uname -m) = x86_64 ] && echo --enable-64bit) &&

make &&

sed -e "s@^\(TK_SRC_DIR='\).*@\1/usr/include'@" \
    -e "/TK_B/s@='\(-L\)\?.*unix@='\1/usr/lib@" \
    -i tkConfig.sh

sudo make install                      &&
sudo make install-private-headers      &&
sudo ln -v -sf wish8.6 /usr/bin/wish   &&
sudo chmod -v 755 /usr/lib/libtk8.6.so &&

cd ../.. &&
rm -rf tk8.6.11
```

## gcr

```sh
wget https://download.gnome.org/sources/gcr/3.40/gcr-3.40.0.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/gcr-3.40.0.tar.xz &&
cd       gcr-3.40.0                      &&

sed -i 's:"/desktop:"/org:' schema/*.xml &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Dgtk_doc=false .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gcr-3.40.0
```

## libsecret

```sh
wget https://download.gnome.org/sources/libsecret/0.20/libsecret-0.20.4.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/libsecret-0.20.4.tar.xz &&
cd       libsecret-0.20.4                      &&

mkdir bld &&
cd bld &&

meson --prefix=/usr \
      --buildtype=release \
      -Dgtk_doc=false .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libsecret-0.20.4
```

## glib-networking

```sh
wget https://download.gnome.org/sources/glib-networking/2.68/glib-networking-2.68.1.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/glib-networking-2.68.1.tar.xz &&
cd       glib-networking-2.68.1                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr          \
      --buildtype=release    \
      -Dlibproxy=disabled .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf glib-networking-2.68.1
```

## rest

```sh
wget https://download.gnome.org/sources/rest/0.8/rest-0.8.1.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/rest-0.8.1.tar.xz &&
cd       rest-0.8.1                      &&

./configure --prefix=/usr \
    --with-ca-certificates=/etc/pki/tls/certs/ca-bundle.crt &&

make              &&
sudo make install &&

cd .. &&
rm -rf rest-0.8.1
```

## VTE

```sh
wget https://gitlab.gnome.org/GNOME/vte/-/archive/0.64.2/vte-0.64.2.tar.gz -P /sources/gnome &&

tar xzvf /sources/gnome/vte-0.64.2.tar.gz &&
cd        vte-0.64.2                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release \
      -Dfribidi=false .. &&

ninja              &&
sudo ninja install &&

sudo rm -v /etc/profile.d/vte.* &&

cd ../.. &&
rm -rf vte-0.64.2
```

## dbus-glib

```sh
wget https://dbus.freedesktop.org/releases/dbus-glib/dbus-glib-0.112.tar.gz -P /sources/gnome &&

tar xzvf /sources/gnome/dbus-glib-0.112.tar.gz &&
cd        dbus-glib-0.112                      &&

./configure --prefix=/usr     \
            --sysconfdir=/etc \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf dbus-glib-0.112
```

## libwpe

```sh
wget http://wpewebkit.org/releases/libwpe-1.10.0.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/libwpe-1.10.0.tar.xz &&
cd       libwpe-1.10.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libwpe-1.10.0
```

## wpebackend-fdo

```sh
wget https://wpewebkit.org/releases/wpebackend-fdo-1.10.0.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/wpebackend-fdo-1.10.0.tar.xz &&
cd       wpebackend-fdo-1.10.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release ..

ninja              &&
sudo ninja install &&


cd ../.. &&
rm -rf wpebackend-fdo-1.10.0
```

## libnotify

```sh
wget https://download.gnome.org/sources/libnotify/0.7/libnotify-0.7.9.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/libnotify-0.7.9.tar.xz &&
cd       libnotify-0.7.9                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Dgtk_doc=false     \
      -Dman=false .. &&

ninja              &&
sudo ninja install &&

sudo mv /usr/share/doc/libnotify /usr/share/doc/libnotify-0.7.9 &&

cd ../.. &&
rm -rf libnotify-0.7.9
```

## WebKitGTK

```sh
wget https://webkitgtk.org/releases/webkitgtk-2.32.1.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/webkitgtk-2.32.1.tar.xz &&
cd       webkitgtk-2.32.1                      &&

mkdir -vp build &&
cd        build &&

cmake -DCMAKE_BUILD_TYPE=Release  \
      -DCMAKE_INSTALL_PREFIX=/usr \
      -DCMAKE_SKIP_RPATH=ON       \
      -DPORT=GTK                  \
      -DLIB_INSTALL_DIR=/usr/lib  \
      -DUSE_LIBHYPHEN=OFF         \
      -DENABLE_GAMEPAD=OFF        \
      -DENABLE_MINIBROWSER=ON     \
      -DUSE_WOFF2=OFF             \
      -DUSE_WPE_RENDERER=ON       \
      -DENABLE_BUBBLEWRAP_SANDBOX=OFF \
      -Wno-dev -G Ninja ..        &&

ninja              &&
sudo ninja install &&

sudo install -vdm755 /usr/share/gtk-doc/html/webkit{2,dom}gtk-4.0 &&
sudo install -vm644  ../Documentation/webkit2gtk-4.0/html/*   \
                     /usr/share/gtk-doc/html/webkit2gtk-4.0       &&
sudo install -vm644  ../Documentation/webkitdomgtk-4.0/html/* \
                     /usr/share/gtk-doc/html/webkitdomgtk-4.0     &&

cd ../.. &&
rm -rf webkitgtk-2.32.1
```

## babl

Pixel format conversion library.

```sh
wget https://download.gimp.org/pub/babl/0.1/babl-0.1.86.tar.xz -P /sources &&

tar xvf /sources/babl-0.1.86.tar.xz &&
cd       babl-0.1.86                &&

mkdir bld &&
cd    bld &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf babl-0.1.86
```

## GEGL

Generic graphics library.

```sh
wget https://download.gimp.org/pub/gegl/0.4/gegl-0.4.30.tar.xz -P /sources &&

tar xvf /sources/gegl-0.4.30.tar.xz &&
cd       gegl-0.4.30                &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gegl-0.4.30
```

## libdazzle

```sh
wget https://download.gnome.org/sources/libdazzle/3.40/libdazzle-3.40.0.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/libdazzle-3.40.0.tar.xz &&
cd       libdazzle-3.40.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libdazzle-3.40.0
```

## gfbgraph

Interface to Facebook graph API.

```sh
wget https://download.gnome.org/sources/gfbgraph/0.2/gfbgraph-0.2.4.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/gfbgraph-0.2.4.tar.xz &&
cd       gfbgraph-0.2.4                      &&

./autogen.sh --prefix=/usr --disable-static &&

make &&
sudo make libgfbgraphdocdir=/usr/share/doc/gfbgraph-0.2.4 install &&

cd .. &&
rm -rf gfbgraph-0.2.4
```

## libadwaita

```sh
wget https://github.com/GNOME/libadwaita/archive/refs/tags/1.0.0-alpha.1.tar.gz -P /sources/gnome &&

tar xzvf /sources/gnome/libadwaita-1.0.0-alpha.1.tar.gz &&
cd        libadwaita-1.0.0-alpha.1                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libadwaita-1.0.0-alpha.1
```

## libmediaart

```sh
wget https://download.gnome.org/sources/libmediaart/1.9/libmediaart-1.9.5.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/libmediaart-1.9.5.tar.xz &&
cd       libmediaart-1.9.5                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libmediaart-1.9.5
```

## grilo-plugins

```sh
wget https://download.gnome.org/sources/grilo-plugins/0.3/grilo-plugins-0.3.9.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/grilo-plugins-0.3.9.tar.xz &&
cd       grilo-plugins-0.3.9                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf grilo-plugins-0.3.9
```

## libgepub

E-Book reading library.

```sh
wget https://download.gnome.org/sources/libgepub/0.6/libgepub-0.6.0.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/libgepub-0.6.0.tar.xz &&
cd       libgepub-0.6.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libgepub-0.6.0
```

## libcairomm

`C++` interface to `Cairo`.

```sh
wget https://www.cairographics.org/releases/cairomm-1.14.0.tar.xz -P /sources &&

tar xvf /sources/cairomm-1.14.0.tar.xz &&
cd       cairomm-1.14.0                &&

mkdir bld &&
cd    bld &&

meson --prefix=/usr       \
      -Dbuild-tests=true  \
      -Dboost-shared=true \
      .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf cairomm-1.14.0
```

## GLibmm

`C++` bindings for `GLib`.

```sh
wget https://download.gnome.org/sources/glibmm/2.66/glibmm-2.66.1.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/glibmm-2.66.1.tar.xz &&
cd       glibmm-2.66.1                      &&

mkdir bld &&
cd    bld &&

meson --prefix=/usr .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf glibmm-2.66.1
```

## Pangomm

`C++` interface to `Pango`.

```sh
wget https://download.gnome.org/sources/pangomm/2.46/pangomm-2.46.1.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/pangomm-2.46.1.tar.xz &&
cd       pangomm-2.46.1                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf pangomm-2.46.1
```

## Atkmm

Accessibility toolkit.

```sh
wget https://download.gnome.org/sources/atkmm/2.28/atkmm-2.28.2.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/atkmm-2.28.2.tar.xz &&
cd       atkmm-2.28.2                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf atkmm-2.28.2
```

## Gtkmm

`C++` interface to `GTK`.

```sh
wget https://download.gnome.org/sources/gtkmm/3.24/gtkmm-3.24.5.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/gtkmm-3.24.5.tar.xz &&
cd       gtkmm-3.24.5                      &&

mkdir gtkmm3-build &&
cd    gtkmm3-build &&

meson --prefix=/usr .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gtkmm-3.24.5
```

## amtk

```sh
wget https://download.gnome.org/sources/amtk/5.3/amtk-5.3.1.tar.xz -P /sources/gnome &&

tar xf /sources/gnome/amtk-5.3.1.tar.xz &&
cd      amtk-5.3.1                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Db_lto=true ..

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf amtk-5.3.1
```

## tepl

```sh
wget https://download.gnome.org/sources/tepl/6.00/tepl-6.00.0.tar.xz -P /sources/gnome &&

tar xf /sources/gnome/tepl-6.00.0.tar.xz &&
cd      tepl-6.00.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Db_lto=true ..

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf tepl-6.00.0
```

