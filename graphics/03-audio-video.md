# Audio and video libraries

## fftw

Fast Fourier Transform, used by many audio and video processing libraries.

`fftw` builds libraries for single-precision, double-precision, and long double-precision (32-bit, 64-bit, and 128-bit) floating-point numbers, each separately.

```sh
curl http://fftw.org/fftw-3.3.9.tar.gz -o /sources/fftw-3.3.9.tar.gz &&

tar xzvf /sources/fftw-3.3.9.tar.gz &&
cd        fftw-3.3.9                &&

./configure --prefix=/usr    \
            --enable-shared  \
            --disable-static \
            --enable-threads \
            --enable-sse2    \
            --enable-avx &&

make          &&
make -k check &&
sudo make install

make clean &&
./configure --prefix=/usr    \
            --enable-shared  \
            --disable-static \
            --enable-threads \
            --enable-sse2    \
            --enable-avx     \
            --enable-float &&

make          &&
make -k check &&
sudo make install

make clean &&
./configure --prefix=/usr    \
            --enable-shared  \
            --disable-static \
            --enable-threads \
            --enable-long-double &&

make              &&
make -k check     &&
sudo make install &&

cd .. &&
rm -rf fftw-3.3.9
```

## Vala

Scripting language for GNOME.

Pass `--disable-valadoc` to `configure` if `GraphViz` is not installed.

```sh
curl https://download.gnome.org/sources/vala/0.52/vala-0.52.4.tar.xz -o /sources/vala-0.52.4.tar.xz &&

tar xvf /sources/vala-0.52.4.tar.xz &&
cd       vala-0.52.4 &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf vala-0.52.4
```

## librsvg

SVG library and toolset. For Hacker News street cred, written in Rust.

```sh
curl https://download.gnome.org/sources/librsvg/2.50/librsvg-2.50.7.tar.xz -o /sources/librsvg-2.50.7.tar.xz &&

tar xvf /sources/librsvg-2.50.7.tar.xz &&
cd       librsvg-2.50.7                &&

./configure --prefix=/usr    \
            --enable-vala    \
            --disable-static \
            --docdir=/usr/share/doc/librsvg-2.50.7 &&

make              &&
make -k check      &&
sudo make install &&

cd .. &&
rm -rf librsvg-2.50.7
```

## LAME

MP3 encoder. Build `libsndfile` first for optional audio sampling support

```sh
curl https://downloads.sourceforge.net/lame/lame-3.100.tar.gz -o /sources/lame-3.100.tar.gz &&

tar xzvf /sources/lame-3.100.tar.gz &&
cd        lame-3.100

./configure --prefix=/usr    \
            --enable-mp3rtp  \
            --disable-static \
            --docdir=/usr/share/doc/lame-3.100 &&

make                                                   &&
sudo make pkghtmldir=/usr/share/doc/lame-3.100 install &&

cd .. &&
rm -rf lame-3.100
```

## libogg

```sh
curl https://downloads.xiph.org/releases/ogg/libogg-1.3.5.tar.xz -o /sources/libogg-1.3.5.tar.xz &&

tar xvf /sources/libogg-1.3.5.tar.xz &&
cd       libogg-1.3.5                &&

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/libogg-1.3.4

make              &&
sudo make install &&

cd .. &&
rm -rf libogg-1.3.5
```

## libvorbis

```sh
curl https://downloads.xiph.org/releases/vorbis/libvorbis-1.3.7.tar.xz -o /sources/libvorbis-1.3.7.tar.xz &&

tar xvf /sources/libvorbis-1.3.7.tar.xz &&
cd       libvorbis-1.3.7

./configure --prefix=/usr \
            --disable-static &&

make                                                             &&
sudo make install                                                &&
sudo install -v -m644 doc/Vorbis* /usr/share/doc/libvorbis-1.3.7 &&

cd .. &&
rm -rf libvorbis-1.3.7
```

## libtheora

```sh
curl https://downloads.xiph.org/releases/theora/libtheora-1.1.1.tar.xz -o /sources/libtheora-1.1.1.tar.xz &&

tar xvf /sources/libtheora-1.1.1.tar.xz &&
cd       libtheora-1.1.1

sed -i 's/png_\(sizeof\)/\1/g' examples/png2theora.c &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libtheora-1.1.1
```

