# Xorg

Create a separate directory to store the enormous number of source packages in the modular Xorg structure.

```sh
mkdir -pv /sources/xorg
```

## util-macros

```sh
curl https://www.x.org/pub/individual/util/util-macros-1.19.3.tar.bz2 -o /sources/xorg/util-macros-1.19.3.tar.bz2 &&

tar xvf /sources/xorg/util-macros-1.19.3.tar.bz2 &&
cd       util-macros-1.19.3                      &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static &&

sudo make install &&

cd .. &&
rm -rf util-macros-1.19.3
```

## xorg-proto

```sh
curl https://xorg.freedesktop.org/archive/individual/proto/xorgproto-2021.4.tar.bz2 -o /sources/xorg/xorgproto-2021.4.tar.bz2 &&

tar xvf /sources/xorg/xorgproto-2021.4.tar.bz2 &&
cd       xorgproto-2021.4                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      -Dlegacy=true .. &&

ninja              &&
sudo ninja install &&

sudo install -vdm755 /usr/share/doc/xorgproto-2021.4                        &&
sudo install -vm644 ../[^m]*.txt ../PM_spec /usr/share/doc/xorgproto-2021.4 &&

cd ../.. &&
rm -rf xorgproto-2021.4
```

## libXau

```sh
curl https://www.x.org/pub/individual/lib/libXau-1.0.9.tar.bz2 -o /sources/xorg/libXau-1.0.9.tar.bz2 &&

tar xvf /sources/xorg/libXau-1.0.9.tar.bz2 &&
cd       libXau-1.0.9                      &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libXau-1.0.9
```

## libXdmcp

```sh
curl https://www.x.org/pub/individual/lib/libXdmcp-1.1.3.tar.bz2 -o /sources/xorg/libXdmcp-1.1.3.tar.bz2 &&

tar xvf /sources/xorg/libXdmcp-1.1.3.tar.bz2 &&
cd       libXdmcp-1.1.3                      &&

./configure --prefix=/usr                          \
            --sysconfdir=/etc                      \
            --localstatedir=/var                   \
            --docdir=/usr/share/doc/libXdmcp-1.1.3 \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libXdmcp-1.1.3
```

## xcb-proto

```sh
curl https://xorg.freedesktop.org/archive/individual/proto/xcb-proto-1.14.1.tar.xz -o /sources/xorg/xcb-proto-1.14.1.tar.xz &&

tar xvf /sources/xorg/xcb-proto-1.14.1.tar.xz &&
cd       xcb-proto-1.14.1                     &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static &&

sudo make install &&

cd .. &&
rm -rf xcb-proto-1.14.1
```

## libxcb

```sh
curl https://xorg.freedesktop.org/archive/individual/lib/libxcb-1.14.tar.xz -o /sources/xorg/libxcb-1.14.tar.xz &&

tar xvf /sources/xorg/libxcb-1.14.tar.xz &&
cd       libxcb-1.14                     &&

CFLAGS="${CFLAGS:--O2 -g} -Wno-error=format-extra-args" \
./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --docdir=/usr/share/doc/libxcb-1.14 &&

make              &&
sudo make install &&

cd .. &&
rm -rf libxcb-1.14
```

## Xorg libraries

Beyond Linux From Scratch provides these as a list due to the large number.

First, create the list of files and md5 sums:

