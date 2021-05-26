# Multimedia and documentation generation

There is no clear dependency ordering between the various libraries and tools presented here and the programming languages presented in the next section. You may need to jump back and forth quite a bit. A lot of this is probably not needed if you only intend to build a headless system, but if you wish to use it as a build server and include building of documentation, many documentation generation tools can do quite a bit more if they include support for graphics, even if the graphics cannot be viewed on the same machine.

If you wish to build `pandoc`, it requires a Haskell compiler, and `mdbook` requires `Rust`, as does `librsvg`. The recommended build order is to build everything else first, in order to have full-fledged documentation generators for `GDB` and the various other compilers, debuggers, and programming standard libraries in the next section, then come back to build `pandoc` and `mdbook`.

Many libraries here may also require `CMake` to generate the `Makefile`, so you may want to head over the devtools section on build system and install `CMake` first, then come back.

## Fonts and typesetting

### FreeType and deps

You may wish to install `brotli` first, which provides implementations of various compression and encoding algorithms used by some of these other libraries.

`FreeType` also depends upon `libpng`, which you will want to build first. You will also need to build `HarfBuzz` after building `FreeType`, then rebuild and reinstall `FreeType`.

#### Graphite2

This is not strictly a dependency of `FreeType` itself, but `HarfBuzz` may not behave as expected if it is not present. Note that `Graphite` also has circular dependencies on `FreeType` and `HarfBuzz`, so though it will build without them, you will also want to rebuild after.

Fetch:

```sh
curl https://github.com/silnrsi/graphite/releases/download/1.3.14/graphite2-1.3.14.tgz -o graphite2-1.3.14.tgz &&
tar xzvf graphite2-1.3.14.tgz
```

Build:

```sh
cd graphite2-1.3.14
sed -i '/cmptest/d' tests/CMakeLists.txt &&
mkdir build &&
cd    build &&
cmake -DCMAKE_INSTALL_PREFIX=/usr ..
make
```

Install:

```sh
sudo make install
```

#### brotli

Fetch:

```sh
curl https://github.com/google/brotli/archive/v1.0.9/brotli-1.0.9.tar.gz -o brotli-1.0.9.tar.gz &&
tar xzvf brotli-1.0.9.tar.gz
```

Build:

```sh
sed -i 's@-R..libdir.@@' scripts/*.pc.in
mkdir out &&
cd    out &&

cmake -DCMAKE_INSTALL_PREFIX=/usr \
      -DCMAKE_BUILD_TYPE=Release  \
      ..
make
```

Test:

```sh
make test
```

To build optional Python bindings:

```sh
pushd ..              &&
python setup.py build &&
popd
```

Install:

```sh
sudo make install
```

Install the optional Python bindings:

```sh
pushd ..                                  &&
sudo python setup.py install --optimize=1 &&
popd
```

#### FreeType

Fetch:

```sh
curl https://downloads.sourceforge.net/freetype/freetype-2.10.4.tar.xz -o freetype-2.10.4.tar.xz &&
curl https://downloads.sourceforge.net/freetype/freetype-doc-2.10.4.tar.xz -o freetype-doc-2.10.4.tar.xz &&
tar xvf freetype-2.10.4.tar.xz &&
tar xvf freetype-doc-2.10.4.tar.xz --strip-components=2 -C freetype-2.10.4/docs
```

Build:

```sh
sed -ri "s:.*(AUX_MODULES.*valid):\1:" modules.cfg &&
sed -r "s:.*(#.*SUBPIXEL_RENDERING) .*:\1:" \
    -i include/freetype/config/ftoption.h  &&
./configure --prefix=/usr --enable-freetype-config --disable-static
make
```

Install:

```sh
sudo make install                                       &&
sudo install -v -m755 -d /usr/share/doc/freetype-2.10.4 &&
sudo cp -v -R docs/*     /usr/share/doc/freetype-2.10.4 &&
sudo rm -v /usr/share/doc/freetype-2.10.4/freetype-config.1
```

