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

./configure --prefix=/usr               \
            --sysconfdir=/etc           \
            --enable-egl-backend        \
            --enable-evdev-input        \
            --enable-wayland-backend    \
            --enable-wayland-compositor &&

make              &&
sudo make install &&

cd .. &&
rm -rf clutter-1.26.4
```

## libgudev

```sh
curl https://download.gnome.org/sources/libgudev/236/libgudev-236.tar.xz -o /sources/gnome/libgudev-236.tar.xz &&

tar xvf /sources/gnome/libgudev-236.tar.xz &&
cd      libgudev-236                       &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libgudev-236
```

## clutter-gst

```sh
curl https://download.gnome.org/sources/clutter-gst/3.0/clutter-gst-3.0.27.tar.xz -o /sources/gnome/clutter-gst-3.0.27.tar.xz &&

tar xvf /sources/gnome/clutter-gst-3.0.27.tar.xz &&
cd       clutter-gst-3.0.27                      &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf clutter-gst-3.0.27
```

## clutter-gtk

```sh
curl https://download.gnome.org/sources/clutter-gtk/1.8/clutter-gtk-1.8.4.tar.xz -o /sources/gnome/clutter-gtk-1.8.4.tar.xz &&

tar xvf /sources/gnome/clutter-gtk-1.8.4.tar.xz &&
cd       clutter-gtk-1.8.4                      &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf clutter-gtk-1.8.4
```

## glibusb

```sh
curl https://github.com/hughsie/libgusb/archive/0.3.7/libgusb-0.3.7.tar.gz -o /sources/gnome/libgusb-0.3.7.tar.gz &&

tar xzvf /sources/gnome/libgusb-0.3.7.tar.gz &&
cd        libgusb-0.3.7                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release \
      -Ddocs=false .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libgusb-0.3.7
```

## colord

`colord` requires its own system user:

```sh
sudo groupadd -g 71 colord &&
sudo useradd  -c "Color Daemon Owner" -d /var/lib/colord -u 71 \
              -g colord -s /bin/false colord
```

Then build and install.

```sh
curl https://www.freedesktop.org/software/colord/releases/colord-1.4.5.tar.xz -o /sources/gnome/colord-1.4.5.tar.xz &&

tar xvf /sources/gnome/colord-1.4.5.tar.xz &&
cd       colord-1.4.5                      &&

mv po/fur.po po/ur.po         &&
sed -i 's/fur/ur/' po/LINGUAS &&

mkdir build &&
cd build &&

meson --prefix=/usr            \
      --buildtype=release      \
      -Ddaemon_user=colord     \
      -Dvapi=true              \
      -Dsystemd=true           \
      -Dlibcolordcompat=true   \
      -Dargyllcms_sensor=false \
      -Dbash_completion=false  \
      -Ddocs=false             \
      -Dman=false .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf colord-1.4.5
```

## colord-gtk

GTK+ bindings for colord.

```sh
curl https://www.freedesktop.org/software/colord/releases/colord-gtk-0.2.0.tar.xz -o /sources/gnome/colord-gtk-0.2.0.tar.xz &&

tar xvf /sources/gnome/colord-gtk-0.2.0.tar.xz &&
cd       colord-gtk-0.2.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Dvapi=true         \
      -Ddocs=false        \
      -Dman=false .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf colord-gtk-0.2.0
```

## libchamplain

Widget for displaying maps.

```sh
curl https://download.gnome.org/sources/libchamplain/0.12/libchamplain-0.12.20.tar.xz -o /sources/gnome/libchamplain-0.12.20.tar.xz &&

tar xvf /sources/gnome/libchamplain-0.12.20.tar.xz &&
cd       libchamplain-0.12.20                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libchamplain-0.12.20
```

## libgdata

```sh
curl https://download.gnome.org/sources/libgdata/0.18/libgdata-0.18.1.tar.xz -o /sources/gnome/libgdata-0.18.1.tar.xz &&

tar xvf /sources/gnome/libgdata-0.18.1.tar.xz &&
cd       libgdata-0.18.1                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr                 \
      --buildtype=release           \
      -Dgtk_doc=false               \
      -Dalways_build_tests=false .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libgdata-0.18.1
```

## libgee

GObject based interfaces for common data structures.

```sh
curl https://download.gnome.org/sources/libgee/0.20/libgee-0.20.4.tar.xz -o /sources/gnome/libgee-0.20.4.tar.xz &&

tar xvf /sources/gnome/libgee-0.20.4.tar.xz &&
cd       libgee-0.20.4                      &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf libgee-0.20.4
```

## libgtop

`GNOME`'s graphical version of `top`.

```sh
curl https://download.gnome.org/sources/libgtop/2.40/libgtop-2.40.0.tar.xz -o /sources/gnome/libgtop-2.40.0.tar.xz &&

tar xvf /sources/gnome/libgtop-2.40.0.tar.xz &&
cd       libgtop-2.40.0                      &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libgtop-2.40.0
```

## geocode-glib

Library for Place Finder API to convert an address to lat/lon coordinates.

```sh
curl https://download.gnome.org/sources/geocode-glib/3.26/geocode-glib-3.26.2.tar.xz -o /sources/gnome/geocode-glib-3.26.2.tar.xz &&

tar xvf /sources/gnome/geocode-glib-3.26.2.tar.xz &&
cd       geocode-glib-3.26.2                      &&

mkdir build                                   &&
cd    build                                   &&

meson --prefix /usr       \
      --buildtype=release \
      -Denable-gtk-doc=false .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf geocode-glib-3.26.2