```sh
cat > /sources/xorg/lib-7.md5 << "EOF"
ce2fb8100c6647ee81451ebe388b17ad  xtrans-1.4.0.tar.bz2
a9a24be62503d5e34df6b28204956a7b  libX11-1.7.2.tar.bz2
f5b48bb76ba327cd2a8dc7a383532a95  libXext-1.3.4.tar.bz2
4e1196275aa743d6ebd3d3d5ec1dff9c  libFS-1.0.8.tar.bz2
76d77499ee7120a56566891ca2c0dbcf  libICE-1.0.10.tar.bz2
87c7fad1c1813517979184c8ccd76628  libSM-1.2.3.tar.bz2
eeea9d5af3e6c143d0ea1721d27a5e49  libXScrnSaver-1.2.3.tar.bz2
b122ff9a7ec70c94dbbfd814899fffa5  libXt-1.2.1.tar.bz2
ac774cff8b493f566088a255dbf91201  libXmu-1.1.3.tar.bz2
6f0ecf8d103d528cfc803aa475137afa  libXpm-3.5.13.tar.bz2
c1ce21c296bbf3da3e30cf651649563e  libXaw-1.0.14.tar.bz2
86f182f487f4f54684ef6b142096bb0f  libXfixes-6.0.0.tar.bz2
3fa0841ea89024719b20cd702a9b54e0  libXcomposite-0.4.5.tar.bz2
802179a76bded0b658f4e9ec5e1830a4  libXrender-0.9.10.tar.bz2
9b9be0e289130fb820aedf67705fc549  libXcursor-1.2.0.tar.bz2
e3f554267a7a04b042dc1f6352bd6d99  libXdamage-1.1.5.tar.bz2
6447db6a689fb530c218f0f8328c3abc  libfontenc-1.1.4.tar.bz2
00516bed7ec1453d56974560379fff2f  libXfont2-2.0.4.tar.bz2
4a433c24627b4ff60a4dd403a0990796  libXft-2.3.3.tar.bz2
62c4af0839072024b4b1c8cbe84216c7  libXi-1.7.10.tar.bz2
0d5f826a197dae74da67af4a9ef35885  libXinerama-1.1.4.tar.bz2
18f3b20d522f45e4dadd34afb5bea048  libXrandr-1.5.2.tar.bz2
e142ef0ed0366ae89c771c27cfc2ccd1  libXres-1.2.1.tar.bz2
ef8c2c1d16a00bd95b9fdcef63b8a2ca  libXtst-1.2.3.tar.bz2
210b6ef30dda2256d54763136faa37b9  libXv-1.0.11.tar.bz2
3569ff7f3e26864d986d6a21147eaa58  libXvMC-1.0.12.tar.bz2
0ddeafc13b33086357cfa96fae41ee8e  libXxf86dga-1.1.5.tar.bz2
298b8fff82df17304dfdb5fe4066fe3a  libXxf86vm-1.1.4.tar.bz2
d2f1f0ec68ac3932dd7f1d9aa0a7a11c  libdmx-1.1.4.tar.bz2
b34e2cbdd6aa8f9cc3fa613fd401a6d6  libpciaccess-0.16.tar.bz2
dd7e1e946def674e78c0efbc5c7d5b3b  libxkbfile-1.1.0.tar.bz2
42dda8016943dc12aff2c03a036e0937  libxshmfence-1.3.tar.bz2
EOF
```

Then perform the downloads:

```sh
pushd /sources/xorg                          &&
mkdir lib                                    &&
cd lib                                       &&
grep -v '^#' ../lib-7.md5 | awk '{print $2}' | wget -i- -c \
    -B https://www.x.org/pub/individual/lib/ &&
md5sum -c ../lib-7.md5                       &&
popd
```

Then build each library.

```sh
for package in $(grep -v '^#' /sources/xorg/lib-7.md5 | awk '{print $2}')
do
  packagedir=${package%.tar.bz2}
  tar -xf /sources/xorg/lib/$package &&
  pushd $packagedir                  &&
  case $packagedir in
    libICE* )
      ./configure --prefix=/usr        \
                  --sysconfdir=/etc    \
                  --localstatedir=/var \
                  --disable-static     \
                  --docdir=/usr/share/doc/$package ICE_LIBS=-lpthread &&
    ;;

    libXfont2-[0-9]* )
      ./configure --prefix=/usr                    \
                  --sysconfdir=/etc                \
                  --localstatedir=/var             \
                  --disable-static                 \
                  --docdir=/usr/share/doc/$package \
                  --disable-devel-docs &&
    ;;

    libXt-[0-9]* )
      ./configure --prefix=/usr                    \
                  --sysconfdir=/etc                \
                  --localstatedir=/var             \
                  --disable-static                 \
                  --docdir=/usr/share/doc/$package \
                  --with-appdefaultdir=/etc/X11/app-defaults &&
    ;;

    * )
      ./configure --prefix=/usr        \
                  --sysconfdir=/etc    \
                  --localstatedir=/var \
                  --disable-static     \
                  --docdir=/usr/share/doc/$package &&
    ;;
  esac
  make                &&
  sudo make install   &&
  popd                &&
  rm -rf $packagedir  &&
  sudo /sbin/ldconfig &&
done
```

