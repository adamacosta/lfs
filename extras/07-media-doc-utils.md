# Multimedia and documentation generation

There is no clear dependency ordering between the various libraries and tools presented here and the programming languages presented in the next section. You may need to jump back and forth quite a bit. A lot of this is probably not needed if you only intend to build a headless system, but if you wish to use it as a build server and include building of documentation, many documentation generation tools can do quite a bit more if they include support for graphics, even if the graphics cannot be viewed on the same machine.

If you wish to build `pandoc`, it requires a Haskell compiler, and `mdbook` requires `Rust`, as does `librsvg`. The recommended build order is to build everything else first, in order to have full-fledged documentation generators for `GDB` and the various other compilers, debuggers, and programming standard libraries in the next section, then come back to build `pandoc` and `mdbook`.

Many libraries here may also require `CMake` to generate the `Makefile`, so you may want to head over the devtools section on build systems and install `CMake` first, then come back.

## Fonts and typesetting

### FreeType and deps

You may wish to install `brotli` first, which provides implementations of various compression and encoding algorithms used by some of these other libraries.

`FreeType` also depends upon `libpng`, which you will want to build first. You will also need to build `HarfBuzz` after building `FreeType`, then rebuild and reinstall `FreeType`.

#### Graphite2

This is not strictly a dependency of `FreeType` itself, but `HarfBuzz` may not behave as expected if it is not present. Note that `Graphite` also has circular dependencies on `FreeType` and `HarfBuzz`, so though it will build without them, you will also want to rebuild after.

```sh
curl https://github.com/silnrsi/graphite/releases/download/1.3.14/graphite2-1.3.14.tgz -o /sources/graphite2-1.3.14.tgz &&

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
curl https://github.com/google/brotli/archive/v1.0.9/brotli-1.0.9.tar.gz -o /sources/brotli-1.0.9.tar.gz &&

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
curl https://downloads.sourceforge.net/freetype/freetype-2.10.4.tar.xz -o /sources/freetype-2.10.4.tar.xz &&
curl https://downloads.sourceforge.net/freetype/freetype-doc-2.10.4.tar.xz -o /sources/freetype-doc-2.10.4.tar.xz &&

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
curl https://github.com/harfbuzz/harfbuzz/releases/download/2.8.1/harfbuzz-2.8.1.tar.xz -o /sources/harfbuzz-2.8.1.tar.xz &&

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

Fetch:

```sh
curl https://downloads.sourceforge.net/libpng/libpng-1.6.37.tar.xz -o /sources/libpng-1.6.37.tar.xz &&
curl https://downloads.sourceforge.net/sourceforge/libpng-apng/libpng-1.6.37-apng.patch.gz -o /sources/libpng-1.6.37-apng.patch.gz &&

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
curl https://github.com/uclouvain/openjpeg/archive/v2.4.0/openjpeg-2.4.0.tar.gz -o /sources/openjpeg-2.4.0.tar.gz &&

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
curl https://downloads.sourceforge.net/libjpeg-turbo/libjpeg-turbo-2.0.6.tar.gz -o /sources/libjpeg-turbo-2.0.6.tar.gz &&

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
curl http://download.osgeo.org/libtiff/tiff-4.2.0.tar.gz -o /sources/tiff-4.2.0.tar.gz &&

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
curl https://download.gnome.org/sources/gdk-pixbuf/2.42/gdk-pixbuf-2.42.6.tar.xz -o /sources/gdk-pixbuf-2.42.6.tar.xz &&

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
curl https://www.cairographics.org/releases/pixman-0.40.0.tar.gz -o /sources/pixman-0.40.0.tar.xz &&

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

## FontConfig

```sh
curl https://www.freedesktop.org/software/fontconfig/release/fontconfig-2.13.1.tar.bz2 -o /sources/fontconfig-2.13.1.tar.bz2 &&

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

2d graphics library.

```sh
curl https://www.cairographics.org/snapshots/cairo-1.17.4.tar.xz -o /sources/cairo-1.17.4.tar.xz &&

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
curl https://github.com/fribidi/fribidi/releases/download/v1.0.9/fribidi-1.0.9.tar.xz -o /sources/fribidi-1.0.9.tar.xz &&

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
curl https://download.gnome.org/sources/pango/1.48/pango-1.48.5.tar.xz -o /sources/pango-1.48.5.tar.xz &&

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

Inspect object APIs.

```sh
curl https://download.gnome.org/sources/gobject-introspection/1.68/gobject-introspection-1.68.0.tar.xz -o /sources/gobject-introspection-1.68.0.tar.xz &&

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

## SDL

```sh
curl https://www.libsdl.org/release/SDL2-2.0.14.tar.gz -o /sources/SDL2-2.0.14.tar.gz &&

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

## libwebp