## fdk-aac

Franhaufer implementation of Advanced Audio Encoding.

```sh
curl https://downloads.sourceforge.net/opencore-amr/fdk-aac-2.0.1.tar.gz -o /sources/fdk-aac-2.0.1.tar.gz &&

tar xzvf /sources/fdk-aac-2.0.1.tar.gz &&
cd        fdk-aac-2.0.1                &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf fdk-aac-2.0.1
```

## Opus

Lossless audio encoding.

```sh
curl https://archive.mozilla.org/pub/opus/opus-1.3.1.tar.gz -o /sources/opus-1.3.1.tar.gz &&

tar xzvf /sources/opus-1.3.1.tar.gz &&
cd        opus-1.3.1

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/opus-1.3.1 &&

make              &&
make -k check     &&
sudo make install &&

cd .. &&
rm -rf opus-1.3.1
```

## libass

Advanced Substation Alpha/Substation Alpha subtitle formatting.

```sh
curl https://github.com/libass/libass/releases/download/0.15.0/libass-0.15.0.tar.xz -o /sources/libass-0.15.0.tar.xz &&

tar xvf /sources/libass-0.15.0.tar.xz &&
cd       libass-0.15.0

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libass-0.15.0
```

## x264

H.264 MPEG-4 video codec.

```sh
curl http://anduin.linuxfromscratch.org/BLFS/x264/x264-20210211.tar.xz -o /sources/x264-20210211.tar.xz &&

tar xvf /sources/x264-20210211.tar.xz &&
cd       x264-20210211                &&

./configure --prefix=/usr   \
            --enable-shared \
            --disable-cli &&

make              &&
sudo make install &&

cd .. &&
rm -rf x264-20210211
```

## x265

H.265 HEVC video codec.

```sh
curl http://anduin.linuxfromscratch.org/BLFS/x265/x265_3.4.tar.gz -o /sources/x265_3.4.tar.gz &&

tar xzvf /sources/x265_3.4.tar.gz &&
cd        x265_3.4                &&

mkdir bld &&
cd    bld &&

cmake -DCMAKE_INSTALL_PREFIX=/usr ../source &&

make                           &&
sudo make install              &&
sudo rm -vf /usr/lib/libx265.a &&

cd ../.. &&
rm -rf x265_3.4
```

## libvpx

HTML5 video encoding.

```sh
curl https://github.com/webmproject/libvpx/archive/v1.9.0/libvpx-1.9.0.tar.gz -o /sources/libvpx-1.9.0.tar.gz &&

tar xzvf /sources/libvpx-1.9.0.tar.gz &&
cd        libvpx-1.9.0

sed -i 's/cp -p/cp/' build/make/Makefile &&
mkdir libvpx-build            &&
cd    libvpx-build            &&
../configure --prefix=/usr    \
             --enable-shared  \
             --disable-static &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf libvpx-1.9.0
```

## ALSA

Advanced Linux Sound Architecture.

### Kernel configuration

Ensure you have the following config settings.

```
Device Drivers --->
  <*/M> Sound card support --->                  [CONFIG_SOUND]
    <*/M> Advanced Linux Sound Architecture ---> [CONFIG_SND]
            Select settings and drivers appropriate for your hardware.
```

If you intend to use Bluetooth, you will also need the following:

```
General Setup --->
  [*] Configure standard kernel features (expert users)               [CONFIG_EXPERT]
    [*] Enable timerfd() system call               [CONFIG_TIMERFD]
    [*] Enable eventfd() system call               [CONFIG_EVENTFD]

[*] Networking support --->                [CONFIG_NET]
  <*/M> Bluetooth subsystem support --->    [CONFIG_BT]
    <*/M> RFCOMM protocol support          [CONFIG_BT_RFCOMM]
    [*]   RFCOMM TTY support               [CONFIG_BT_RFCOMM_TTY]
    <*/M> BNEP protocol support            [CONFIG_BT_BNEP]
    [*]   Multicast filter support         [CONFIG_BT_BNEP_MC_FILTER]
    [*]   Protocol filter support          [CONFIG_BT_BNEP_PROTO_FILTER]
    <*/M> HIDP protocol support            [CONFIG_BT_HIDP]
        Bluetooth device drivers --->
          (Select the appropriate drivers for your Bluetooth hardware)

   <*/M> RF switch subsystem support ----   [CONFIG_RFKILL]

-*- Cryptographic API --->
   <*/M*> User-space cryptographic algorithm configuration         [CONFIG_CRYPTO_USER]
   <*/M*> User-space interface for hash algorithms                 [CONFIG_CRYPTO_USER_API_HASH]
   <*/M*> User-space interface for symmetric key cipher algorithms [CONFIG_CRYPTO_USER_API_SKCIPHER]
   <*/M*> MD5 digest algorithm                                     [CONFIG_CRYPTO_MD5]
   <*/M*> SHA1 digest algorithm                                    [CONFIG_CRYPTO_SHA1]

Security Options --->
  <*/M*> Diffie-Hellman operations on retained keys [CONFIG_KEY_DH_OPERATIONS]
```

### alsa-lib

```sh
curl https://www.alsa-project.org/files/pub/lib/alsa-lib-1.2.5.tar.bz2 -o /sources/alsa-lib-1.2.5.tar.bz2 &&

tar xvf /sources/alsa-lib-1.2.5.tar.bz2 &&
cd       alsa-lib-1.2.5                 &&

./configure &&

make              &&
make doc          &&
sudo make install &&

sudo install -v -d -m755 /usr/share/doc/alsa-lib-1.2.5/html/search &&
sudo install -v -m644 doc/doxygen/html/*.* \
                /usr/share/doc/alsa-lib-1.2.5/html                 &&
sudo install -v -m644 doc/doxygen/html/search/* \
                /usr/share/doc/alsa-lib-1.2.5/html/search          &&

cd .. &&
rm -rf alsa-lib-1.2.5
```

### FLAC

Similar to mp3 encoding, but lossless.

```sh
curl https://downloads.xiph.org/releases/flac/flac-1.3.3.tar.xz -o /sources/flac-1.3.3.tar.xz &&
curl https://www.linuxfromscratch.org/patches/blfs/svn/flac-1.3.3-security_fixes-1.patch -o /sources/flac-1.3.3-security_fixes-1.patch &&

tar xvf /sources/flac-1.3.3.tar.xz &&
cd       flac-1.3.3                &&

patch -Np1 -i /sources/flac-1.3.3-security_fixes-1.patch &&

./configure --prefix=/usr            \
            --disable-thorough-tests \
            --docdir=/usr/share/doc/flac-1.3.3 &&

make              &&
sudo make install &&

cd .. &&
rm -rf flac-1.3.3
```

### libsndfile

```sh
curl https://github.com/libsndfile/libsndfile/releases/download/1.0.31/libsndfile-1.0.31.tar.bz2 -o /sources/libsndfile-1.0.31.tar.bz2 &&

tar xvf /sources/libsndfile-1.0.31.tar.bz2 &&
cd       libsndfile-1.0.31                 &&

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/libsndfile-1.0.31 &&

make              &&
sudo make install &&

cd .. &&
rm -rf libsndfile-1.0.31
```

### libsamplerate

```sh
curl https://github.com/libsndfile/libsamplerate/releases/download/0.2.1/libsamplerate-0.2.1.tar.bz2 -o /sources/libsamplerate-0.2.1.tar.bz2 &&

tar xvf /sources/libsamplerate-0.2.1.tar.bz2 &&
cd       libsamplerate-0.2.1                 &&

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/libsamplerate-0.2.1 &&

make              &&
make -k check     &&
sudo make install &&

cd .. &&
rm -rf libsamplerate-0.2.1
```

### speex

Specialized audio codec for handling speech.