## xcb-util

```sh
curl https://xcb.freedesktop.org/dist/xcb-util-0.4.0.tar.bz2 -o /sources/xorg/xcb-util-0.4.0.tar.bz2 &&

tar xvf /sources/xorg/xcb-util-0.4.0.tar.bz2 &&
cd       xcb-util-0.4.0                      &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf xcb-util-0.4.0
```

## xcb-util-image

```sh
curl https://xcb.freedesktop.org/dist/xcb-util-image-0.4.0.tar.bz2 -o /sources/xorg/xcb-util-image-0.4.0.tar.bz2 &&

tar xvf /sources/xorg/xcb-util-image-0.4.0.tar.bz2 &&
cd       xcb-util-image-0.4.0                      &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf xcb-util-image-0.4.0
```

## xcb-util-keysyms

```sh
curl https://xcb.freedesktop.org/dist/xcb-util-keysyms-0.4.0.tar.bz2 -o /sources/xorg/xcb-util-keysyms-0.4.0.tar.bz2 &&

tar xvf /sources/xorg/xcb-util-keysyms-0.4.0.tar.bz2 &&
cd       xcb-util-keysyms-0.4.0                      &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf xcb-util-keysyms-0.4.0
```

## xcb-render-util

```sh
curl https://xcb.freedesktop.org/dist/xcb-util-renderutil-0.3.9.tar.bz2 -o /sources/xorg/xcb-util-renderutil-0.3.9.tar.bz2 &&

tar xvf /sources/xorg/xcb-util-renderutil-0.3.9.tar.bz2 &&
cd       xcb-util-renderutil-0.3.9                      &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf xcb-util-renderutil-0.3.9
```

## xcb-util-wm

```sh
curl https://xcb.freedesktop.org/dist/xcb-util-wm-0.4.1.tar.bz2 -o /sources/xorg/xcb-util-wm-0.4.1.tar.bz2 &&

tar xvf /sources/xorg/xcb-util-wm-0.4.1.tar.bz2 &&
cd       xcb-util-wm-0.4.1                      &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf xcb-util-wm-0.4.1
```

## xcb-util-cursor

```sh
curl https://xcb.freedesktop.org/dist/xcb-util-cursor-0.1.3.tar.bz2 -o /sources/xorg/xcb-util-cursor-0.1.3.tar.bz2 &&

tar xvf /sources/xorg/xcb-util-cursor-0.1.3.tar.bz2 &&
cd       xcb-util-cursor-0.1.3                      &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf xcb-util-cursor-0.1.3
```

## libdrm

```sh
curl https://dri.freedesktop.org/libdrm/libdrm-2.4.106.tar.xz -o /sources/xorg/libdrm-2.4.106.tar.xz &&

tar xvf /sources/xorg/libdrm-2.4.106.tar.xz &&
cd       libdrm-2.4.106                     &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Dudev=true         \
      -Dvalgrind=false &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libdrm-2.4.106
```

## Wayland

```sh
curl https://wayland.freedesktop.org/releases/wayland-1.19.0.tar.xz -o /sources/wayland-1.19.0.tar.xz &&

tar xvf /sources/wayland-1.19.0.tar.xz &&
cd       wayland-1.19.0                &&

./configure --prefix=/usr    \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf wayland-1.19.0
```

## Wayland Protocols

```sh
curl https://wayland.freedesktop.org/releases/wayland-protocols-1.21.tar.xz -o /sources/wayland-protocols-1.21.tar.xz &&

tar xvf /sources/wayland-protocols-1.21.tar.xz &&
cd       wayland-protocols-1.21                &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf wayland-protocols-1.21
```

## Mesa