```

## libgweather

Libraries for accessing remote weather service APIs.

```sh
curl https://download.gnome.org/sources/libgweather/40/libgweather-40.0.tar.xz -o /sources/gnome/libgweather-40.0.tar.xz &&

tar xvf /sources/gnome/libgweather-40.0.tar.xz &&
cd       libgweather-40.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libgweather-40.0
```

## Gjs

JavaScript bindings for `GNOME`.

```sh
curl https://download.gnome.org/sources/gjs/1.68/gjs-1.68.1.tar.xz -o /sources/gnome/gjs-1.68.1.tar.xz &&

tar xvf /sources/gnome/gjs-1.68.1.tar.xz &&
cd       gjs-1.68.1                      &&

mkdir gjs-build &&
cd    gjs-build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

sudo ln -sfv gjs-console /usr/bin/gjs &&

cd ../.. &&
rm -rf gjs-1.68.1
```

## libpeas

Plugin engine for `GNOME` so applications can create their own extensions.

```sh
curl https://download.gnome.org/sources/libpeas/1.30/libpeas-1.30.0.tar.xz -o /sources/gnome/libpeas-1.30.0.tar.xz &&

tar xvf /sources/gnome/libpeas-1.30.0.tar.xz &&
cd       libpeas-1.30.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libpeas-1.30.0
```

## libwnck

The Window Navigator Construction Kit.

```sh
curl https://download.gnome.org/sources/libwnck/40/libwnck-40.0.tar.xz -o /sources/gnome/libwnck-40.0.tar.xz &&

tar xvf /sources/gnome/libwnck-40.0.tar.xz &&
cd       libwnck-40.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libwnck-40.0
```

## Evolution Data Server

If they exist, remove older version of `systemd` services which are incompatible:

```sh
sudo rm -fv /usr/lib/systemd/user/evolution-*.service
```

```sh
curl https://download.gnome.org/sources/evolution-data-server/3.40/evolution-data-server-3.40.2.tar.xz -o /sources/gnome/evolution-data-server-3.40.2.tar.xz &&

tar xvf /sources/gnome/evolution-data-server-3.40.2.tar.xz &&
cd       evolution-data-server-3.40.2                      &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr   \
      -DSYSCONF_INSTALL_DIR=/etc    \
      -DENABLE_VALA_BINDINGS=ON     \
      -DENABLE_INSTALLED_TESTS=ON   \
      -DENABLE_GOOGLE=OFF           \
      -DWITH_OPENLDAP=OFF           \
      -DWITH_KRB5=OFF               \
      -DENABLE_INTROSPECTION=ON     \
      -DENABLE_GTK_DOC=OFF          \
      .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf evolution-data-server-3.40.2
```

## UPower

Interface for enumerating power devices and listening to events.

This requires user namespaces to be enabled in the kernel.

```
General Setup --->
    [*] Namespaces support --->     [CONFIG_NAMESPACES]
       [*] User namespace           [CONFIG_USER_NS]
```

```sh
curl https://gitlab.freedesktop.org/upower/upower/uploads/93cfe7c8d66ed486001c4f3f55399b7a/upower-0.99.11.tar.xz -o /sources/gnome/upower-0.99.11.tar.xz &&

tar xvf /sources/gnome/upower-0.99.11.tar.xz &&
cd       upower-0.99.11                      &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --enable-deprecated  \
            --disable-static &&

make              &&
sudo make install &&

sudo systemctl enable upower &&

cd .. &&
rm -rf upower-0.99.11
```

## Tracker

The `GNOME` file indexing and search service.

```sh
curl https://download.gnome.org/sources/tracker/3.1/tracker-3.1.1.tar.xz -o /sources/gnome/tracker-3.1.1.tar.xz &&

tar xvf /sources/gnome/tracker-3.1.1.tar.xz &&
cd       tracker-3.1.1                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Ddocs=false        \
      -Dman=false .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf tracker-3.1.1
```

## gexiv

`GNOME` wrapper for `exiv2` library.

```sh
curl https://download.gnome.org/sources/gexiv2/0.12/gexiv2-0.12.2.tar.xz -o /sources/gnome/gexiv2-0.12.2.tar.xz &&

tar xvf /sources/gnome/gexiv2-0.12.2.tar.xz &&
cd       gexiv2-0.12.2                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gexiv2-0.12.2
```

## libgrss

Libraries for manipulating RSS and ATOM feeds.

```sh
curl https://download.gnome.org/sources/libgrss/0.7/libgrss-0.7.0.tar.xz -o /sources/gnome/libgrss-0.7.0.tar.xz &&
curl https://www.linuxfromscratch.org/patches/blfs/svn/libgrss-0.7.0-bugfixes-1.patch -o /sources/patches/libgrss-0.7.0-bugfixes-1.patch &&

tar xvf /sources/gnome/libgrss-0.7.0.tar.xz &&
cd       libgrss-0.7.0                      &&

patch -Np1 -i /sources/patches/libgrss-0.7.0-bugfixes-1.patch &&

autoreconf -fv               &&
./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libgrss-0.7.0
```

## libgxps

Libraries for manipulating xps documents.

```sh
curl https://download.gnome.org/sources/libgxps/0.3/libgxps-0.3.2.tar.xz -o /sources/gnome/libgxps-0.3.2.tar.xz &&

