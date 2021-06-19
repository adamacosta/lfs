# Multimedia tools

There is no clear dependency ordering between the various libraries and tools presented here and the programming languages presented in the next section. You may need to jump back and forth quite a bit. A lot of this is probably not needed if you only intend to build a headless system, but if you wish to use it as a build server and include building of documentation, many documentation generation tools can do quite a bit more if they include support for graphics, even if the graphics cannot be viewed on the same machine.

There is an additional chicken and egg problem here in that `Cairo` will support `OpenGL` and `X11` if they are present, but can support its own graphics devices without needing an X server. You may want to recompile `Cairo` and `Pango` after installing `Xorg` and the `GTK` libraries for `GNOME`, but I present them first. `GraphViz` and `ImageMagick` will also add graphical features if recompiled after `Xorg` and `GTK`.

## Fonts and typesetting

### FreeType and deps

You may wish to install `brotli` first, which provides implementations of various compression and encoding algorithms used by some of these other libraries.

`FreeType` also depends upon `libpng`, which you will want to build first. You will also need to build `HarfBuzz` after building `FreeType`, then rebuild and reinstall `FreeType`.

#### Graphite2

This is not strictly a dependency of `FreeType` itself, but `HarfBuzz` may not behave as expected if it is not present. Note that `Graphite` also has circular dependencies on `FreeType` and `HarfBuzz`, so though it will build without them, you will also want to rebuild after.

```sh
wget https://github.com/silnrsi/graphite/releases/download/1.3.14/graphite2-1.3.14.tgz -P /sources &&

tar xzvf /sources/graphite2-1.3.14.tgz &&
cd        graphite2-1.3.14             &&

sed -i '/cmptest/d' tests/CMakeLists.txt &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf graphite2-1.3.14
```

#### brotli

```sh
wget https://github.com/google/brotli/archive/v1.0.9/brotli-1.0.9.tar.gz -P /sources &&

tar xzvf /sources/brotli-1.0.9.tar.gz &&
cd        brotli-1.0.9                &&

sed -i 's@-R..libdir.@@' scripts/*.pc.in &&

mkdir out &&
cd    out &&

cmake -DCMAKE_INSTALL_PREFIX=/usr \
      -DCMAKE_BUILD_TYPE=Release  \
      .. &&

make                                      &&
make test                                 &&
pushd ..                                  &&
python setup.py build                     &&
popd                                      &&
sudo make install                         &&
pushd ..                                  &&
sudo python setup.py install --optimize=1 &&
popd                                      &&

cd ../.. &&
sudo rm -rf brotli-1.0.9
```

#### FreeType

```sh
wget https://downloads.sourceforge.net/freetype/freetype-2.10.4.tar.xz -P /sources &&
wget https://downloads.sourceforge.net/freetype/freetype-doc-2.10.4.tar.xz -P /sources &&

tar xvf /sources/freetype-2.10.4.tar.xz &&
tar xvf /sources/freetype-doc-2.10.4.tar.xz \
    --strip-components=2 \
    -C freetype-2.10.4/docs             &&
cd freetype-2.10.4                      &&


sed -ri "s:.*(AUX_MODULES.*valid):\1:" modules.cfg &&
sed -r "s:.*(#.*SUBPIXEL_RENDERING) .*:\1:" \
    -i include/freetype/config/ftoption.h          &&

./configure --prefix=/usr            \
            --enable-freetype-config \
            --disable-static &&

make                                                        &&
sudo make install                                           &&
sudo install -v -m755 -d /usr/share/doc/freetype-2.10.4     &&
sudo cp -v -R docs/*     /usr/share/doc/freetype-2.10.4     &&
sudo rm -v /usr/share/doc/freetype-2.10.4/freetype-config.1 &&

cd .. &&
rm -rf freetype-2.10.4
```

#### HarfBuzz

```sh
wget https://github.com/harfbuzz/harfbuzz/releases/download/2.8.1/harfbuzz-2.8.1.tar.xz -P /sources &&

tar xvf /sources/harfbuzz-2.8.1.tar.xz &&
cd      harfbuzz-2.8.1                 &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      -Dgraphite=enabled \
      -Dbenchmark=disabled &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf harfbuzz-2.8.1
```