```sh
curl https://mesa.freedesktop.org/archive/mesa-21.1.2.tar.xz -o /sources/xorg/mesa-21.1.2.tar.xz &&

tar xvf /sources/xorg/mesa-21.1.2.tar.xz &&
cd       mesa-21.1.2                     &&

mkdir build &&
cd    build &&

meson --prefix=/usr           \
      --buildtype=release     \
      -Dplatforms=x11,wayland \
      -Dvalgrind=disabled     \
      -Dlibunwind=disabled    \
      .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf mesa-21.1.2
```

## xbitmaps

```sh
curl https://www.x.org/pub/individual/data/xbitmaps-1.1.2.tar.bz2 -o /sources/xorg/xbitmaps-1.1.2.tar.bz2 &&

tar xvf /sources/xorg/xbitmaps-1.1.2.tar.bz2 &&
cd       xbitmaps-1.1.2                      &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf xbitmaps-1.1.2
```

## XKeyboardConfig

```sh
curl https://www.x.org/pub/individual/data/xkeyboard-config/xkeyboard-config-2.33.tar.bz2 -o /sources/xorg/xkeyboard-config-2.33.tar.bz2 &&

tar xvf /sources/xorg/xkeyboard-config-2.33.tar.bz2 &&
cd       xkeyboard-config-2.33                      &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static     \
            --with-xkb-rules-symlink=xorg &&

make              &&
sudo make install &&

cd .. &&
rm -rf xkeyboard-config-2.33
```

## Xorg server

```sh
curl https://www.x.org/pub/individual/xserver/xorg-server-1.20.11.tar.bz2 -o /sources/xorg/xorg-server-1.20.11.tar.bz2 &&

tar xvf /sources/xorg/xorg-server-1.20.11.tar.bz2 &&
cd       xorg-server-1.20.11                      &&

./configure --prefix=/usr         \
            --sysconfdir=/etc     \
            --localstatedir=/var  \
            --disable-static      \
            --enable-glamor       \
            --enable-suid-wrapper \
            --with-xkb-output=/var/lib/xkb &&

make                                &&
sudo make install                   &&
sudo mkdir -pv /etc/X11/xorg.conf.d &&

cd .. &&
rm -rf xorg-server-1.20.11
```

## libevdev

The kernel needs to be compiled with the following options to support generic inputs:

```
Device Drivers  --->
  Input device support --->
    <*> Generic input layer (needed for keyboard, mouse, ...) [CONFIG_INPUT]
    <*>   Event interface                   [CONFIG_INPUT_EVDEV]
    [*]   Miscellaneous devices  --->       [CONFIG_INPUT_MISC]
      <*>    User level driver support      [CONFIG_INPUT_UINPUT]
```

```sh
curl https://www.freedesktop.org/software/libevdev/libevdev-1.11.0.tar.xz -o /sources/xorg/libevdev-1.11.0.tar.xz &&

tar xvf /sources/xorg/libevdev-1.11.0.tar.xz &&
cd       libevdev-1.11.0                     &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libevdev-1.11.0
```

## Xorg evdev driver

```sh
curl https://www.x.org/pub/individual/driver/xf86-input-evdev-2.10.6.tar.bz2 -o /sources/xorg/xf86-input-evdev-2.10.6.tar.bz2 &&

tar xvf /sources/xorg/xf86-input-evdev-2.10.6.tar.bz2 &&
cd       xf86-input-evdev-2.10.6                      &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf xf86-input-evdev-2.10.6
```

## libinput

```sh
curl https://www.freedesktop.org/software/libinput/libinput-1.18.0.tar.xz -o /sources/xorg/libinput-1.18.0.tar.xz &&

tar xvf /sources/xorg/libinput-1.18.0.tar.xz &&
cd       libinput-1.18.0                     &&

mkdir build &&
cd    build &&

meson --prefix=/usr         \
      --buildtype=release   \
      -Ddebug-gui=false     \
      -Dtests=false         \
      -Ddocumentation=false \
      -Dlibwacom=false      \
      .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libinput-1.18.0
```

## Xorg libinput driver

```sh
curl https://www.x.org/pub/individual/driver/xf86-input-libinput-1.0.1.tar.bz2 -o /sources/xorg/xf86-input-libinput-1.0.1.tar.bz2 &&

tar xvf /sources/xorg/xf86-input-libinput-1.0.1.tar.bz2 &&
cd       xf86-input-libinput-1.0.1                      &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf xf86-input-libinput-1.0.1
```