```sh
curl https://downloads.xiph.org/releases/speex/speex-1.2.0.tar.gz -o /sources/speex-1.2.0.tar.gz &&
curl https://downloads.xiph.org/releases/speex/speexdsp-1.2.0.tar.gz -o /sources/speexdsp-1.2.0.tar.gz &&

tar xzvf /sources/speex-1.2.0.tar.gz &&
cd        speex-1.2.0                &&

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/speex-1.2.0 &&

make              &&
sudo make install &&

cd ..              &&
rm -rf speex-1.2.0 &&

tar xzvf /sources/speexdsp-1.2.0.tar.gz &&
cd        speexdsp-1.2.0                &&

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/speexdsp-1.2.0 &&

make              &&
sudo make install &&

cd .. &&
rm -rf speexdsp-1.2.0
```

### libical

iCalendar format libraries.

```sh
curl https://github.com/libical/libical/releases/download/v3.0.10/libical-3.0.10.tar.gz -o /sources/libical-3.0.10.tar.gz &&

tar xzvf /sources/libical-3.0.10.tar.gz &&
cd        libical-3.0.10                &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr  \
      -DCMAKE_BUILD_TYPE=Release   \
      -DSHARED_ONLY=yes            \
      -DICAL_BUILD_DOCS=false      \
      -DGOBJECT_INTROSPECTION=true \
      -DICAL_GLIB_VAPI=true        \
      .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf libical-3.0.10
```

### BlueZ

The Linux bluetooth stack.

```sh
curl https://www.kernel.org/pub/linux/bluetooth/bluez-5.58.tar.xz -o /sources/bluez-5.58.tar.xz &&

tar xvf /sources/bluez-5.58.tar.xz &&
cd       bluez-5.58                &&

./configure --prefix=/usr         \
            --sysconfdir=/etc     \
            --localstatedir=/var  \
            --enable-library      &&

make                                                         &&
sudo make install                                            &&
sudo ln -svf ../libexec/bluetooth/bluetoothd /usr/sbin       &&
sudo install -v -dm755 /etc/bluetooth                        &&
sudo install -v -m644 src/main.conf /etc/bluetooth/main.conf &&
sudo install -v -dm755 /usr/share/doc/bluez-5.58             &&
sudo install -v -m644 doc/*.txt /usr/share/doc/bluez-5.58    &&

cd .. &&
rm -rf bluez-5.58
```

### v4l-utils

Camera file handling.

```sh
curl https://www.linuxtv.org/downloads/v4l-utils/v4l-utils-1.20.0.tar.bz2 -o /sources/v4l-utils-1.20.0.tar.bz2 &&
curl https://www.linuxfromscratch.org/patches/blfs/svn/v4l-utils-1.20.0-upstream_fixes-1.patch -o /sources/patches/v4l-utils-1.20.0-upstream_fixes-1.patch &&

tar xvf /sources/v4l-utils-1.20.0.tar.bz2 &&
cd       v4l-utils-1.20.0                 &&

patch -Np1 -i /sources/patches/v4l-utils-1.20.0-upstream_fixes-1.patch &&

./configure --prefix=/usr     \
            --sysconfdir=/etc \
            --disable-static  &&

make              &&
sudo make install &&

cd .. &&
rm -rf v4l-utils-1.20.0
```

### PulseAudio

```sh
curl https://www.freedesktop.org/software/pulseaudio/releases/pulseaudio-14.2.tar.xz -o /sources/pulseaudio-14.2.tar.xz &&

tar xvf /sources/pulseaudio-14.2.tar.xz &&
cd       pulseaudio-14.2                &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Ddatabase=gdbm     \
      -Dbluez5=false &&

ninja                                                   &&
sudo ninja install                                      &&
sudo rm -fv /etc/dbus-1/system.d/pulseaudio-system.conf &&

cd ../.. &&
rm -rf pulseaudio-14.2
```

### alsa-plugins

```sh
curl https://www.alsa-project.org/files/pub/plugins/alsa-plugins-1.2.5.tar.bz2 -o /sources/alsa-plugins-1.2.5.tar.bz2 &&

tar xvf /sources/alsa-plugins-1.2.5.tar.bz2 &&
cd       alsa-plugins-1.2.5                 &&

./configure --sysconfdir=/etc &&

make              &&
sudo make install &&

cd .. &&
rm -rf alsa-plugins-1.2.5
```