After `HarfBuzz` is installed, you should now rebuild and reinstall `Graphite2` and `FreeType`.

## Image libraries

### libpng

```sh
wget https://downloads.sourceforge.net/libpng/libpng-1.6.37.tar.xz -P /sources &&
wget https://downloads.sourceforge.net/sourceforge/libpng-apng/libpng-1.6.37-apng.patch.gz -P /sources/patches &&

tar xvf /sources/libpng-1.6.37.tar.xz         &&
gzip -d /sources/libpng-1.6.37-apng.patch.gz  &&
cd       libpng-1.6.37                        &&

cat /sources/libpng-1.6.37-apng.patch | patch -p1

./configure --prefix=/usr \
            --disable-static &&

make                                                             &&
make -k check                                                    &&
sudo make install                                                &&
sudo install -vdm755 /usr/share/doc/libpng-1.6.37                &&
sudo cp -v README libpng-manual.txt /usr/share/doc/libpng-1.6.37 &&

cd .. &
rm -rf libpng-1.6.37
```

### OpenJPEG

```sh
wget https://github.com/uclouvain/openjpeg/archive/v2.4.0/openjpeg-2.4.0.tar.gz -P /sources &&

tar xzvf /sources/openjpeg-2.4.0.tar.gz &&
cd        openjpeg-2.4.0                &&

mkdir -v build &&
cd       build &&

cmake -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_INSTALL_PREFIX=/usr \
      -DBUILD_STATIC_LIBS=OFF .. &&

make              &&
sudo make install &&
pushd ../doc      &&
    for man in man/man?/* ; do
        sudo install -v -D -m 644 $man /usr/share/$man
    done
popd              &&

cd ../.. &&
rm -rf openjpeg-2.4.0
```

### libjpeg-turbo

```sh
wget https://downloads.sourceforge.net/libjpeg-turbo/libjpeg-turbo-2.0.6.tar.gz -P /sources &&

tar xzvf /sources/libjpeg-turbo-2.0.6.tar.gz &&
cd        libjpeg-turbo-2.0.6                &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr \
      -DCMAKE_BUILD_TYPE=RELEASE  \
      -DENABLE_STATIC=FALSE       \
      -DCMAKE_INSTALL_DOCDIR=/usr/share/doc/libjpeg-turbo-2.0.6 \
      -DCMAKE_INSTALL_DEFAULT_LIBDIR=lib  \
      .. &&

make              &&
make test         &&
sudo make install &&

cd ../.. &&
rm -rf libjpeg-turbo-2.0.6
```

### libTIFF

```sh
wget http://download.osgeo.org/libtiff/tiff-4.2.0.tar.gz -P /sources &&

tar xzvf /sources/tiff-4.2.0.tar.gz &&
cd        tiff-4.2.0                &&

mkdir -p libtiff-build &&
cd       libtiff-build &&

cmake -DCMAKE_INSTALL_DOCDIR=/usr/share/doc/libtiff-4.2.0 \
      -DCMAKE_INSTALL_PREFIX=/usr -G Ninja .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf tiff-4.2.0
```

### gdk-pixbuf

Pixel buffer library divested from GTK+.

```sh
wget https://download.gnome.org/sources/gdk-pixbuf/2.42/gdk-pixbuf-2.42.6.tar.xz -P /sources &&

tar xvf /sources/gdk-pixbuf-2.42.6.tar.xz &&
cd       gdk-pixbuf-2.42.6                &&

mkdir build &&
cd build    &&

meson --prefix=/usr .. &&

ninja              &&
ninja test         &&
sudo ninja install &&

cd ../.. &&
rm -rf gdk-pixbuf-2.42.6
```

### Pixman

Libraries for low-level pixel manipulation.

```sh
wget https://www.cairographics.org/releases/pixman-0.40.0.tar.gz -P /sources &&

tar xvf /sources/pixman-0.40.0.tar.xz &&
cd       pixman-0.40.0                &&

mkdir build &&
cd    build &&

meson --prefix=/usr &&

ninja              &&
ninja test         &&
sudo ninja install &&

cd ../.. &&
rm -rf pixman-0.40.0
```