```sh
curl http://downloads.webmproject.org/releases/webp/libwebp-1.2.0.tar.gz -o /sources/libwebp-1.2.0.tar.gz &&

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
curl https://github.com/google/woff2/archive/v1.0.2/woff2-1.0.2.tar.gz -o /sources/woff2-1.0.2.tar.gz &&

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
curl https://poppler.freedesktop.org/poppler-21.05.0.tar.xz -o /sources/poppler-21.05.0.tar.xz &&

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
curl https://www2.graphviz.org/Packages/stable/portable_source/graphviz-2.44.1.tar.gz -o /sources/graphviz-2.44.1.tar.gz &&

tar xzvf /sources/graphviz-2.44.1.tar.gz &&
cd        graphviz-2.44.1                &&

sed -i '/LIBPOSTFIX="64"/s/64//' configure.ac &&
autoreconf                                    &&
./configure --prefix=/usr \
            --disable-php \
            PS2PDF=true                       &&

make              &&
sudo make install &&

cd .. &&
rm -rf graphviz-2.44.1
```

## ImageMagick

```sh
curl https://download.imagemagick.org/ImageMagick/download/releases/ImageMagick-7.0.11-13.tar.gz -o /sources/ImageMagick-7.0.11-13.tar.gz &&

tar xzvf /sources/ImageMagick-7.0.11-13.tar.gz &&
cd        ImageMagick-7.0.11-13                &&

./configure --prefix=/usr     \
            --sysconfdir=/etc \
            --enable-hdri     \
            --with-modules    \
            --with-perl       \
            --disable-static &&

make                                                                   &&
make -k check                                                          &&
sudo make DOCUMENTATION_PATH=/usr/share/doc/imagemagick-7.0.11 install &&

cd .. &&
sudo rm -rf ImageMagick-7.0.11-13
```

## Documentation generation tools

### GhostScript

Libraries for rendering postscript.

We remove vendored libraries since we have newer versions of these already installed.

```sh
curl https://github.com/ArtifexSoftware/ghostpdl-downloads/releases/download/gs9533/ghostscript-9.53.3.tar.xz -o /sources/ghostscript-9.53.3.tar.xz &&
curl http://www.linuxfromscratch.org/patches/blfs/10.1/ghostscript-9.53.3-freetype_fix-1.patch -o /sources/ghostscript-9.53.3-freetype_fix-1.patch &&
curl https://downloads.sourceforge.net/gs-fonts/ghostscript-fonts-std-8.11.tar.gz -o /sources/ghostscript-fonts-std-8.11.tar.gz &&
curl https://downloads.sourceforge.net/gs-fonts/gnu-gs-fonts-other-6.0.tar.gz -o /sources/gnu-gs-fonts-other-6.0.tar.gz &&

tar xvf /sources/ghostscript-9.53.3.tar.xz &&
cd       ghostscript-9.53.3                &&

rm -rf freetype jpeg libpng openjpeg zlib                      &&
patch -Np1 -i /sources/ghostscript-9.53.3-freetype_fix-1.patch &&

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

### asciidoc

```sh
curl https://github.com/asciidoc/asciidoc-py3/releases/download/9.1.0/asciidoc-9.1.0.tar.gz -o /sources/asciidoc-9.1.0.tar.gz &&

tar xzvf /sources/asciidoc-9.1.0.tar.gz &&
cd        asciidoc-9.1.0                &&

sed -i 's:doc/testasciidoc.1::' Makefile.in &&
rm doc/testasciidoc.1.txt                   &&

./configure --prefix=/usr     \
            --sysconfdir=/etc \
            --docdir=/usr/share/doc/asciidoc-9.1.0 &&

make              &&
sudo make install &&
sudo make docs    &&

cd .. &&
rm -rf asciidoc-9.1.0
```

### Doxygen

```sh
curl http://doxygen.nl/files/doxygen-1.9.1.src.tar.gz -o /sources/doxygen-1.9.1.src.tar.gz &&

tar xzvf /sources/doxygen-1.9.1.src.tar.gz &&
cd        doxygen-1.9.1                    &&

rm src/._xmlgen.cpp &&

mkdir -v build &&
cd       build &&

cmake -G "Unix Makefiles"         \
      -DCMAKE_BUILD_TYPE=Release  \
      -DCMAKE_INSTALL_PREFIX=/usr \
      -Wno-dev .. &&