## libwacom

```sh
curl https://github.com/linuxwacom/libwacom/releases/download/libwacom-1.10/libwacom-1.10.tar.bz2 -o /sources/xorg/libwacom-1.10.tar.bz2 &&

tar xvf /sources/xorg/libwacom-1.10.tar.bz2 &&
cd       libwacom-1.10                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Dtests=disabled .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libwacom-1.10
```

## Wacom tablet driver

Required for `GNOME` settings daemon.

Requires kernel drivers if you wish to actually use a Wacom tablet.

```
Device Drivers  --->
  HID support  --->
    -*- HID bus support                                      [CONFIG_HID]
         Special HID drivers --->
              <*/M> Wacom Intuos/Graphire tablet support (USB) [CONFIG_HID_WACOM]
```

```sh
curl https://github.com/linuxwacom/xf86-input-wacom/releases/download/xf86-input-wacom-0.40.0/xf86-input-wacom-0.40.0.tar.bz2 -o /sources/xorg/xf86-input-wacom-0.40.0.tar.bz2 &&

tar xvf /sources/xorg/xf86-input-wacom-0.40.0.tar.bz2 &&
cd       xf86-input-wacom-0.40.0                      &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static     \
            --with-xkb-rules-symlink=xorg &&

make              &&
sudo make install &&

cd .. &&
rm -rf xf86-input-wacom-0.40.0
```

## libva

```sh
curl https://github.com/intel/libva/releases/download/2.11.0/libva-2.11.0.tar.bz2 -o /sources/xorg/libva-2.11.0.tar.bz2 &&

tar xvf /sources/xorg/libva-2.11.0.tar.bz2 &&
cd       libva-2.11.0                      &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libva-2.11.0
```

## libvdpau

```sh
curl https://gitlab.freedesktop.org/vdpau/libvdpau/-/archive/1.4/libvdpau-1.4.tar.bz2 -o /sources/xorg/libvdpau-1.4.tar.bz2 &&

tar xvf /sources/xorg/libvdpau-1.4.tar.bz2 &&
cd       libvdpau-1.4                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libvdpau-1.4
```

## libvdpau_gl

```sh
curl https://github.com/i-rinat/libvdpau-va-gl/archive/v0.4.0/libvdpau-va-gl-0.4.0.tar.gz -o /sources/xorg/libvdpau-va-gl-0.4.0.tar.gz &&

tar xzvf /sources/xorg/libvdpau-va-gl-0.4.0.tar.gz &&
cd        libvdpau-va-gl-0.4.0                     &&

mkdir build &&
cd    build &&

cmake -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_INSTALL_PREFIX=/usr .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf libvdpau-va-gl-0.4.0
```

## GPM

Mouse daemon that provides cut and paste capabilities in a console. This requires kernel drivers.

```
Device Drivers  --->
  Input device support ---> [CONFIG_INPUT]
    <*/M> Mouse interface   [CONFIG_INPUT_MOUSEDEV]
```

```sh
curl https://anduin.linuxfromscratch.org/BLFS/gpm/gpm-1.20.7.tar.bz2 -o /sources/gpm-1.20.7.tar.bz2 &&
curl https://www.linuxfromscratch.org/patches/blfs/svn/gpm-1.20.7-consolidated-1.patch -o /sources/patches/gpm-1.20.7-consolidated-1.patch &&

tar xvf /sources/gpm-1.20.7.tar.bz2 &&
cd       gpm-1.20.7                 &&

patch -Np1 -i /sources/patches/gpm-1.20.7-consolidated-1.patch &&

./autogen.sh                  &&
./configure --prefix=/usr \
            --sysconfdir=/etc &&

make              &&
sudo make install &&

sudo install-info --dir-file=/usr/share/info/dir           \
                  /usr/share/info/gpm.info                 &&

sudo ln -sfv libgpm.so.2.1.0 /usr/lib/libgpm.so            &&
sudo install -v -m644 conf/gpm-root.conf /etc              &&

sudo install -v -m755 -d /usr/share/doc/gpm-1.20.7/support &&
sudo install -v -m644    doc/support/*                     \
                        /usr/share/doc/gpm-1.20.7/support  &&
sudo install -v -m644    doc/{FAQ,HACK_GPM,README*}        \
                        /usr/share/doc/gpm-1.20.7          &&

cd .. &&
rm -rf gpm-1.20.7
```

