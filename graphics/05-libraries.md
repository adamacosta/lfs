# Graphics libraries

## babl

Pixel format conversion library.

```sh
curl https://download.gimp.org/pub/babl/0.1/babl-0.1.86.tar.xz -o /sources/babl-0.1.86.tar.xz &&

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
curl https://download.gimp.org/pub/gegl/0.4/gegl-0.4.30.tar.xz -o /sources/gegl-0.4.30.tar.xz &&

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
curl https://download.gnome.org/sources/libdazzle/3.40/libdazzle-3.40.0.tar.xz -o /sources/gnome/libdazzle-3.40.0.tar.xz &&

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
curl https://download.gnome.org/sources/gfbgraph/0.2/gfbgraph-0.2.4.tar.xz -o /sources/gnome/gfbgraph-0.2.4.tar.xz &&

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
curl https://github.com/GNOME/libadwaita/archive/refs/tags/1.0.0-alpha.1.tar.gz -o /sources/gnome/libadwaita-1.0.0-alpha.1.tar.gz &&

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
curl https://download.gnome.org/sources/libmediaart/1.9/libmediaart-1.9.5.tar.xz -o /sources/gnome/libmediaart-1.9.5.tar.xz &&

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
curl https://download.gnome.org/sources/grilo-plugins/0.3/grilo-plugins-0.3.9.tar.xz -o /sources/gnome/grilo-plugins-0.3.9.tar.xz &&

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
curl https://download.gnome.org/sources/libgepub/0.6/libgepub-0.6.0.tar.xz -o /sources/gnome/libgepub-0.6.0.tar.xz &&

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