make                                               &&
make tests                                         &&
sudo make install                                  &&
sudo install -vm644 ../doc/*.1 /usr/share/man/man1 &&

cd ../.. &&
rm -rf doxygen-1.9.1
```

### Sphinx

`sphinx-build` is a Python console script and can be installed by `pip`:

```sh
curl https://github.com/sphinx-doc/sphinx/archive/refs/tags/v4.0.2.tar.gz -o /sources/sphinx-4.0.2.tar.gz &&

tar xzvf /sources/sphinx-4.0.2.tar.gz &&
cd        sphinx-4.0.2                &&

python3 setup.py build        &&
sudo python3 setup.py install &&

cd .. &&
rm -rf sphinx-4.0.2
```

### pandoc

`pandoc` has many dependencies.

```
Glob                  >= 0.7      && < 0.11,
HTTP                  >= 4000.0.5 && < 4000.4,
HsYAML                >= 0.2      && < 0.3,
JuicyPixels           >= 3.1.6.1  && < 3.4,
SHA                   >= 1.6      && < 1.7,
aeson                 >= 0.7      && < 1.6,
aeson-pretty          >= 0.8.5    && < 0.9,
array                 >= 0.5      && < 0.6,
attoparsec            >= 0.12     && < 0.15,
base64-bytestring     >= 0.1      && < 1.3,
binary                >= 0.7      && < 0.11,
blaze-html            >= 0.9      && < 0.10,
blaze-markup          >= 0.8      && < 0.9,
bytestring            >= 0.9      && < 0.12,
case-insensitive      >= 1.2      && < 1.3,
citeproc              >= 0.4      && < 0.4.1,
commonmark            >= 0.2      && < 0.3,
commonmark-extensions >= 0.2.1.2  && < 0.3,
commonmark-pandoc     >= 0.2.1    && < 0.3,
connection            >= 0.3.1,
containers            >= 0.4.2.1  && < 0.7,
data-default          >= 0.4      && < 0.8,
deepseq               >= 1.3      && < 1.5,
directory             >= 1.2.3    && < 1.4,
doclayout             >= 0.3.0.1  && < 0.4,
doctemplates          >= 0.9      && < 0.10,
emojis                >= 0.1      && < 0.2,
exceptions            >= 0.8      && < 0.11,
file-embed            >= 0.0      && < 0.1,
filepath              >= 1.1      && < 1.5,
haddock-library       >= 1.10     && < 1.11,
hslua                 >= 1.1      && < 1.4,
hslua-module-path     >= 0.1.0    && < 0.2.0,
hslua-module-system   >= 0.2      && < 0.3,
hslua-module-text     >= 0.2.1    && < 0.4,
http-client           >= 0.4.30   && < 0.8,
http-client-tls       >= 0.2.4    && < 0.4,
http-types            >= 0.8      && < 0.13,
ipynb                 >= 0.1      && < 0.2,
jira-wiki-markup      >= 1.4      && < 1.5,
mtl                   >= 2.2      && < 2.3,
network               >= 2.6,
network-uri           >= 2.6      && < 2.8,
pandoc-types          >= 1.22     && < 1.23,
parsec                >= 3.1      && < 3.2,
process               >= 1.2.3    && < 1.7,
random                >= 1        && < 1.3,
safe                  >= 0.3.18   && < 0.4,
scientific            >= 0.3      && < 0.4,
skylighting           >= 0.10.5.1 && < 0.10.6,
skylighting-core      >= 0.10.5.1 && < 0.10.6,
split                 >= 0.2      && < 0.3,
syb                   >= 0.1      && < 0.8,
tagsoup               >= 0.14.6   && < 0.15,
temporary             >= 1.1      && < 1.4,
texmath               >= 0.12.3   && < 0.12.4,
text                  >= 1.1.1.0  && < 1.3,
text-conversions      >= 0.3      && < 0.4,
time                  >= 1.5      && < 1.12,
unicode-transforms    >= 0.3      && < 0.4,
unordered-containers  >= 0.2      && < 0.3,
xml                   >= 1.3.12   && < 1.4,
xml-conduit           >= 1.9.1.1  && < 1.10,
unicode-collation     >= 0.1.1    && < 0.2,
zip-archive           >= 0.2.3.4  && < 0.5,
zlib                  >= 0.5      && < 0.7
```

```sh
curl https://hackage.haskell.org/package/pandoc-2.14.0.1/pandoc-2.14.0.1.tar.gz -o /sources/pandoc-2.14.0.1.tar.gz &&

tar xvzf /sources/pandoc-2.14.0.1.tar.gz &&
cd        pandoc-2.14.0.1                &&

cabal update                           &&
sudo cabal install --lib                       \
                   --prefix=/usr               \
                   --libdir=/usr/lib           \
                   --dynlibdir=/usr/lib        \
                   --global                    \
                   --enable-shared             \
                   --enable-optimization=2     \
                   --enable-executable-dynamic \
                   --disable-library-vanilla   \
                   --disable-static            \
                   --only-dependencies &&

cabal configure --prefix=/usr               \
                --global                    \
                --enable-executable-dynamic \
                --enable-optimization=2     \
                --docdir=/usr/share/doc/pandoc-2.14.0.1 &&

runghc Setup build -j      &&
rungch Setup test          &&
sudo runghc Setup copy     &&
sudo rungch Setup register &&

cd .. &&
rm -rf pandoc-2.14.0.1
```

```
th-compat-0.1.2
splitmix-0.1.0.3
random-1.2.0
hashable-1.3.2.0
async-2.2.3
base16-bytestring-0.1.1.7
base64-bytestring-1.0.0.3
digest-0.0.1.2
zlib-0.6.2.3
network-3.1.2.1
network-uri-2.6.4.1
```

```sh
sudo ghc-pkg unregister --force network-uri-2.6.4.1  &&
sudo cabal install --lib                       \
                   --prefix=/usr               \
                   --libdir=/usr/lib           \
                   --dynlibdir=/usr/lib        \
                   --global                    \
                   --enable-shared             \
                   --enable-optimization=2     \
                   --enable-executable-dynamic \
                   --disable-library-vanilla   \
                   --disable-static            \
                   --reinstall                 \
                   network-uri-2.6.4.1
```

### mdbook

```sh

```