tar xvf /sources/gnome/libgxps-0.3.2.tar.xz &&
cd       libgxps-0.3.2                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libgxps-0.3.2
```

## libgsf

```sh
curl https://download.gnome.org/sources/libgsf/1.14/libgsf-1.14.47.tar.xz -o /sources/gnome/libgsf-1.14.47.tar.xz &&

tar xvf /sources/gnome/libgsf-1.14.47.tar.xz &&
cd       libgsf-1.14.47                      &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libgsf-1.14.47
```

## DConf

Backend for editing `GNOME` settings.

```sh
curl https://download.gnome.org/sources/dconf/0.40/dconf-0.40.0.tar.xz -o /sources/gnome/dconf-0.40.0.tar.xz &&

tar xvf /sources/gnome/dconf-0.40.0.tar.xz &&
cd       dconf-0.40.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Dbash_completion=false .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf dconf-0.40.0
```

## Tracker Miners

Data extractors for `tracker`.

If files from version 2 exist, they need to be removed:

```sh
sudo rm -v /etc/xdg/autostart/tracker-miner-*
```

Then install the current version.

```sh
curl https://download.gnome.org/sources/tracker-miners/3.1/tracker-miners-3.1.1.tar.xz -o /sources/gnome/tracker-miners-3.1.1.tar.xz &&

tar xvf /sources/gnome/tracker-miners-3.1.1.tar.xz &&
cd       tracker-miners-3.1.1                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Dman=false .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf tracker-miners-3.1.1
```

## GSound

Library for playing system sounds.

```sh
curl https://download.gnome.org/sources/gsound/1.0/gsound-1.0.2.tar.xz -o /sources/gnome/gsound-1.0.2.tar.xz &&

tar xvf /sources/gnome/gsound-1.0.2.tar.xz &&
cd       gsound-1.0.2                      &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf gsound-1.0.2
```

## gnome-backgrounds

Framework and example files for desktop wallpaper.

```sh
curl https://download.gnome.org/sources/gnome-backgrounds/40/gnome-backgrounds-40.1.tar.xz -o /sources/gnome/gnome-backgrounds-40.1.tar.xz &&

tar xvf /sources/gnome/gnome-backgrounds-40.1.tar.xz &&
cd       gnome-backgrounds-40.1                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gnome-backgrounds-40.1
```

## Gvfs

`GNOME`'s userspace virtual filesystem.

```sh
curl https://download.gnome.org/sources/gvfs/1.48/gvfs-1.48.1.tar.xz -o /sources/gnome/gvfs-1.48.1.tar.xz &&

tar xvf /sources/gnome/gvfs-1.48.1.tar.xz &&
cd       gvfs-1.48.1                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Dgphoto2=false     \
      -Dafc=false         \
      -Dbluray=false      \
      -Dnfs=false         \
      -Dmtp=false         \
      -Dsmb=false         \
      -Ddnssd=false       \
      -Dgoa=false         \
      -Dgoogle=false .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gvfs-1.48.1
```

## libhandy

GTK widgets for user interfaces.

```sh
curl https://download.gnome.org/sources/libhandy/1.2/libhandy-1.2.2.tar.xz -o /sources/gnome/libhandy-1.2.2.tar.xz &&

tar xvf /sources/gnome/libhandy-1.2.2.tar.xz &&
cd       libhandy-1.2.2                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libhandy-1.2.2
```

## Desktop Fileutils

Libraries used to manipulate desktop entries.

```sh
curl https://www.freedesktop.org/software/desktop-file-utils/releases/desktop-file-utils-0.26.tar.xz -o /sources/gnome/desktop-file-utils-0.26.tar.xz &&

tar xvf /sources/gnome/desktop-file-utils-0.26.tar.xz &&
cd       desktop-file-utils-0.26                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf desktop-file-utils-0.26
```

## libportal

For manipulating flatpak portals.

```sh
curl https://github.com/flatpak/libportal/releases/download/0.4/libportal-0.4.tar.xz -o /sources/gnome/libportal-0.4.tar.xz &&

tar xvf /sources/gnome/libportal-0.4.tar.xz &&
cd       libportal-0.4                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Dgtk_doc=false .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libportal-0.4
```

## Nautilus

The `GNOME` file manager.

```sh
curl https://download.gnome.org/sources/nautilus/40/nautilus-40.2.tar.xz -o /sources/gnome/nautilus-40.2.tar.xz &&

tar xvf /sources/gnome/nautilus-40.2.tar.xz &&
cd       nautilus-40.2                      &&

sed -e 's/test_env/#&/' \
    -e 's/env: \[/env: test_env + [/' \
    -i test/automated/displayless/meson.build &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Dselinux=false     \
      -Dpackagekit=false  \
      .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf nautilus-40.2
```

## Zenity

Dialog boxes.

```sh
curl https://download.gnome.org/sources/zenity/3.32/zenity-3.32.0.tar.xz -o /sources/gnome/zenity-3.32.0.tar.xz &&

tar xvf /sources/gnome/zenity-3.32.0.tar.xz &&
cd       zenity-3.32.0                      &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf zenity-3.32.0
```

## GNOME Bluetooth

Manipulate Bluetooth devices from the `GNOME` desktop.

```sh
curl https://download.gnome.org/sources/gnome-bluetooth/3.34/gnome-bluetooth-3.34.5.tar.xz -o /sources/gnome/gnome-bluetooth-3.34.5.tar.xz &&

tar xvf /sources/gnome/gnome-bluetooth-3.34.5.tar.xz &&
cd       gnome-bluetooth-3.34.5                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gnome-bluetooth-3.34.5
```

## GNOME Keyring

Desktop secrets management.

```sh
curl https://download.gnome.org/sources/gnome-keyring/40/gnome-keyring-40.0.tar.xz -o /sources/gnome/gnome-keyring-40.0.tar.xz &&