### giflib

```sh
wget https://sourceforge.net/projects/giflib/files/giflib-5.2.1.tar.gz -P /sources &&

tar xzvf /sources/giflib-5.2.1.tar.gz &&
cd        giflib-5.2.1                &&

make                          &&
sudo make PREFIX=/usr install &&

sudo rm -fv /usr/lib/libgif.a                      &&
sudo find doc \( -name Makefile\* -o -name \*.1 \
         -o -name \*.xml \) -exec rm -v {} \;      &&
sudo install -v -dm755 /usr/share/doc/giflib-5.2.1 &&
sudo cp -v -R doc/* /usr/share/doc/giflib-5.2.1    &&

cd .. &&
rm -rf giflib-5.2.1
```

## libexif

```sh
wget https://github.com/libexif/libexif/releases/download/libexif-0_6_22-release/libexif-0.6.22.tar.xz -P /sources &&
wget https://www.linuxfromscratch.org/patches/blfs/svn/libexif-0.6.22-security_fixes-1.patch -P /sources/patches &&

tar xvf /sources/libexif-0.6.22.tar.xz &&
cd       libexif-0.6.22                &&

patch -Np1 -i /sources/patches/libexif-0.6.22-security_fixes-1.patch &&

./configure --prefix=/usr    \
            --disable-static \
            --with-doc-dir=/usr/share/doc/libexif-0.6.22 &&

make              &&
sudo make install &&

cd .. &&
rm -rf libexif-0.6.22
```

## taglib

```sh
wget https://taglib.github.io/releases/taglib-1.12.tar.gz -P /sources &&

tar xzvf /sources/taglib-1.12.tar.gz &&
cd        taglib-1.12                &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr \
      -DCMAKE_BUILD_TYPE=Release  \
      -DBUILD_SHARED_LIBS=ON \
      .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf tablib-1.12
```

## FontConfig

```sh
wget https://www.freedesktop.org/software/fontconfig/release/fontconfig-2.13.1.tar.bz2 -P /sources &&

tar xvf /sources/fontconfig-2.13.1.tar.bz2 &&
cd       fontconfig-2.13.1                 &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-docs       \
            --docdir=/usr/share/doc/fontconfig-2.13.1 &&

make              &&
sudo make install &&

cd .. &&
rm -rf fontconfig-2.13.1
```

## Cairo

2d graphics library. If you come to rebuild after installing `Xorg` and `OpenGL`, add the `--enable-gl` flag. `X11` libraries will be detected and enabled automatically.

```sh
wget https://www.cairographics.org/snapshots/cairo-1.17.4.tar.xz -P /sources &&

tar xvf /sources/cairo-1.17.4.tar.xz &&
cd       cairo-1.17.4                &&

autoreconf -fv           &&
./configure --prefix=/usr    \
            --disable-static \
            --enable-tee &&

make              &&
sudo make install &&

cd .. &&
rm -rf cairo-1.17.4
```

## FriBidi

Unicode Bidirectional Algorithm for bidirectional alphabet rendering.

```sh
wget https://github.com/fribidi/fribidi/releases/download/v1.0.9/fribidi-1.0.9.tar.xz -P /sources &&

tar xvf /sources/fribidi-1.0.9.tar.xz &&
cd       fribidi-1.0.9                &&

mkdir build &&
cd    build &&

meson --prefix=/usr .. &&

ninja
sudo ninja install &&

cd ../.. &&
rm -rf fribidi-1.0.9
```

## Pango

Text rendering and internationalization.

```sh
wget https://download.gnome.org/sources/pango/1.48/pango-1.48.5.tar.xz -P /sources &&

tar xvf /sources/pango-1.48.5.tar.xz &&
cd       pango-1.48.5                &&

mkdir build &&
cd    build &&

meson --prefix=/usr .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf pango-1.48.5
```

## GObject introspection

Inspect object APIs. Like `GLib`, is a critical component of `GNOME` but also used by other tools.