#### HarfBuzz

Fetch:

```sh
curl https://github.com/harfbuzz/harfbuzz/releases/download/2.7.4/harfbuzz-2.7.4.tar.xz -o harfbuzz-2.7.4.tar.xz &&
tar xvf harfbuzz-2.7.4.tar.xz
```

Build:

```sh
mkdir build &&
cd    build &&
meson --prefix=/usr -Dgraphite=enabled -Dbenchmark=disabled
ninja
```

Install:

```sh
sudo ninja install
```

After `HarfBuzz` is installed, you should now rebuild and reinstall `Graphite2` and `FreeType`.

## Image libraries

### libpng

Fetch:

```sh
curl https://downloads.sourceforge.net/libpng/libpng-1.6.37.tar.xz -o libpng-1.6.37.tar.xz &&
curl https://downloads.sourceforge.net/sourceforge/libpng-apng/libpng-1.6.37-apng.patch.gz -o libpng-1.6.37-apng.patch.gz &&
tar xvf libpng-1.6.37.tar.xz &&
gzip -d libpng-1.6.37-apng.patch.gz
```

Build:

```sh
cat ../libpng-1.6.37-apng.patch | patch -p1
./configure --prefix=/usr --disable-static
make
```

Test:

```sh
make check
```

Install:

```sh
sudo make install &&
sudo install -vdm755 /usr/share/doc/libpng-1.6.37 &&
sudo cp -v README libpng-manual.txt /usr/share/doc/libpng-1.6.37
```

### OpenJPEG

Fetch:

```sh
curl https://github.com/uclouvain/openjpeg/archive/v2.4.0/openjpeg-2.4.0.tar.gz -o openjpeg-2.4.0.tar.gz &&
tar xzvf openjpeg-2.4.0.tar.gz
```

Build:

```sh
cd openjpeg-2.4.0

mkdir -v build &&
cd       build &&
cmake -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_INSTALL_PREFIX=/usr \
      -DBUILD_STATIC_LIBS=OFF ..
make
```

Install:

```sh
sudo make install &&
pushd ../doc &&
    for man in man/man?/* ; do
        sudo install -v -D -m 644 $man /usr/share/$man
    done
popd
```

### libjpeg-turbo

Fetch:

```sh
curl https://downloads.sourceforge.net/libjpeg-turbo/libjpeg-turbo-2.0.6.tar.gz -o libjpeg-turbo-2.0.6.tar.gz &&
tar xzvf libjpeg-turbo-2.0.6.tar.gz
```

Build:

```sh
mkdir build &&
cd    build &&
cmake -DCMAKE_INSTALL_PREFIX=/usr \
      -DCMAKE_BUILD_TYPE=RELEASE  \
      -DENABLE_STATIC=FALSE       \
      -DCMAKE_INSTALL_DOCDIR=/usr/share/doc/libjpeg-turbo-2.0.6 \
      -DCMAKE_INSTALL_DEFAULT_LIBDIR=lib  \
      ..
make
```

Test:

```sh
make test
```

Install:

```sh
sudo make install
```

### libTIFF

Fetch:

```sh
curl http://download.osgeo.org/libtiff/tiff-4.2.0.tar.gz -o tiff-4.2.0.tar.gz &&
tar xzvf tiff-4.2.0.tar.gz
```

Build:

```sh
cd tiff-4.2.0

mkdir -p libtiff-build &&
cd       libtiff-build &&
cmake -DCMAKE_INSTALL_DOCDIR=/usr/share/doc/libtiff-4.2.0 \
      -DCMAKE_INSTALL_PREFIX=/usr -G Ninja ..
ninja
```

Install:

```sh
sudo ninja install
```

### gdk-pixbuf

Pixel buffer library divested from GTK+.

Fetch:

```sh
curl https://download.gnome.org/sources/gdk-pixbuf/2.42/gdk-pixbuf-2.42.2.tar.xz -o gdk-pixbuf-2.42.2.tar.xz &&
tar xvf gdk-pixbuf-2.42.2.tar.xz
```

Build:

```sh
cd gdk-pixbuf-2.42.2

mkdir build &&
cd build    &&
meson --prefix=/usr ..
ninja
```

Test:

```sh
ninja test
```

Install

```sh
sudo ninja install
```

### Pixman

Libraries for low-level pixel manipulation.

Fetch:

```sh
curl https://www.cairographics.org/releases/pixman-0.40.0.tar.gz -o pixman-0.40.0.tar.xz &&
tar xvf pixman-0.40.0.tar.gz
```

Build:

```sh
cd pixman-0.40.0

mkdir build &&
cd    build &&
meson --prefix=/usr
ninja
```

Test:

```sh
ninja test
```

Install:

```sh
sudo ninja install
```

### Cairo

2d graphics library.

Fetch:

```sh
curl http://anduin.linuxfromscratch.org/BLFS/cairo/cairo-1.17.2+f93fc72c03e.tar.xz -o cairo-1.17.2+f93fc72c03e.tar.xz &&
tar xvf cairo-1.17.2+f93fc72c03e.tar.xz
```

Build:

```sh
cd cairo-1.17.2+f93fc72c03e

autoreconf -fv &&
./configure --prefix=/usr    \
            --disable-static \
            --enable-tee
make
```

Install:

```sh
sudo make install
```

### Pango

Text rendering and internationalization.

Fetch:

```sh
curl https://download.gnome.org/sources/pango/1.48/pango-1.48.2.tar.xz -o pango-1.48.2.tar.xz &&
tar xvf pango-1.48.2.tar.xz
```

Build:

```sh
cd pango-1.48.2

mkdir build &&
cd    build &&
meson --prefix=/usr ..
ninja
```

Install:

```sh
sudo ninja install
```

### GObject introspection

Inspect object APIs.

Fetch:

```sh
curl https://download.gnome.org/sources/gobject-introspection/1.66/gobject-introspection-1.66.1.tar.xz -o gobject-introspection-1.66.1.tar.xz &&
tar xvf gobject-introspection-1.66.1.tar.xz
```

Build:

```sh
cd gobject-introspection-1.66.1

mkdir build &&
cd    build &&
meson --prefix=/usr ..
ninja
```

Test:

This requires the `Python` `markdown` package, which can be installed temporarily into a virtual environment.

```sh
python -m venv testenv
. ./testenv/bin/activate
pip install markdown

ninja test -k0

deactivate
```

Install:

```sh
sudo ninja install
```

### Vala

Scripting language for GNOME.

Fetch:

```sh
curl https://download.gnome.org/sources/vala/0.50/vala-0.50.3.tar.xz -o vala-0.59.3.tar.xz &&
tar xvf vala-0.59.3.tar.xz
```

Build:

Requires `--disable-valadoc` if `GraphViz` is not installed.

```sh
cd vala-0.59.3

./configure --prefix=/usr
make
```

Install:

```sh
sudo make install
```

### librsvg

SVG library and toolset. For Hacker News street cred, written in Rust.

Fetch:

```sh
curl https://download.gnome.org/sources/librsvg/2.50/librsvg-2.50.3.tar.xz -o librsvg-2.50.3.tar.xz &&
tar xvf librsvg-2.50.3.tar.xz
```

Build:

```sh
cd librsvg-2.50.3

./configure --prefix=/usr    \
            --enable-vala    \
            --disable-static \
            --docdir=/usr/share/doc/librsvg-2.50.3
make
```

This may fail the first time due to circular dependencies in the graph between this and `gdk-pixbuf`, which may need to be rebuilt first.

Test:

```sh
make check
```

Install:

```sh
sudo make install
```

## Poppler

PDF rendering library.

Fetch:

```sh
curl https://poppler.freedesktop.org/poppler-21.05.0.tar.xz -o poppler-21.05.0.tar.xz &&
tar xvf poppler-21.05.0.tar.xz
```

Build:

`clang` is optional but required for the fuzzing and address sanitizers to work, which may help prevent security issues.

```sh
cd poppler-21.05.0

mkdir -v build &&
cd       build &&
cmake -DCMAKE_INSTALL_PREFIX=/usr                      \
      -DCMAKE_BUILD_TYPE=release                       \
      -DBUILD_GTK_TESTS=OFF                            \
      -DECM_ENABLE_SANITIZERS='address;leak;undefined' \
      -DCMAKE_CXX_COMPILER=clang++                     \
      ..
make
```

Install:

```sh
sudo make install
```

## GraphViz

Fetch:

```sh
curl https://www2.graphviz.org/Packages/stable/portable_source/graphviz-2.44.1.tar.gz -o graphviz-2.44.1.tar.gz &&
tar xzvf graphviz-2.44.1.tar.gz
```

Build:

```sh
cd graphviz-2.44.1

sed -i '/LIBPOSTFIX="64"/s/64//' configure.ac &&
autoreconf                                    &&
./configure --prefix=/usr \
            --disable-php \
            PS2PDF=true
make
```

Install:

```sh
sudo make install
```

## FFmpeg and deps

`FFmpeg` has many optional features. If you wish to use it for full-fledged audio and video editing and transcoding, it will need these.

### FreeType2

Allow rendering of certain fonts.

Fetch:

```sh
curl https://downloads.sourceforge.net/freetype/freetype-2.10.4.tar.xz -o freetype-2.10.4.tar.xz &&
tar xvf freetype-2.10.4.tar.xz
```

Build:

```sh

```

### FontConfig

Fetch:

```sh
curl https://www.freedesktop.org/software/fontconfig/release/fontconfig-2.13.1.tar.bz2 -o fontconfig-2.13.1.tar.bz2 &&
tar xvf fontconfig-2.13.1.tar.bz2
```

Build:

```sh
cd fontconfig-2.13.1

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-docs       \
            --docdir=/usr/share/doc/fontconfig-2.13.1
make
```

Install:

```sh
sudo make install
```

### LAME

Fetch:

```sh
curl https://downloads.sourceforge.net/lame/lame-3.100.tar.gz -o lame-3.100.tar.gz &&
tar xzvf lame-3.100.tar.gz
```

Build:

```sh
cd lame-3.100

./configure --prefix=/usr --enable-mp3rtp --disable-static
make
```

Install:

```sh
sudo make pkghtmldir=/usr/share/doc/lame-3.100 install
```

### libogg

Fetch:

```sh
curl https://downloads.xiph.org/releases/ogg/libogg-1.3.4.tar.xz -o libogg-1.3.4.tar.xz &&
tar xvf libogg-1.3.4.tar.xz
```

Build:

```sh
cd libogg-1.3.4

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/libogg-1.3.4
make
```

Install:

```sh
sudo make install
```

### libvorgis

Fetch:

```sh
curl https://downloads.xiph.org/releases/vorbis/libvorbis-1.3.7.tar.xz -o libvorbis-1.3.7.tar.xz &&
tar xvf libvorbis-1.3.7.tar.xz
```

Build:

```sh
cd libvorbis-1.3.7

./configure --prefix=/usr --disable-static
make
```

Install:

```sh
sudo make install &&
sudo install -v -m644 doc/Vorbis* /usr/share/doc/libvorbis-1.3.7
```

### libtheora

Fetch:

```sh
curl https://downloads.xiph.org/releases/theora/libtheora-1.1.1.tar.xz -0 libtheora-1.1.1.tar.xz &&
tar xvf libtheora-1.1.1.tar.xz
```

Build:

```sh
cd libtheora-1.1.1

sed -i 's/png_\(sizeof\)/\1/g' examples/png2theora.c &&
./configure --prefix=/usr --disable-static
make
```