## GLU

```sh
curl ftp://ftp.freedesktop.org/pub/mesa/glu/glu-9.0.1.tar.xz -o /sources/xorg/glu-9.0.1.tar.xz &&

tar xvf /sources/xorg/glu-9.0.1.tar.xz &&
cd       glu-9.0.1                     &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf glu-9.0.1
```

## libxkbcommon

```sh
curl https://xkbcommon.org/download/libxkbcommon-1.3.0.tar.xz -o /sources/xorg/libxkbcommon-1.3.0.tar.xz &&

tar xvf /sources/xorg/libxkbcommon-1.3.0.tar.xz &&
cd       libxkbcommon-1.3.0                     &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Denable-docs=false .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libxkbcommon-1.3.0
```

## startup-notification

Library for notifying a user that an application is loading.

```sh
curl https://www.freedesktop.org/software/startup-notification/releases/startup-notification-0.12.tar.gz -o /sources/xorg/startup-notification-0.12.tar.gz &&

tar xzvf /sources/xorg/startup-notification-0.12.tar.gz &&
cd        startup-notification-0.12                     &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&
sudo install -v -m644 -D doc/startup-notification.txt \
    /usr/share/doc/startup-notification-0.12/startup-notification.txt &&

cd .. &&
rm -rf startup-notification-0.12
```

## Links

Required by `xdg-utils`.

```sh
curl http://links.twibright.com/download/links-2.23.tar.bz2 -o /sources/links-2.23.tar.bz2 &&

tar xvf /sources/links-2.23.tar.bz2 &&
cd       links-2.23                 &&

./configure --prefix=/usr \
            --mandir=/usr/share/man &&

make              &&
sudo make install &&

sudo install -v -d -m755 /usr/share/doc/links-2.23 &&
sudo install -v -m644 doc/links_cal/* KEYS BRAILLE_HOWTO \
    /usr/share/doc/links-2.23                      &&

cd .. &&
rm -rf links-2.23
```

## xclock

```sh
curl https://www.x.org/pub/individual/app/xclock-1.0.9.tar.bz2 -o /sources/xorg/xclock-1.0.9.tar.bz2 &&

tar xvf /sources/xorg/xclock-1.0.9.tar.bz2 &&
cd       xclock-1.0.9                      &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf xclock-1.0.9
```

## xterm

```sh
curl https://invisible-mirror.net/archives/xterm/xterm-368.tgz -o /sources/xorg/xterm-368.tgz &&

tar xzvf /sources/xorg/xterm-368.tgz &&
cd        xterm-368                  &&

sed -i '/v0/{n;s/new:/new:kb=^?:/}' termcap &&
printf '\tkbs=\\177,\n' >> terminfo &&

TERMINFO=/usr/share/terminfo \
./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static     \
    --with-app-defaults=/etc/X11/app-defaults &&

make                 &&
sudo make install    &&
sudo make install-ti &&

sudo mkdir -pv /usr/share/applications        &&
sudo cp -v *.desktop /usr/share/applications/ &&

cd .. &&
rm -rf xterm-368
```

```sh
sudo su -
cat >> /etc/X11/app-defaults/XTerm << "EOF"
*VT100*locale: true
*VT100*faceName: Monospace
*VT100*faceSize: 10
*backarrowKeyIsErase: true
*ptyInitialErase: true
EOF
exit
```

## Xinit

```sh
curl https://www.x.org/pub/individual/app/xinit-1.4.1.tar.bz2 -o /sources/xorg/xinit-1.4.1.tar.bz2 &&

tar xvf /sources/xorg/xinit-1.4.1.tar.bz2 &&
cd       xinit-1.4.1                      &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static     \
            --with-xinitdir=/etc/X11/app-defaults &&

make              &&
sudo make install &&
sudo ldconfig     &&

cd .. &&
rm -rf xinit-1.4.1
```