### alsa-utils

```sh
curl https://www.alsa-project.org/files/pub/utils/alsa-utils-1.2.5.tar.bz2 -o /sources/alsa-utils-1.2.5.tar.bz2 &&

tar xvf /sources/alsa-utils-1.2.5.tar.bz2 &&
cd       alsa-utils-1.2.5                 &&

./configure --disable-alsaconf \
            --disable-bat   \
            --disable-xmlto \
            --with-curses=ncursesw &&

make              &&
sudo make install &&

cd .. &&
rm -rf alsa-utils-1.2.5
```

### alsa-oss

```sh
curl https://www.alsa-project.org/files/pub/oss-lib/alsa-oss-1.1.8.tar.bz2 -o /sources/alsa-oss-1.1.8.tar.bz2 &&

tar xvf /sources/alsa-oss-1.1.8.tar.bz2 &&
cd       alsa-oss-1.1.8                 &&

./configure --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf alsa-oss-1.1.8
```

## OpenCV

Libraries for computer vision.

```sh
curl https://github.com/opencv/opencv/archive/4.5.2/opencv-4.5.2.tar.gz -o /sources/opencv-4.5.2.tar.gz &&
curl https://github.com/opencv/opencv_contrib/archive/4.5.2/opencv_contrib-4.5.2.tar.gz -o /sources/opencv_contrib-4.5.2.tar.gz &&

tar xzvf /sources/opencv-4.5.2.tar.gz &&
cd        opencv-4.5.2                &&

tar xzvf /sources/opencv_contrib-4.5.2.tar.gz &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr      \
      -DCMAKE_BUILD_TYPE=Release       \
      -DENABLE_CXX11=ON                \
      -DBUILD_PERF_TESTS=OFF           \
      -DWITH_XINE=ON                   \
      -DBUILD_TESTS=OFF                \
      -DENABLE_PRECOMPILED_HEADERS=OFF \
      -DCMAKE_SKIP_RPATH=ON            \
      -DBUILD_WITH_DEBUG_INFO=OFF      \
      -Wno-dev .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf opencv-4.5.2
```

## Python OpenCV

Python bindings for `OpenCV`.

```sh
curl https://github.com/opencv/opencv-python/archive/refs/tags/54.tar.gz -o /sources/python-opencv-4.5.2.54.tar.gz &&

tar xzvf /sources/python-opencv-4.5.2.54.tar.gz &&
cd        opencv-python-54               &&


```

## FFmpeg

A huge suite of CLI tools for manipulating audio and video files. If you wish to build the documentation, `GhostScript` and `doxygen` are required.

```sh
curl https://ffmpeg.org/releases/ffmpeg-4.4.tar.xz -o /sources/ffmpeg-4.4.tar.xz &&

tar xvf /sources/ffmpeg-4.4.tar.xz &&
cd       ffmpeg-4.4

sed -i 's/-lflite"/-lflite -lasound"/' configure &&

./configure --prefix=/usr        \
            --enable-gpl         \
            --enable-version3    \
            --enable-nonfree     \
            --disable-static     \
            --enable-shared      \
            --disable-debug      \
            --enable-avresample  \
            --enable-libass      \
            --enable-libfdk-aac  \
            --enable-libfreetype \
            --enable-libmp3lame  \
            --enable-libopus     \
            --enable-libtheora   \
            --enable-libvorbis   \
            --enable-libvpx      \
            --enable-libx264     \
            --enable-libx265     \
            --enable-openssl     \
            --docdir=/usr/share/doc/ffmpeg-4.4 &&

make                                           &&
gcc tools/qt-faststart.c -o tools/qt-faststart &&

doxygen doc/Doxyfile &&

sudo make install                                                         &&
sudo install -v -m755    tools/qt-faststart /usr/bin                      &&
sudo install -v -m755 -d           /usr/share/doc/ffmpeg-4.4              &&
sudo install -v -m644    doc/*.txt /usr/share/doc/ffmpeg-4.4              &&
sudo install -v -m755 -d /usr/share/doc/ffmpeg-4.4/api                    &&
sudo cp -vr doc/doxy/html/* /usr/share/doc/ffmpeg-4.4/api                 &&
find /usr/share/doc/ffmpeg-4.4/api -type f -exec sudo chmod -c 0644 {} \; &&
find /usr/share/doc/ffmpeg-4.4/api -type d -exec sudo chmod -c 0755 {} \; &&

cd .. &&
rm -rf ffmpeg-4.4
```