Install:

```sh
sudo make install
```

### fdk-aac

Franhaufer implementation of Advanced Audio Encoding.

Fetch:

```sh
curl https://downloads.sourceforge.net/opencore-amr/fdk-aac-2.0.1.tar.gz -o fdk-aac-2.0.1.tar.gz &&
tar xzvf fdk-aac-2.0.1.tar.gz
```

Build:

```sh
cd fdk-aac-2.0.1

./configure --prefix=/usr --disable-static
make
```

Install:

```sh
sudo make install
```

### Opus

Loss audio encoding.

Fetch:

```sh
curl https://archive.mozilla.org/pub/opus/opus-1.3.1.tar.gz -o opus-1.3.1.tar.gz &&
tar xzvf opus-1.3.1.tar.gz
```

Build:

```sh
cd opus-1.3.1

./configure --prefix=/usr    \
            --disable-static \
            --docdir=/usr/share/doc/opus-1.3.1
make
```

Test:

```sh
make check
```

Install:

```sh
sudo make install
```

### FriBidi

Unicode Bidirectional Algorithm for bidirectional alphabet rendering.

Fetch:

```sh
curl https://github.com/fribidi/fribidi/releases/download/v1.0.9/fribidi-1.0.9.tar.xz -o fribidi-1.0.9.tar.xz &&
tar xvf fribidi-1.0.9.tar.xz
```

Build:

```sh
cd fribidi-1.0.9

mkdir build &&
cd    build &&
meson --prefix=/usr ..
ninja
```

Install:

```sh
sudo ninja install
```

### nasm

The Netwide Assembler. Required by `libass`.

Fetch:

```sh
curl http://www.nasm.us/pub/nasm/releasebuilds/2.15.05/nasm-2.15.05.tar.xz -o nasm-2.15.05.tar.xz &&
curl http://www.nasm.us/pub/nasm/releasebuilds/2.15.05/nasm-2.15.05-xdoc.tar.xz -o nasm-2.15.05-xdoc.tar.xz &&
tar xvf nasm-2.15.05.tar.xz &&
tar xvf nasm-2.15.05-xdoc.tar.xz --strip-components=1
```

Build:

```sh
cd nasm-2.15.05

./configure --prefix=/usr
make
```

Install:

```sh
sudo make install &&
sudo install -m755 -d     /usr/share/doc/nasm-2.15.05/ &&
sudo cp -v doc/*.{txt,ps} /usr/share/doc/nasm-2.15.05
```

### yasm

Although listed as optional on the BLFS page, `FFmpeg` strongly recommends the `yasm` assembler for x86_64 architectures.

Fetch:


```sh
curl http://www.tortall.net/projects/yasm/releases/yasm-1.3.0.tar.gz -o yasm-1.3.0.tar.gz && tar xzvf yasm-1.3.0.tar.gz
```

Build:

```sh
cd yasm-1.3.0

sed -i 's#) ytasm.*#)#' Makefile.in &&
./configure --prefix=/usr
make
```

Install:

```sh
sudo make install
```

Fetch:

```sh
curl http://ffmpeg.org/releases/ffmpeg-4.3.2.tar.xz -o ffmpeg-4.3.2.tar.xz &&
tar xvf ffmpeg-4.3.2.tar.xz
```

### libass

Advanced Substation Alpha/Substation Alpha subtitle formatting.

Fetch:

```sh
curl https://github.com/libass/libass/releases/download/0.15.0/libass-0.15.0.tar.xz -o libass-0.15.0.tar.xz &&
tar xvf libass-0.15.0.tar.xz
```

Build:

```sh
cd libass-0.15.0

./configure --prefix=/usr --disable-static
make
```

Install:

```sh
sudo make install
```

### x264

H.264 MPEG-4 video codec.

Fetch:

```sh
curl http://anduin.linuxfromscratch.org/BLFS/x264/x264-20210211.tar.xz -o x264-20210211.tar.xz &&
tar xvf x264-20210211.tar.xz
```

Build:

```sh
cd x264-20210211

./configure --prefix=/usr   \
            --enable-shared \
            --disable-cli
make
```

Install:

```sh
sudo make install
```

### x265

H.265 HEVC video codec.

Fetch:

```sh
curl http://anduin.linuxfromscratch.org/BLFS/x265/x265_3.4.tar.gz -o x265_3.4.tar.gz &&
tar xzvf x265_3.4.tar.gz
```

Build:

```sh
cd x265_3.4

mkdir bld &&
cd    bld &&
cmake -DCMAKE_INSTALL_PREFIX=/usr ../source
make
```

Install:

```sh
sudo make install &&
sudo rm -vf /usr/lib/libx265.a
```

### libvpx

HTML5 video encoding.

Fetch:

```sh
curl https://github.com/webmproject/libvpx/archive/v1.9.0/libvpx-1.9.0.tar.gz -o libvpx-1.9.0.tar.gz &&
tar xzvf libvpx-1.9.0.tar.gz
```

Build:

```sh
cd libvpx-1.9.0

sed -i 's/cp -p/cp/' build/make/Makefile &&
mkdir libvpx-build            &&
cd    libvpx-build            &&
../configure --prefix=/usr    \
             --enable-shared  \
             --disable-static
make
```

Install:

```sh
sudo make install
```

### FFmpeg

A huge suite of CLI tools for manipulating audio and video files. If you wish to build the documentation, skip ahead and install `GhostScript` and `Doxygen` first.

Build:

```sh
cd ffmpeg-4.3.2
sed -i 's/-lflite"/-lflite -lasound"/' configure &&
./configure --prefix=/usr        \
            --enable-gpl         \
            --enable-version3    \
            --enable-nonfree     \
            --disable-static     \
            --enable-shared      \
            --disable-debug      \
            --enable-swresample  \
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
            --docdir=/usr/share/doc/ffmpeg-4.3.2
make &&
gcc tools/qt-faststart.c -o tools/qt-faststart
```

Build the documentation (if doxygen is installed):

```sh
doxygen doc/Doxyfile
```

Install:

```sh
sudo make install &&
sudo install -v -m755    tools/qt-faststart /usr/bin           &&
sudo install -v -m755 -d           /usr/share/doc/ffmpeg-4.3.2 &&
sudo install -v -m644    doc/*.txt /usr/share/doc/ffmpeg-4.3.2
```

Install doxygen-generated docs:

```sh
sudo install -v -m755 -d /usr/share/doc/ffmpeg-4.3.2/api                    &&
sudo cp -vr doc/doxy/html/* /usr/share/doc/ffmpeg-4.3.2/api                 &&
find /usr/share/doc/ffmpeg-4.3.2/api -type f -exec sudo chmod -c 0644 {} \; &&
find /usr/share/doc/ffmpeg-4.3.2/api -type d -exec sudo chmod -c 0755 {} \;
```

## ImageMagick

Fetch:

```sh
curl https://download.imagemagick.org/ImageMagick/download/releases/ImageMagick-7.0.11-13.tar.gz -o ImageMagick-7.0.11-13.tar.gz &&
tar xzvf ImageMagick-7.0.11-13.tar.gz
```

Build:

```sh
cd ImageMagick-7.0.11-13

./configure --prefix=/usr     \
            --sysconfdir=/etc \
            --enable-hdri     \
            --with-modules    \
            --with-perl       \
            --disable-static
make
```

Test:

```sh
make check
```

Install:

```sh
sudo make DOCUMENTATION_PATH=/usr/share/doc/imagemagick-7.0.11 install
```

## Documentation generation tools

## Sphinx

`sphinx-build` is a Python console script and can be installed by `pip`:

```sh
pip install sphinx
```