```sh
wget https://download.gnome.org/sources/gobject-introspection/1.68/gobject-introspection-1.68.0.tar.xz -P /sources &&

tar xvf /sources/gobject-introspection-1.68.0.tar.xz &&
cd       gobject-introspection-1.68.0                &&

mkdir build &&
cd    build &&

meson --prefix=/usr .. &&

ninja &&

python -m venv testenv   &&
. ./testenv/bin/activate &&
pip install markdown     &&

ninja test -k0 &&

deactivate &&

sudo ninja install &&

cd ../.. &&
rm -rf gobject-introspection-1.68.0
```

## SDL2

```sh
wget https://www.libsdl.org/release/SDL2-2.0.14.tar.gz -P /sources &&

tar xzvf /sources/SDL2-2.0.14.tar.gz &&
cd        SDL2-2.0.14                &&

./configure --prefix=/usr &&

make &&

pushd docs &&
  doxygen  &&
popd       &&

sudo make install              &&
sudo rm -v /usr/lib/libSDL2*.a &&

sudo install -v -m755 -d        /usr/share/doc/SDL2-2.0.14/html &&
sudo cp -Rv  docs/output/html/* /usr/share/doc/SDL2-2.0.14/html &&

cd .. &&
rm -rf SDL2-2.0.14
```

## SDL

```sh
wget https://www.libsdl.org/release/SDL-1.2.15.tar.gz -P /sources &&

tar xzvf /sources/SDL-1.2.15.tar.gz &&
cd        SDL-1.2.15                &&

sed -e '/_XData32/s:register long:register _Xconst long:' \
    -i src/video/x11/SDL_x11sym.h &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

sudo install -v -m755 -d /usr/share/doc/SDL-1.2.15/html &&
sudo install -v -m644    docs/html/*.html \
                         /usr/share/doc/SDL-1.2.15/html &&

cd .. &&
rm -rf SDL-1.2.15
```

## SDL Image

```sh
wget https://www.libsdl.org/projects/SDL_image/release/SDL_image-1.2.12.tar.gz -P /sources &&

tar xzvf /sources/SDL_image-1.2.12.tar.gz &&
cd        SDL_image-1.2.12                &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf SDL_image-1.2.12
```

## libwebp

```sh
wget http://downloads.webmproject.org/releases/webp/libwebp-1.2.0.tar.gz -P /sources &&

tar xzvf /sources/libwebp-1.2.0.tar.gz &&
cd        libwebp-1.2.0                &&

./configure --prefix=/usr           \
            --enable-libwebpmux     \
            --enable-libwebpdemux   \
            --enable-libwebpdecoder \
            --enable-libwebpextras  \
            --enable-swap-16bit-csp \
            --disable-static        &&

make              &&
sudo make install &&

cd .. &&
rm -rf libwebp-1.2.0
```

## woff

```sh
wget https://github.com/google/woff2/archive/v1.0.2/woff2-1.0.2.tar.gz -P /sources &&

tar xzvf /sources/woff2-1.0.2.tar.gz &&
cd        woff2-1.0.2                &&

mkdir out &&
cd    out &&

cmake -DCMAKE_INSTALL_PREFIX=/usr \
      -DCMAKE_BUILD_TYPE=Release .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf woff2-1.0.2
```

## Poppler

PDF rendering library.

`clang` is optional but required for the fuzzing and address sanitizers to work, which may help prevent security issues.

```sh
wget https://poppler.freedesktop.org/poppler-21.05.0.tar.xz -P /sources &&

tar xvf /sources/poppler-21.05.0.tar.xz &&
cd       poppler-21.05.0                &&

mkdir -v build &&
cd       build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr                      \
      -DCMAKE_BUILD_TYPE=release                       \
      -DBUILD_GTK_TESTS=OFF                            \
      -DECM_ENABLE_SANITIZERS='address;leak;undefined' \
      -DCMAKE_CXX_COMPILER=clang++                     \
      .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf poppler-21.05.0
```

## GraphViz