### Pipewire

```sh
curl https://github.com/PipeWire/pipewire/archive/0.3.30/pipewire-0.3.30.tar.gz -o /sources/pipewire-0.3.30.tar.gz &&

tar xzvf /sources/pipewire-0.3.30.tar.gz &&
cd        pipewire-0.3.30                &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf pipewire-0.3.30
```

## gstreamer

```sh
curl https://gstreamer.freedesktop.org/src/gstreamer/gstreamer-1.18.4.tar.xz -o /sources/gstreamer-1.18.4.tar.xz &&

tar xvf /sources/gstreamer-1.18.4.tar.xz &&
cd        gstreamer-1.18.4               &&

mkdir build &&
cd    build &&

meson  --prefix=/usr                                                    \
       --buildtype=release                                              \
       -Dgst_debug=false                                                \
       -Dpackage-origin=https://www.linuxfromscratch.org/blfs/view/svn/ \
       -Dpackage-name="GStreamer 1.18.4 BLFS" &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gstreamer-1.18.4
```

## graphene

```sh
curl https://github.com/ebassi/graphene/releases/download/1.10.6/graphene-1.10.6.tar.xz -o /sources/graphene-1.10.6.tar.xz &&

tar xvf /sources/graphene-1.10.6.tar.xz &&
cd       graphene-1.10.6                &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf graphene-1.10.6
```

## gst-plugins-base

```sh
curl https://gstreamer.freedesktop.org/src/gst-plugins-base/gst-plugins-base-1.18.4.tar.xz -o /sources/gst-plugins-base-1.18.4.tar.xz &&

tar xvf /sources/gst-plugins-base-1.18.4.tar.xz &&
cd       gst-plugins-base-1.18.4                &&

sed -i 's|implicit_include_directories : false||' gst-libs/gst/gl/meson.build &&

mkdir build &&
cd    build &&

meson  --prefix=/usr       \
       --buildtype=release \
       -Dpackage-origin=https://www.linuxfromscratch.org/blfs/view/svn/ \
       -Dpackage-name="GStreamer 1.18.4 BLFS"    \
       --wrap-mode=nodownload &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gst-plugins-base-1.18.4
```

## gst-plugins-bad

```sh
curl https://gstreamer.freedesktop.org/src/gst-plugins-bad/gst-plugins-bad-1.18.4.tar.xz -o /sources/gst-plugins-bad-1.18.4.tar.xz &&

tar xvf /sources/gst-plugins-bad-1.18.4.tar.xz &&
cd       gst-plugins-bad-1.18.4                &&

mkdir build &&
cd    build &&

meson  --prefix=/usr       \
       --buildtype=release \
       -Dpackage-origin=https://www.linuxfromscratch.org/blfs/view/svn/ \
       -Dpackage-name="GStreamer 1.18.4 BLFS" &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gst-plugins-bad-1.18.4
```

## gst-plugins-good

```sh
curl https://gstreamer.freedesktop.org/src/gst-plugins-good/gst-plugins-good-1.18.4.tar.xz -o /sources/gst-plugins-good-1.18.4.tar.xz &&

tar xvf /sources/gst-plugins-good-1.18.4.tar.xz &&
cd       gst-plugins-good-1.18.4                &&

mkdir build &&
cd    build &&

meson  --prefix=/usr       \
       --buildtype=release \
       -Dpackage-origin=https://www.linuxfromscratch.org/blfs/view/svn/ \
       -Dpackage-name="GStreamer 1.18.4 BLFS" &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gst-plugins-good-1.18.4
```

## gst-libav