To install globally for your user only:

```sh
pip install --user sphinx
```

## GhostScript

Libraries for rendering postscript.

Fetch:

```sh
curl https://github.com/ArtifexSoftware/ghostpdl-downloads/releases/download/gs9533/ghostscript-9.53.3.tar.xz -o ghostscript-9.53.3.tar.xz &&
curl http://www.linuxfromscratch.org/patches/blfs/10.1/ghostscript-9.53.3-freetype_fix-1.patch -o ghostscript-9.53.3-freetype_fix-1.patch &&
curl https://downloads.sourceforge.net/gs-fonts/ghostscript-fonts-std-8.11.tar.gz -o ghostscript-fonts-std-8.11.tar.gz &&
curl https://downloads.sourceforge.net/gs-fonts/gnu-gs-fonts-other-6.0.tar.gz -o gnu-gs-fonts-other-6.0.tar.gz &&
tar xvf ghostscript-9.53.3.tar.xz
```

Build (we remove vendored libraries since we have newer versions of these already installed):

```sh
cd ghostscript-9.53.3

rm -rf freetype jpeg libpng openjpeg zlib
patch -Np1 -i ../ghostscript-9.53.3-freetype_fix-1.patch
./configure --prefix=/usr           \
            --disable-compile-inits \
            --enable-dynamic        \
            --with-system-libtiff
make && make so
```

Install:

```sh
sudo make install   &&
sudo make soinstall &&
sudo install -v -m644 base/*.h /usr/include/ghostscript                         &&
sudo ln -sfvn ghostscript /usr/include/ps                                       &&
sudo mv -v /usr/share/doc/ghostscript/9.53.3 /usr/share/doc/ghostscript-9.53.3  &&
sudo rm -rfv /usr/share/doc/ghostscript                                         &&
sudo cp -r examples/ /usr/share/ghostscript/9.53.3/
```

Install fonts:

```sh
sudo tar -xvf ../ghostscript-fonts-std-8.11.tar.gz -C /usr/share/ghostscript --no-same-owner &&
sudo tar -xvf ../gnu-gs-fonts-other-6.0.tar.gz     -C /usr/share/ghostscript --no-same-owner &&
sudo fc-cache -v /usr/share/ghostscript/fonts/
```

## asciidoc

Fetch:

```sh
curl https://github.com/asciidoc/asciidoc-py3/releases/download/9.1.0/asciidoc-9.1.0.tar.gz -o asciidoc-9.1.0.tar.gz &&
tar xzvf asciidoc-9.1.0.tar.gz
```

Build:

```sh
cd asciidoc-9.1.0

sed -i 's:doc/testasciidoc.1::' Makefile.in &&
rm doc/testasciidoc.1.txt
./configure --prefix=/usr     \
            --sysconfdir=/etc \
            --docdir=/usr/share/doc/asciidoc-9.1.0
make
```

Install:

```sh
sudo make install &&
sudo make docs
```

## TeX Live

```sh

```

## Doxygen

Fetch:

```sh
curl http://doxygen.nl/files/doxygen-1.9.1.src.tar.gz -o doxygen-1.9.1.src.tar.gz &&
tar xzvf doxygen-1.9.1.src.tar.gz
```

Build:

```sh
cd doxygen-1.9.1

mkdir -v build &&
cd       build &&
cmake -G "Unix Makefiles"         \
      -DCMAKE_BUILD_TYPE=Release  \
      -DCMAKE_INSTALL_PREFIX=/usr \
      -Wno-dev ..
make
```

Test:

```sh
make tests
```

Build documentation:

```sh
cmake -DDOC_INSTALL_DIR=share/doc/doxygen-1.9.1 -Dbuild_doc=ON .. &&
make docs
```

Install:

```sh
sudo make install &&
sudo install -vm644 ../doc/*.1 /usr/share/man/man1
```

### pandoc

```sh

```

### mdbook

```sh

```
