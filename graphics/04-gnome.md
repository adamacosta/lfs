# GNOME desktop

Create a separate directory for GNOME:

```sh
mkdir -pv /sources/gnome
```

## libepoxy

```sh
curl https://github.com/anholt/libepoxy/releases/download/1.5.8/libepoxy-1.5.8.tar.xz -o /sources/gnome/libepoxy-1.5.8.tar.xz &&

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
curl https://download.gnome.org/sources/atk/2.36/atk-2.36.0.tar.xz -o /sources/gnome/atk-2.36.0.tar.xz &&

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
curl https://download.gnome.org/sources/at-spi2-core/2.40/at-spi2-core-2.40.2.tar.xz -o /sources/gnome/at-spi2-core-2.40.2.tar.xz &&

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
curl https://download.gnome.org/sources/at-spi2-atk/2.38/at-spi2-atk-2.38.0.tar.xz -o /sources/gnome/at-spi2-atk-2.38.0.tar.xz &&

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
curl https://download.gnome.org/sources/gtk-doc/1.33/gtk-doc-1.33.2.tar.xz -o /sources/gnome/gtk-doc-1.33.2.tar.xz &&

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
curl https://download.gnome.org/sources/json-glib/1.6/json-glib-1.6.2.tar.xz -o /sources/gnome/json-glib-1.6.2.tar.xz &&

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
curl http://anduin.linuxfromscratch.org/BLFS/iso-codes/iso-codes-4.6.0.tar.xz -o /sources/gnome/iso-codes-4.6.0.tar.xz &&

tar xvf /sources/gnome/iso-codes-4.6.0.tar.xz &&
cd       iso-codes-4.6.0                      &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf iso-codes-4.6.0
```

## GTK+-3

```sh
curl https://download.gnome.org/sources/gtk+/3.24/gtk+-3.24.29.tar.xz -o /sources/gnome/gtk+-3.24.29.tar.xz &&

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
curl https://download.gnome.org/sources/gtk/4.2/gtk-4.2.1.tar.xz -o /sources/gnome/gtk-4.2.1.tar.xz &&

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

At this point, you can rebuild `Python` to include the `TKinter` graphics toolkit if you build `tk`.

## tk

```sh
curl https://downloads.sourceforge.net/tcl/tk8.6.11.1-src.tar.gz -o /sources/tk8.6.11-1-src.tar.gz &&

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
curl https://download.gnome.org/sources/gcr/3.40/gcr-3.40.0.tar.xz -o /sources/gnome/gcr-3.40.0.tar.xz &&

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

## gsettings-desktop-schemas

```sh
curl https://download.gnome.org/sources/gsettings-desktop-schemas/40/gsettings-desktop-schemas-40.0.tar.xz -o /sources/gnome/gsettings-desktop-schemas-40.0.tar.xz &&

tar xvf /sources/gnome/gsettings-desktop-schemas-40.0.tar.xz &&
cd       gsettings-desktop-schemas-40.0                      &&

sed -i -r 's:"(/system):"/org/gnome\1:g' schemas/*.in &&

mkdir build &&
cd    build &&

meson --prefix=/usr .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gsettings-desktop-schemas-40.0
```

## libsecret

```sh
curl https://download.gnome.org/sources/libsecret/0.20/libsecret-0.20.4.tar.xz -o /sources/gnome/libsecret-0.20.4.tar.xz &&

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
curl https://download.gnome.org/sources/glib-networking/2.68/glib-networking-2.68.1.tar.xz -o /sources/gnome/glib-networking-2.68.1.tar.xz &&

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
curl https://download.gnome.org/sources/rest/0.8/rest-0.8.1.tar.xz -o /sources/gnome/rest-0.8.1.tar.xz &&

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
curl https://gitlab.gnome.org/GNOME/vte/-/archive/0.64.2/vte-0.64.2.tar.gz -o /sources/gnome/vte-0.64.2.tar.gz &&

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

## yelp-xsl

```sh
curl https://download.gnome.org/sources/yelp-xsl/40/yelp-xsl-40.2.tar.xz -o /sources/gnome/yelp-xsl-40.2.tar.xz &&

tar xvf /sources/gnome/yelp-xsl-40.2.tar.xz &&
cd       yelp-xsl-40.2                      &&

./configure --prefix=/usr &&

sudo make install &&

cd .. &&
rm -rf yelp-xsl-40.2
```

## dbus-glib

```sh
curl https://dbus.freedesktop.org/releases/dbus-glib/dbus-glib-0.112.tar.gz -o /sources/gnome/dbus-glib-0.112.tar.gz &&

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

## GConf

```sh
curl https://download.gnome.org/sources/GConf/3.2/GConf-3.2.6.tar.xz -o /sources/gnome/GConf-3.2.6.tar.xz &&

tar xvf /sources/gnome/GConf-3.2.6.tar.xz &&
cd       GConf-3.2.6                      &&

./configure --prefix=/usr \
            --sysconfdir=/etc \
            --disable-orbit \
            --disable-static &&

make                                                        &&
sudo make install                                           &&
sudo ln -sfv gconf.xml.defaults /etc/gconf/gconf.xml.system &&

cd .. &&
rm -rf GConf-3.2.6
```

## gnome-autoar