## TWM

The Tiny Window Manager.

```sh
curl https://www.x.org/pub/individual/app/twm-1.0.11.tar.xz -o /sources/xorg/twm-1.0.11.tar.xz &&

tar xvf /sources/xorg/twm-1.0.11.tar.xz &&
cd       twm-1.0.11                     &&

sed -i -e '/^rcdir =/s,^\(rcdir = \).*,\1/etc/X11/app-defaults,' src/Makefile.in &&
./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf twm-1.0.11
```

## xdg-utils

```sh
curl https://portland.freedesktop.org/download/xdg-utils-1.1.3.tar.gz -o /sources/xorg/xdg-utils-1.1.3.tar.gz &&

tar xzvf /sources/xorg/xdg-utils-1.1.3.tar.gz &&
cd        xdg-utils-1.1.3                     &&

./configure --prefix=/usr \
            --mandir=/usr/share/man &&

make              &&
sudo make install &&

cd .. &&
rm -rf xdg-utils-1.1.3
```

## Xdg-user-dirs

```sh
curl https://user-dirs.freedesktop.org/releases/xdg-user-dirs-0.17.tar.gz -o /sources/xorg/xdg-user-dirs-0.17.tar.gz &&

tar xzvf /sources/xorg/xdg-user-dirs-0.17.tar.gz &&
cd        xdg-user-dirs-0.17                     &&

./configure --prefix=/usr \
            --sysconfdir=/etc &&

make              &&
sudo make install &&

cd .. &&
rm -rf xdg-user-dirs-0.17
```

To create your user dirs:

```sh
xdg-user-dirs-update
```

## Additional Fonts

Create a separate directory for downloading these:

```sh
mkdir -pv /sources/fonts
```

### DejaVU Fonts

```sh
curl https://sourceforge.net/projects/dejavu/files/dejavu/2.37/dejavu-fonts-ttf-2.37.tar.bz2/download/ -o /sources/fonts/dejavu-fonts-ttf-2.37.tar.bz2 &&

tar xvf /sources/fonts/dejavu-fonts-ttf-2.37.tar.bz2 &&
cd       dejavu-fonts-ttf-2.37                       &&

sudo install -v -d -m755 /usr/share/fonts/dejavu        &&
sudo install -v -m644 ttf/*.ttf /usr/share/fonts/dejavu &&
sudo fc-cache -v /usr/share/fonts/dejavu                &&

cd .. &&
rm -rf dejavu-fonts-ttf-2.37
```

### Liberation Fonts

```sh
curl https://github.com/liberationfonts/liberation-fonts/files/6418984/liberation-fonts-ttf-2.1.4.tar.gz -o /sources/fonts/liberation-fonts-ttf-2.1.4.tar.gz &&

tar xzvf /sources/fonts/liberation-fonts-ttf-2.1.4.tar.gz &&
cd        liberation-fonts-ttf-2.1.4                      &&

sudo install -v -d -m755 /usr/share/fonts/liberation    &&
sudo install -v -m644 *.ttf /usr/share/fonts/liberation &&
sudo fc-cache -v /usr/share/fonts/liberation            &&

cd .. &&
rm -rf liberation-fonts-ttf-2.1.4
```

### Source Code Pro

```sh
curl https://github.com/adobe-fonts/source-code-pro/releases/download/2.038R-ro%2F1.058R-it%2F1.018R-VAR/TTF-source-code-pro-2.038R-ro-1.058R-it.zip -o /sources/fonts/TTF-source-code-pro-2.038R-ro-1.05R-it.zip &&

unzip -d source-code-pro /sources/fonts/TTF-source-code-pro-2.038R-ro-1.05R-it.zip &&
cd       source-code-pro                                                           &&

sudo install -v -d -m755 /usr/share/fonts/source-code-pro    &&
sudo install -v -m644 *.ttf /usr/share/fonts/source-code-pro &&
sudo fc-cache -v /usr/share/fonts/source-code-pro            &&

cd .. &&
rm -rf source-code-pro
```