tar xvf /sources/gnome/gnome-keyring-40.0.tar.xz &&
cd       gnome-keyring-40.0                      &&

sed -i 's:"/desktop:"/org:' schema/*.xml &&

./configure --prefix=/usr     \
            --sysconfdir=/etc &&

make              &&
sudo make install &&

cd .. &&
rm -rf gnome-keyring-40.0
```

## GeoClue

Enabled creation of location-aware applications.

```sh
curl https://gitlab.freedesktop.org/geoclue/geoclue/-/archive/2.5.7/geoclue-2.5.7.tar.bz2 -o /sources/gnome/geoclue-2.5.7.tar.bz2 &&

tar xvf /sources/gnome/geoclue-2.5.7.tar.bz2 &&
cd       geoclue-2.5.7                       &&

mkdir build &&
cd    build &&

meson --prefix=/usr            \
      --buildtype=release      \
      -D3g-source=false        \
      -Dmodem-gps-source=false \
      -Dcdma-source=false      \
      -Dnmea-source=false      \
      -Dgtk-doc=false .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf geoclue-2.5.7
```

## CUPS

Not part of `GNOME`, but in an era where printing has been obsolete for a decade and I don't own a printer, the `GNOME` settings daemon will refuse to build if this is not present.

Kernel drivers are required if you want to actually use a printer.

```
Device Drivers  --->
  [*] USB support  --->                          [CONFIG_USB_SUPPORT]
    <*/M>  OHCI HCD (USB 1.1) support            [CONFIG_USB_OHCI_HCD]
    <*/M>  UHCI HCD (most Intel and VIA) support [CONFIG_USB_UHCI_HCD]
    <*/M>  USB Printer support                   [CONFIG_USB_PRINTER]
```

`CUPS` requires a system user:

```sh
sudo useradd -c "Print Service User" -d /var/spool/cups -g lp -s /bin/false -u 9 lp &&
sudo groupadd -g 19 lpadmin
```

```sh
curl https://github.com/OpenPrinting/cups/releases/download/v2.3.3op2/cups-2.3.3op2-source.tar.gz -o /sources/gnome/cups-2.3.3op2-source.tar.gz &&

tar xzvf /sources/gnome/cups-2.3.3op2-source.tar.gz &&
cd        cups-2.3.3op2                             &&

sed -e "s/-Wno-format-truncation//" \
    -i configure \
    -i config-scripts/cups-compiler.m4 &&

./configure --libdir=/usr/lib            \
            --with-system-groups=lpadmin \
            --with-docdir=/usr/share/cups/doc-2.3.3op2 &&

make              &&
sudo make install &&

sudo ln -svnf ../cups/doc-2.3.3op2 /usr/share/doc/cups-2.3.3op2 &&

cd .. &&
rm -rf cups-2.3.3op2
```

Configuration for CUPS:

```sh
sudo su -
echo "ServerName /run/cups/cups.sock" > /etc/cups/client.conf
cat > /etc/pam.d/cups << "EOF"
# Begin /etc/pam.d/cups

auth    include system-auth
account include system-account
session include system-session

# End /etc/pam.d/cups
EOF
systemctl enable cups
exit
```

## ModemManager

Also not a part of `GNOME`, but gnome-settings-daemon will also refuse to build without this. If you're reading this 25 years in the past and have a modem, cheers.

```sh
curl https://www.freedesktop.org/software/ModemManager/ModemManager-1.16.6.tar.xz -o /sources/gnome/ModemManager-1.16.6.tar.xz &&

tar xvf /sources/gnome/ModemManager-1.16.6.tar.xz &&
cd       ModemManager-1.16.6                      &&

./configure --prefix=/usr                 \
            --sysconfdir=/etc             \
            --localstatedir=/var          \
            --with-systemd-journal        \
            --with-systemd-suspend-resume \
            --without-mbim                \
            --without-qmi                 \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf ModemManager-1.16.6
```

## NetworkManager

Unfortunately, gnome-settings-daemon will also refuse to build without this, even though it is not needed. `NetworkManager` provides a convenient graphical display for managing network connections, but `systemd-networkd` works fine and was already configured to set up the base system.

First, `NetworkManager` has its own dependencies we would not otherwise need.

### Jansson

Another JSON library.

```sh
curl https://digip.org/jansson/releases/jansson-2.13.1.tar.gz -o /sources/gnome/jansson-2.13.1.tar.gz &&

tar xzvf /sources/gnome/jansson-2.13.1.tar.gz &&
cd        jansson-2.13.1                      &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf jansson-2.13.1
```

### libndp

IPv6 neighbor discovery protocol.

```sh
curl http://libndp.org/files/libndp-1.8.tar.gz -o /sources/gnome/libndp-1.8.tar.gz &&

tar xzvf /sources/gnome/libndp-1.8.tar.gz &&
cd        libndp-1.8                      &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static     &&

make              &&
sudo make install &&

cd .. &&
rm -rf libndp-1.8
```

### slang

Scripting language for embedding into applications.

```sh
curl https://www.jedsoft.org/releases/slang/slang-2.3.2.tar.bz2 -o /sources/gnome/slang-2.3.2.tar.bz2 &&

tar xvf /sources/gnome/slang-2.3.2.tar.bz2 &&
cd       slang-2.3.2                       &&