```sh
curl https://gstreamer.freedesktop.org/src/gst-libav/gst-libav-1.18.4.tar.xz -o /sources/gst-libav-1.18.4.tar.xz &&

tar xvf /sources/gst-libav-1.18.4.tar.xz &&
cd       gst-libav-1.18.4                &&

mkdir build &&
cd    build &&

meson  --prefix=/usr       \
       --buildtype=release \
       -Dpackage-origin=https://www.linuxfromscratch.org/blfs/view/svn/ \
       -Dpackage-name="GStreamer 1.18.4 BLFS" &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gst-libav-1.18.4
```

## libcanberra

Library for generating desktop sounds.

```sh
curl http://0pointer.de/lennart/projects/libcanberra/libcanberra-0.30.tar.xz -o /sources/libcanberra-0.30.tar.xz &&
curl https://www.linuxfromscratch.org/patches/blfs/svn/libcanberra-0.30-wayland-1.patch -o /sources/patches/libcanberra-0.30-wayland-1.patch &&

tar xvf /sources/libcanberra-0.30.tar.xz &&
cd       libcanberra-0.30                &&

patch -Np1 -i /sources/patches/libcanberra-0.30-wayland-1.patch &&

./configure --prefix=/usr \
            --disable-gtk \
            --disable-oss &&

make              &&
sudo make install &&

cd .. &&
rm -rf libcanberra-0.30
```

## djbfft

Library for floating-point convolutions that claims to hold speed records for fft on general-purpose computers.

```sh
curl https://cr.yp.to/djbfft/djbfft-0.76.tar.gz -o /sources/djbfft-0.76.tar.gz &&

tar xzvf /sources/djbfft-0.76.tar.gz &&
cd        djbfft-0.76                &&

sed -i 's/\/usr\/local\/djbfft/\/usr/' conf-home         &&
sed -i 's/-O1/-march=native -O2/'      conf-cc           &&
sed -i 's/extern int errno;/#include <errno.h>/' error.h &&

make                  &&
sudo make setup check &&

cd .. &&
rm -rf djbfft-0.76
```

## liba52

Library for decoding ATSC A/52 streams.

```sh
curl http://liba52.sourceforge.net/files/a52dec-0.7.4.tar.gz -o /sources/a52dec-0.7.4.tar.gz &&

tar xzvf /sources/a52dec-0.7.4.tar.gz &&
cd        a52dec-0.7.4                &&

./configure --prefix=/usr \
            --mandir=/usr/share/man \
            --enable-shared \
            --disable-static \
            CFLAGS="${CFLAGS:--g -O2 -fPIC}" &&

make              &&
sudo make install &&

sudo cp liba52/a52_internal.h /usr/include/a52dec &&
sudo install -v -m644 -D doc/liba52.txt \
    /usr/share/doc/liba52-0.7.4/liba52.txt

cd .. &&
rm -rf a52dec-0.7.4
```

## libmad

24-bit MPEG audio decoder.

```sh
curl https://downloads.sourceforge.net/mad/libmad-0.15.1b.tar.gz -o /sources/libmad-0.15.1b.tar.gz &&
curl https://www.linuxfromscratch.org/patches/blfs/svn/libmad-0.15.1b-fixes-1.patch -o /sources/patches/libmad-0.15.1b-fixes-1.patch &&

tar xzvf /sources/libmad-0.15.1b.tar.gz &&
cd        libmad-0.15.1b                &&

patch -Np1 -i /sources/patches/libmad-0.15.1b-fixes-1.patch  &&
sed "s@AM_CONFIG_HEADER@AC_CONFIG_HEADERS@g" -i configure.ac &&
touch NEWS AUTHORS ChangeLog                                 &&
autoreconf -fi                                               &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libmad-0.15.1b
```

This package does not come with a `pkg-config` manifest, so we create one:

```sh
sudo su -
cat > /usr/lib/pkgconfig/mad.pc << "EOF"
prefix=/usr
exec_prefix=${prefix}
libdir=${exec_prefix}/lib
includedir=${prefix}/include

Name: mad
Description: MPEG audio decoder
Requires:
Version: 0.15.1b
Libs: -L${libdir} -lmad
Cflags: -I${includedir}
EOF
exit
```