```sh
wget https://www2.graphviz.org/Packages/stable/portable_source/graphviz-2.44.1.tar.gz -P /sources &&

tar xzvf /sources/graphviz-2.44.1.tar.gz &&
cd        graphviz-2.44.1                &&

sed -i '/LIBPOSTFIX="64"/s/64//' configure.ac &&

autoreconf               &&
./configure --prefix=/usr \
            --disable-php \
            PS2PDF=true  &&

make              &&
sudo make install &&

cd .. &&
rm -rf graphviz-2.44.1
```

## LittleCMS

The Little Color Management System.

```sh
wget https://downloads.sourceforge.net/lcms/lcms2-2.12.tar.gz -P /sources &&

tar xzvf /sources/lcms2-2.12.tar.gz &&
cd        lcms2-2.12                &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf lcms2-2.12
```

## ImageMagick

```sh
wget https://download.imagemagick.org/ImageMagick/download/releases/ImageMagick-7.0.11-13.tar.gz -P /sources/ &&

tar xzvf /sources/ImageMagick-7.0.11-13.tar.gz &&
cd        ImageMagick-7.0.11-13                &&

./configure --prefix=/usr     \
            --sysconfdir=/etc \
            --enable-hdri     \
            --with-modules    \
            --with-perl       \
            --with-gslib      \
            --with-rsvg       \
            --with-gvc        \
            --disable-static &&

make                                                                   &&
make -k check                                                          &&
sudo make DOCUMENTATION_PATH=/usr/share/doc/imagemagick-7.0.11 install &&

cd .. &&
sudo rm -rf ImageMagick-7.0.11-13
```

## sassc

CSS preprocessor.

```sh
wget https://github.com/sass/sassc/archive/3.6.2/sassc-3.6.2.tar.gz -P /sources &&
wget https://github.com/sass/libsass/archive/3.6.5/libsass-3.6.5.tar.gz -P /sources &&

tar xzvf /sources/libsass-3.6.5.tar.gz &&
cd        libsass-3.6.5                &&

autoreconf -fi &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd ..                &&
rm -rf libsass-3.6.5 &&

tar xzvf /sources/sassc-3.6.2.tar.gz &&
cd        sassc-3.6.2                &&

autoreconf -fi &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf sassc-3.6.2
```

## GhostScript

Libraries for rendering postscript.

We remove vendored libraries since we have newer versions of these already installed.

```sh
wget https://github.com/ArtifexSoftware/ghostpdl-downloads/releases/download/gs9533/ghostscript-9.53.3.tar.xz -P /sources &&
wget http://www.linuxfromscratch.org/patches/blfs/10.1/ghostscript-9.53.3-freetype_fix-1.patch -P /sources/patches &&
wget https://downloads.sourceforge.net/gs-fonts/ghostscript-fonts-std-8.11.tar.gz -P /sources &&
wget https://downloads.sourceforge.net/gs-fonts/gnu-gs-fonts-other-6.0.tar.gz -P /sources &&

tar xvf /sources/ghostscript-9.53.3.tar.xz &&
cd       ghostscript-9.53.3                &&

rm -rf freetype jpeg libpng openjpeg zlib                              &&
patch -Np1 -i /sources/patches/ghostscript-9.53.3-freetype_fix-1.patch &&

./configure --prefix=/usr           \
            --disable-compile-inits \
            --enable-dynamic        \
            --with-system-libtiff &&

make                                                                            &&
make so                                                                         &&
sudo make install                                                               &&
sudo make soinstall                                                             &&
sudo install -v -m644 base/*.h /usr/include/ghostscript                         &&
sudo ln -sfvn ghostscript /usr/include/ps                                       &&
sudo mv -v /usr/share/doc/ghostscript/9.53.3 /usr/share/doc/ghostscript-9.53.3  &&
sudo rm -rfv /usr/share/doc/ghostscript                                         &&
sudo cp -r examples/ /usr/share/ghostscript/9.53.3/                             &&

sudo tar -xvf /sources/ghostscript-fonts-std-8.11.tar.gz -C /usr/share/ghostscript --no-same-owner                            &&
sudo tar -xvf /sources/gnu-gs-fonts-other-6.0.tar.gz     -C /usr/share/ghostscript --no-same-owner                            &&
sudo fc-cache -v /usr/share/ghostscript/fonts/ &&

cd .. &&
rm -rf ghostscript-9.53.3
```