./configure --prefix=/usr \
            --sysconfdir=/etc \
            --with-readline=gnu &&

make -j1
sudo make install_doc_dir=/usr/share/doc/slang-2.3.2   \
          SLSH_DOC_DIR=/usr/share/doc/slang-2.3.2/slsh \
          install-all &&

sudo chmod -v 755 /usr/lib/libslang.so.2.3.2 \
                  /usr/lib/slang/v2/modules/*.so

cd .. &&
sudo rm -rf slang-2.3.2
```

### newt

Libraries for building a text user interface.

```sh
curl https://releases.pagure.org/newt/newt-0.52.21.tar.gz -o /sources/gnome/newt-0.52.21.tar.gz &&

tar xzvf /sources/gnome/newt-0.52.21.tar.gz &&
cd        newt-0.52.21                      &&

sed -e 's/^LIBNEWT =/#&/'                   \
    -e '/install -m 644 $(LIBNEWT)/ s/^/#/' \
    -e 's/$(LIBNEWT)/$(LIBNEWTSONAME)/g'    \
    -i Makefile.in                          &&

./configure --prefix=/usr           \
            --with-gpm-support      \
            --with-python=python3.9 &&

make              &&
sudo make install &&

cd .. &&
rm -rf newt-0.52.21
```

Now install `NetworkManager`.

```sh
curl https://download.gnome.org/sources/NetworkManager/1.30/NetworkManager-1.30.4.tar.xz -o /sources/gnome/NetworkManager-1.30.4.tar.xz &&

tar xvf /sources/gnome/NetworkManager-1.30.4.tar.xz &&
cd       NetworkManager-1.30.4                      &&

sed '/initrd/d' -i src/core/meson.build &&

mkdir build &&
cd    build    &&

CXXFLAGS+="-O2 -fPIC"            \
meson --prefix=/usr              \
      --buildtype=release        \
      -Dlibaudit=no              \
      -Dlibpsl=false             \
      -Dnmtui=true               \
      -Dovs=false                \
      -Dppp=false                \
      -Dselinux=false            \
      -Dqt=false                 \
      -Dsession_tracking=systemd \
      -Dmodem_manager=false      \
      .. &&

ninja                                              &&
sudo ninja install                                 &&
sudo mv -v /usr/share/doc/NetworkManager{,-1.30.4} &&

cd ../.. &&
rm -rf NetworkManager-1.30.4
```

In case you do want to use `NetworkManager`, it requires some configuration. You will also need to disable `systemd-networkd` and `iwd`, as you cannot have multiple daemons attempting to control your network interface devices.

Consult documentation from Beyond Linux From Scratch or the wonderful Arch Wiki or Gentoo Wiki to see how to configure it.

## GNOME Settings Daemon

```sh
curl https://download.gnome.org/sources/gnome-settings-daemon/40/gnome-settings-daemon-40.0.1.tar.xz -o /sources/gnome/gnome-settings-daemon-40.0.1.tar.xz &&

tar xvf /sources/gnome/gnome-settings-daemon-40.0.1.tar.xz &&
cd       gnome-settings-daemon-40.0.1                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gnome-settings-daemon-40.0.1
```

## Mutter

The `GNOME` window manager.

```sh
curl https://download.gnome.org/sources/mutter/40/mutter-40.2.tar.xz -o /sources/gnome/mutter-40.2.tar.xz &&

tar xvf /sources/gnome/mutter-40.2.tar.xz &&
cd       mutter-40.2                      &&

sed -i '/libmutter_dep = declare_dependency(/a sources: mutter_built_sources,' src/meson.build &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=debugoptimized .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf mutter-40.2
```

## Sound Theme FreeDesktop

```sh
curl https://people.freedesktop.org/~mccann/dist/sound-theme-freedesktop-0.8.tar.bz2 -o /sources/gnome/sound-theme-freedesktop-0.8.tar.bz2 &&

tar xvf /sources/gnome/sound-theme-freedesktop-0.8.tar.bz2 &&
cd       sound-theme-freedesktop-0.8                       &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf sound-theme-freedesktop-0.8
```

## adwaita icon theme

```sh
curl https://download.gnome.org/sources/adwaita-icon-theme/40/adwaita-icon-theme-40.1.1.tar.xz -o /sources/gnome/adwaita-icon-theme-40.1.1.tar.xz &&

tar xvf /sources/gnome/adwaita-icon-theme-40.1.1.tar.xz &&
cd       adwaita-icon-theme-40.1.1                      &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf adwaita-icon-theme-40.1.1
```

## libnma

GUI libraries for `NetworkManager`.

```sh
curl https://download.gnome.org/sources/libnma/1.8/libnma-1.8.30.tar.xz -o /sources/gnome/libnma-1.8.30.tar.xz &&

tar xvf /sources/gnome/libnma-1.8.30.tar.xz &&
cd       libnma-1.8.30                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr                             \
      --buildtype=release                       \
      -Dgtk_doc=false                           \
      -Dmobile_broadband_provider_info=false .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libnma-1.8.30
```

## NetworkManager Applet

```sh
curl https://download.gnome.org/sources/network-manager-applet/1.22/network-manager-applet-1.22.0.tar.xz -o /sources/gnome/network-manager-applet-1.22.0.tar.xz &&

tar xvf /sources/gnome/network-manager-applet-1.22.0.tar.xz &&
cd       network-manager-applet-1.22.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr     \
      -Dappindicator=no \
      -Dselinux=false   \
      -Dteam=false      .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf network-manager-applet-1.22.0
```

## polkit GNOME

```sh
curl https://download.gnome.org/sources/polkit-gnome/0.105/polkit-gnome-0.105.tar.xz -o /sources/gnome/polkit-gnome-0.105.tar.xz &&

tar xvf /sources/gnome/polkit-gnome-0.105.tar.xz &&
cd       polkit-gnome-0.105                      &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf polkit-gnome-0.105
```

`polkit-gnome` requires an autostart file to run:

```sh
sudo su -
mkdir -p /etc/xdg/autostart &&
cat > /etc/xdg/autostart/polkit-gnome-authentication-agent-1.desktop << "EOF"
[Desktop Entry]
Name=PolicyKit Authentication Agent
Comment=PolicyKit Authentication Agent
Exec=/usr/libexec/polkit-gnome-authentication-agent-1
Terminal=false
Type=Application
Categories=
NoDisplay=true
OnlyShowIn=GNOME;XFCE;Unity;
AutostartCondition=GNOME3 unless-session gnome
EOF
exit
```

## Cheese

An application for taking and viewing photos. I have no idea why, but GNOME control center requires this, even though I have no intention of ever taking any photos.

If you do intend to actually use a webcam, kernel drivers are needed.

```
Device Drivers  --->
  Multimedia support --->
    <*> Autoselect ancillary drivers (tuners, sensors, i2c, spi, frontends) [CONFIG_MEDIA_SUBDRV_AUTOSELECT]
    Media device types --->
      <*> Cameras/video grabbers support  [CONFIG_MEDIA_CAMERA_SUPPORT]
    Media drivers --->
      <*> Media USB Adapters  --->         [CONFIG_MEDIA_USB_SUPPORT]
                  Select device(s) as needed
```

```sh
curl https://download.gnome.org/sources/cheese/3.38/cheese-3.38.0.tar.xz -o /sources/gnome/cheese-3.38.0.tar.xz &&
curl https://www.linuxfromscratch.org/patches/blfs/svn/cheese-3.38.0-upstream_fixes-1.patch -o /sources/patches/cheese-3.38.0-upstream_fixes-1.patch &&

tar xvf /sources/gnome/cheese-3.38.0.tar.xz &&
cd       cheese-3.38.0                      &&

sed -i "s/&version;/3.38.0/" docs/reference/cheese{,-docs}.xml      &&
patch -Np1 -i /sources/patches/cheese-3.38.0-upstream_fixes-1.patch &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Dgtk_doc=false .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
sudo rm -rf cheese-3.38.0
```

## ibus

Not a part of `GNOME`, but required by GNOME Control Center. This unfortunately requires GTK-2, so we now need 3 different versions of GTK.

### GTK+-2

```sh
curl https://download.gnome.org/sources/gtk+/2.24/gtk+-2.24.33.tar.xz -o /sources/gnome/gtk+-2.24.33.tar.xz &&

tar xvf /sources/gnome/gtk+-2.24.33.tar.xz &&
cd       gtk+-2.24.33                      &&

sed -e 's#l \(gtk-.*\).sgml#& -o \1#' \
    -i docs/{faq,tutorial}/Makefile.in      &&

./configure --prefix=/usr \
            --sysconfdir=/etc &&

make              &&
sudo make install &&

cd .. &&
rm -rf gtk+-2.24.33
```

Now we can install `ibus`.

```sh
curl https://github.com/ibus/ibus/releases/download/1.5.24/ibus-1.5.24.tar.gz -o /sources/gnome/ibus-1.5.24.tar.gz &&
curl https://www.unicode.org/Public/zipped/13.0.0/UCD.zip -o /sources/UCD.zip &&

tar xzvf /sources/gnome/ibus-1.5.24.tar.gz &&
cd        ibus-1.5.24                      &&

sudo mkdir -p                      /usr/share/unicode/ucd &&
sudo unzip -uo /sources/UCD.zip -d /usr/share/unicode/ucd &&

sed -i 's@/desktop/ibus@/org/freedesktop/ibus@g' \
    data/dconf/org.freedesktop.ibus.gschema.xml &&

./configure --prefix=/usr             \
            --sysconfdir=/etc         \
            --disable-unicode-dict    \
            --disable-emoji-dict      &&
rm -f tools/main.c                    &&

make              &&
sudo make install &&

sudo gzip -dfv /usr/share/man/man{{1,5}/ibus*.gz,5/00-upstream-settings.5.gz} &&

cd .. &&
rm -rf ibus-1.5.24
```

## Samba

Unfortunately, GNOME Control Center seems to also require `smbclient`. `samba` itself has a few dependencies we need to grab.

### lmdb

A key-value store.

```sh
curl https://github.com/LMDB/lmdb/archive/LMDB_0.9.29.tar.gz -o /sources/LMDB_0.9.29.tar.gz &&

tar xzvf /sources/LMDB_0.9.29.tar.gz &&
cd        lmdb-LMDB_0.9.29                &&

cd libraries/liblmdb &&
make                 &&

sed -i 's| liblmdb.a||' Makefile &&
sudo make prefix=/usr install    &&

cd ../../.. &&
rm -rf lmdb-LMDB_0.9.29
```

### rpcsvc-proto

```sh
curl https://github.com/thkukuk/rpcsvc-proto/releases/download/v1.4.2/rpcsvc-proto-1.4.2.tar.xz -o /sources/rpcsvc-proto-1.4.2.tar.xz &&

tar xvf /sources/rpcsvc-proto-1.4.2.tar.xz &&
cd       rpcsvc-proto-1.4.2                &&

./configure --sysconfdir=/etc &&

make              &&
sudo make install &&

cd .. &&
rm -rf rpcsvc-proto-1.4.2
```

### YAPP Perl module

```sh
curl https://www.cpan.org/authors/id/W/WB/WBRASWELL/Parse-Yapp-1.21.tar.gz -o /sources/Parse-Yapp-1.21.tar.gz &&

tar xzvf /sources/Parse-Yapp-1.21.tar.gz &&
cd        Parse-Yapp-1.21                &&

perl Makefile.PL  &&
make              &&
sudo make install &&

cd .. &&
rm -rf Parse-Yapp-1.21
```

### python-iso8601

```sh
curl https://github.com/micktwomey/pyiso8601/archive/refs/tags/0.1.14.tar.gz -o /sources/python-iso8601-0.1.14.tar.gz &&

tar xzvf /sources/python-iso8601-0.1.14.tar.gz &&
cd        pyiso8601-0.1.14                     &&

sudo python3 setup.py install --optimize=1 &&

cd .. &&
rm -rf pyiso8601-0.1.14
```

### python-setuptools_rust

```sh
curl https://files.pythonhosted.org/packages/12/22/6ba3031e7cbd6eb002e13ffc7397e136df95813b6a2bd71ece52a8f89613/setuptools-rust-0.12.1.tar.gz -o /sources/setuptools_rust-0.12.1.tar.gz &&

tar xzvf /sources/setuptools_rust-0.12.1.tar.gz &&
cd        setuptools-rust-0.12.1                &&

python3 setup.py build                     &&
sudo python3 setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf setuptools-rust-0.12.1
```

### python-cryptography

```sh
curl https://github.com/pyca/cryptography/archive/refs/tags/3.4.7.tar.gz -o /sources/python-cryptography-3.4.7.tar.gz &&

tar xzvf /sources/python-cryptography-3.4.7.tar.gz &&
cd        cryptography-3.4.7                       &&

python3 setup.py build                     &&
sudo python3 setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf cryptography-3.4.7
```

### python-asn1

```sh
curl https://files.pythonhosted.org/packages/a4/db/fffec68299e6d7bad3d504147f9094830b704527a7fc098b721d38cc7fa7/pyasn1-0.4.8.tar.gz -o /sources/pyasn1-0.4.8.tar.gz &&

tar xzvf /sources/pyasn1-0.4.8.tar.gz &&
cd        pyasn1-0.4.8                &&

python3 setup.py build                     &&
sudo python3 setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf pyasn1-0.4.8
```

Now we can install `samba`.

```sh
curl https://www.samba.org/ftp/samba/stable/samba-4.14.5.tar.gz -o /sources/samba-4.14.5.tar.gz &&

tar xzvf /sources/samba-4.14.5.tar.gz &&
cd        samba-4.14.5                &&

CPPFLAGS="-I/usr/include/tirpc"            \
LDFLAGS="-ltirpc"                          \
./configure                                \
    --prefix=/usr                          \
    --sysconfdir=/etc                      \
    --localstatedir=/var                   \
    --with-piddir=/run/samba               \
    --with-pammodulesdir=/usr/lib/security \
    --enable-fhs                           \
    --without-ad-dc                        \
    --enable-selftest &&

make              &&
sudo make install &&

sudo install -v -m644    examples/smb.conf.default /etc/samba &&
sudo mkdir   -pv /etc/openldap/schema                         &&

sudo install -v -m644    examples/LDAP/README              \
                        /etc/openldap/schema/README.LDAP      &&
sudo install -v -m644    examples/LDAP/samba*              \
                        /etc/openldap/schema                  &&
sudo install -v -m755    examples/LDAP/{get*,ol*} \
                        /etc/openldap/schema                  &&

cd .. &&
rm -rf samba-4.14.5
```

## GNOME Control Center

```sh
curl https://download.gnome.org/sources/gnome-control-center/40/gnome-control-center-40.0.tar.xz -o /sources/gnome/gnome-control-center-40.0.tar.xz &&

tar xvf /sources/gnome/gnome-control-center-40.0.tar.xz &&
cd       gnome-control-center-40.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gnome-control-center-40.0
```

## GNOME Shell

The heart of the desktop itself.

```sh
curl https://download.gnome.org/sources/gnome-shell/40/gnome-shell-40.2.tar.xz -o /sources/gnome/gnome-shell-40.2.tar.xz &&

tar xvf /sources/gnome/gnome-shell-40.2.tar.xz &&
cd       gnome-shell-40.2                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gnome-shell-40.2
```

## GNOME Shell Extensions

```sh
curl https://download.gnome.org/sources/gnome-shell-extensions/40/gnome-shell-extensions-40.2.tar.xz -o /sources/gnome/gnome-shell-extensions-40.2.tar.xz &&

tar xvf /sources/gnome/gnome-shell-extensions-40.2.tar.xz &&
cd       gnome-shell-extensions-40.2                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr .. &&

sudo ninja install &&

cd ../.. &&
rm -rf gnome-shell-extensions-40.2
```

## GNOME Session

```sh
curl https://download.gnome.org/sources/gnome-session/40/gnome-session-40.1.1.tar.xz -o /sources/gnome/gnome-session-40.1.1.tar.xz &&

tar xvf /sources/gnome/gnome-session-40.1.1.tar.xz &&
cd       gnome-session-40.1.1                      &&

sed 's@/bin/sh@/bin/sh -l@' -i gnome-session/gnome-session.in &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

sudo mv -v /usr/share/doc/gnome-session{,-40.1.1} &&

cd ../.. &&
rm -rf gnome-session-40.1.1
```

## GDM

The `GNOME` graphical login manager.

First, create a system user to run the daemon:

```sh
sudo groupadd -g 21 gdm               &&
sudo useradd -c "GDM Daemon Owner" -d /var/lib/gdm -u 21 \
             -g gdm -s /bin/false gdm &&
sudo passwd  -ql gdm
```

Then build and install.

```sh
curl https://download.gnome.org/sources/gdm/40/gdm-40.0.tar.xz -o /sources/gnome/gdm-40.0.tar.xz &&

tar xvf /sources/gnome/gdm-40.0.tar.xz &&
cd       gdm-40.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Dplymouth=disabled \
      -Dgdm-xsession=true .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gdm-40.0
```

To automatically start the service at boot:

```sh
sudo systemctl enable gdm
```

If you wish to enable automatic login:

```sh
sudo su -
cat > /etc/gdm/custom.conf << EOF
# Start GDM configuraiton settings

[daemon]
AutomaticLogin=lfs
AutomaticLoginEnable=true

[security]

[xdmcp]

[chooser]

[debug]
# Uncommend the line below to turn on debugging
#Enable=true

# End GDM configuration settings
EOF
exit
```

Everything else is not strictly needed for a graphical desktop, but will make things nicer, prettier, and more usable.

## GNOME User Documentation

```sh
curl https://download.gnome.org/sources/gnome-user-docs/40/gnome-user-docs-40.1.tar.xz -o /sources/gnome/gnome-user-docs-40.1.tar.xz &&

tar xvf /sources/gnome/gnome-user-docs-40.1.tar.xz &&
cd       gnome-user-docs-40.1                      &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf gnome-user-docs-40.1
```

## Yelp XSL

Stylesheets to format the `GNOME` help pages.

```sh
curl https://download.gnome.org/sources/yelp-xsl/40/yelp-xsl-40.2.tar.xz -o /sources/gnome/yelp-xsl-40.2.tar.xz &&

tar xvf /sources/gnome/yelp-xsl-40.2.tar.xz &&
cd       yelp-xsl-40.2                      &&

./configure --prefix=/usr &&

sudo make install &&

cd .. &&
rm -rf yelp-xsl-40.2
```

## Yelp

The `GNOME` help browser.

```sh
curl https://download.gnome.org/sources/yelp/40/yelp-40.2.tar.xz -o /sources/gnome/yelp-40.2.tar.xz &&

tar xvf /sources/gnome/yelp-40.2.tar.xz &&
cd       yelp-40.2                      &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

sudo update-desktop-database &&

cd .. &&
rm -rf yelp-40.2
```

## GNOME Themes Extra

```sh
curl https://download.gnome.org/sources/gnome-themes-extra/3.28/gnome-themes-extra-3.28.tar.xz -o /sources/gnome/gnome-themes-extra-3.28.tar.xz &&

tar xvf /sources/gnome/gnome-themes-extra-3.28.tar.xz &&
cd       gnome-themes-extra-3.28                      &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf  gnome-themes-extra-3.28
```

## XML::Simple Perl Module

Deprecated, but required by `GNOME`'s icon naming utils.

```sh
curl https://www.cpan.org/authors/id/G/GR/GRANTM/XML-Simple-2.25.tar.gz -o /sources/XML-Simple-2.25.tar.gz &&

tar xzvf /sources/XML-Simple-2.25.tar.gz &&
cd        XML-Simple-2.25                &&

perl Makefile.PL  &&
make              &&
sudo make install &&

cd .. &&
rm -rf XML-Simple-2.25
```

## Icon Naming Utils

```sh
curl http://tango.freedesktop.org/releases/icon-naming-utils-0.8.90.tar.bz2 -o /sources/gnome/icon-naming-utils-0.8.90.tar.bz2 &&

tar xvf /sources/gnome/icon-naming-utils-0.8.90.tar.bz2 &&
cd       icon-naming-utils-0.8.90                       &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf icon-naming-utils-0.8.90
```

## HiColor Icon Theme

```sh
curl https://icon-theme.freedesktop.org/releases/hicolor-icon-theme-0.17.tar.xz -o /sources/gnome/hicolor-icon-theme-0.17.tar.xz &&

tar xvf /sources/gnome/hicolor-icon-theme-0.17.tar.xz &&
cd       hicolor-icon-theme-0.17                      &&

./configure --prefix=/usr &&

sudo make install &&

cd .. &&
rm -rf hicolor-icon-theme-0.17
```

## GNOME Icon Theme

```sh
curl https://download.gnome.org/sources/gnome-icon-theme/3.12/gnome-icon-theme-3.12.0.tar.xz -o /sources/gnome/gnome-icon-theme-3.12.0.tar.xz &&

tar xvf /sources/gnome/gnome-icon-theme-3.12.0.tar.xz &&
cd       gnome-icon-theme-3.12.0                      &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf gnome-icon-theme-3.12.0
```

## GNOME Icon Theme Extras

```sh
curl https://download.gnome.org/sources/gnome-icon-theme-extras/3.12/gnome-icon-theme-extras-3.12.0.tar.xz -o /sources/gnome/gnome-icon-theme-extras-3.12.0.tar.xz &&

tar xvf /sources/gnome/gnome-icon-theme-extras-3.12.0.tar.xz &&
cd       gnome-icon-theme-extras-3.12.0                      &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf gnome-icon-theme-extras-3.12.0
```