```sh
curl https://download.gnome.org/sources/gnome-autoar/0.3/gnome-autoar-0.3.3.tar.xz -o /sources/gnome/gnome-autoar-0.3.3.tar.xz &&

tar xvf /sources/gnome/gnome-autoar-0.3.3.tar.xz &&
cd       gnome-autoar-0.3.3                      &&

./configure --prefix=/usr    \
            --disable-debug  \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf gnome-autoar-0.3.3
```

## gnome-desktop

```sh
curl https://download.gnome.org/sources/gnome-desktop/40/gnome-desktop-40.2.tar.xz -o /sources/gnome/gnome-desktop-40.2.tar.xz &&

tar xvf /sources/gnome/gnome-desktop-40.2.tar.xz &&
cd       gnome-desktop-40.2                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr                 \
      --buildtype=release           \
      -Dgnome_distributor="BLFS" .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
sudo rm -rf gnome-desktop-40.2
```

## gnome-menus

```sh
curl https://download.gnome.org/sources/gnome-menus/3.36/gnome-menus-3.36.0.tar.xz -o /sources/gnome/gnome-menus-3.36.0.tar.xz &&

tar xvf /sources/gnome/gnome-menus-3.36.0.tar.xz &&
cd       gnome-menus-3.36.0                      &&

./configure --prefix=/usr \
            --sysconfdir=/etc \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf gnome-menus-3.36.0
```

## gnome-video-effects

```sh
curl https://download.gnome.org/sources/gnome-video-effects/0.5/gnome-video-effects-0.5.0.tar.xz -o /sources/gnome/gnome-video-effects-0.5.0.tar.xz &&

tar xvf /sources/gnome/gnome-video-effects-0.5.0.tar.xz &&
cd       gnome-video-effects-0.5.0 &&

mkdir build &&
cd    build &&

meson --prefix=/usr .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gnome-video-effects-0.5.0
```

## libwpe

```sh
curl http://wpewebkit.org/releases/libwpe-1.10.0.tar.xz -o /sources/gnome/libwpe-1.10.0.tar.xz &&

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
curl https://wpewebkit.org/releases/wpebackend-fdo-1.10.0.tar.xz -o /sources/gnome/wpebackend-fdo-1.10.0.tar.xz &&

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
curl https://download.gnome.org/sources/libnotify/0.7/libnotify-0.7.9.tar.xz -o /sources/gnome/libnotify-0.7.9.tar.xz &&

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

cd ../.. &&
rm -rf libnotify-0.7.9
```

## WebKitGTK

```sh
curl https://webkitgtk.org/releases/webkitgtk-2.32.1.tar.xz -o /sources/gnome/webkitgtk-2.32.1.tar.xz &&

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

## Gnome Online Accounts

```sh
curl https://download.gnome.org/sources/gnome-online-accounts/3.40/gnome-online-accounts-3.40.0.tar.xz -o /sources/gnome/gnome-online-accounts-3.40.0.tar.xz &&

tar xvf /sources/gnome/gnome-online-accounts-3.40.0.tar.xz &&
cd       gnome-online-accounts-3.40.0                      &&

./configure --prefix=/usr \
            --disable-static \
            --with-google-client-secret=5ntt6GbbkjnTVXx-MSxbmx5e \
            --with-google-client-id=595013732528-llk8trb03f0ldpqq6nprjp1s79596646.apps.googleusercontent.com &&

make              &&
sudo make install &&

cd .. &&
rm -rf gnome-online-accounts-3.40.0
```

## totem-pl-parser

```sh
curl https://download.gnome.org/sources/totem-pl-parser/3.26/totem-pl-parser-3.26.5.tar.xz -o /sources/gnome/totem-pl-parser-3.26.5.tar.xz &&

tar xvf /sources/gnome/totem-pl-parser-3.26.5.tar.xz &&
cd       totem-pl-parser-3.26.5                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf totem-pl-parser-3.26.5
```

## grilo

```sh
curl https://download.gnome.org/sources/grilo/0.3/grilo-0.3.13.tar.xz -o /sources/gnome/grilo-0.3.13.tar.xz &&

tar xvf /sources/gnome/grilo-0.3.13.tar.xz &&
cd       grilo-0.3.13                      &&

mkdir build &&
cd    build    &&

meson --prefix=/usr       \
      --buildtype=release \
      -Denable-gtk-doc=false .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf grilo-0.3.13
```

## cogl

```sh
curl https://download.gnome.org/sources/cogl/1.22/cogl-1.22.8.tar.xz -o /sources/gnome/cogl-1.22.8.tar.xz &&

tar xvf /sources/gnome/cogl-1.22.8.tar.xz &&
cd       cogl-1.22.8                      &&

./configure --prefix=/usr  \
            --enable-gles1 \
            --enable-gles2 \
            --enable-{kms,wayland,xlib}-egl-platform \
            --enable-wayland-egl-server &&

make              &&
sudo make install &&

cd .. &&
rm -rf cogl-1.22.8
```

## clutter

```sh
curl https://download.gnome.org/sources/clutter/1.26/clutter-1.26.4.tar.xz -o /sources/gnome/clutter-1.26.4.tar.xz &&

tar xvf /sources/gnome/clutter-1.26.4.tar.xz &&
cd       clutter-1.26.4                      &&


```
