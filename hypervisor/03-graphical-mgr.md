# Graphical guest managers

## gtk-vnc

VNC viewer widget for `GTK+`.

```sh
curl https://download.gnome.org/sources/gtk-vnc/1.2/gtk-vnc-1.2.0.tar.xz -o /sources/gtk-vnc-1.2.0.tar.xz &&

tar xvf /sources/gtk-vnc-1.2.0.tar.xz &&
cd       gtk-vnc-1.2.0                &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      -Dbuildtype=release \
      -Db_lto=true &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gtk-vnc-1.2.0
```

## libvirt-glib

`GLib` bindings for `libvirt`.

```sh
curl https://libvirt.org/sources/glib/libvirt-glib-4.0.0.tar.xz -o /sources/libvirt-glib-4.0.0.tar.xz &&

tar xvf /sources/libvirt-glib-4.0.0.tar.xz &&
cd       libvirt-glib-4.0.0                &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      -Dbuildtype=release \
      -Db_lto=true &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libvirt-glib-4.0.0
```

## pyparsing

```sh
curl https://files.pythonhosted.org/packages/c1/47/dfc9c342c9842bbe0036c7f763d2d6686bcf5eb1808ba3e170afdb282210/pyparsing-2.4.7.tar.gz -o /sources/python/python-pyparsing-2.4.7.tar.gz &&

tar xzvf /sources/python/python-pyparsing-2.4.7.tar.gz &&
cd        pyparsing-2.4.7                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf pyparsing-2.4.7
```

## libvirt-python

```sh
curl https://files.pythonhosted.org/packages/da/d3/769b2bc22568da32d1d48ebbbf3c907f3bdbd8061f5aa02bd5d61c09efdc/libvirt-python-7.4.0.tar.gz -o /sources/python/libvirt-python-7.4.0.tar.gz &&

tar xzvf /sources/python/libvirt-python-7.4.0.tar.gz &&
cd        libvirt-python-7.4.0                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf libvirt-python-7.4.0
```

## spice-protocol

Headers for `spice`.

```sh
curl https://gitlab.freedesktop.org/spice/spice-protocol/-/archive/v0.14.3/spice-protocol-v0.14.3.tar.gz -o /sources/spice-protocol-v0.14.3.tar.gz &&

tar xzvf /sources/spice-protocol-v0.14.3.tar.gz &&
cd        spice-protocol-v0.14.3                &&

mkdir build &&
cd    build &&

meson --prefix=/usr .. &&

sudo ninja install &&

cd ../.. &&
rm -rf spice-protocol-v0.14.3
```

## spice-gtk

```sh
curl https://www.spice-space.org/download/gtk/spice-gtk-0.39.tar.xz -o /sources/spice-gtk-0.39.tar.xz &&

tar xvf /sources/spice-gtk-0.39.tar.xz &&
cd       spice-gtk-0.39                &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      -Dbuildtype=release \
      -Db_lto=true        \
      -Dgtk-doc=false .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf spice-gtk-0.39
```

## osinfo-db-tools

```sh
curl https://releases.pagure.org/libosinfo/osinfo-db-tools-1.9.0.tar.xz -o /sources/osinfo-db-tools-1.9.0.tar.xz &&
curl https://releases.pagure.org/libosinfo/osinfo-db-20210531.tar.xz -o /sources/osinfo-db-20210531.tar.xz &&

tar xvf /sources/osinfo-db-tools-1.9.0.tar.xz &&
cd       osinfo-db-tools-1.9.0                &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      -Dbuildtype=release \
      -Db_lto=true .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf osinfo-db-tools-1.9.0
```

Then import the os data:

```sh
sudo osinfo-db-import --system /sources/osinfo-db-20210531.tar.xz
```

## libosinfo

`GObject` based library for managing osinfo.

```sh
curl https://releases.pagure.org/libosinfo/libosinfo-1.9.0.tar.xz -o /sources/libosinfo-1.9.0.tar.xz &&

tar xvf /sources/libosinfo-1.9.0.tar.xz &&
cd       libosinfo-1.9.0                &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      -Dbuildtype=release \
      -Db_lto=true ..

ninja              &&
sudo ninja install &&

sudo install -vdm755 /usr/share/doc/libosinfo-1.9.0            &&
sudo cp -av docs/reference/html /usr/share/doc/libosinfo-1.9.0 &&

cd ../.. &&
sudo rm -rf libosinfo-1.9.0
```

## virt-manager

```sh
curl https://virt-manager.org/download/sources/virt-manager/virt-manager-3.2.0.tar.gz -o /sources/virt-manager-3.2.0.tar.gz &&

tar xzvf /sources/virt-manager-3.2.0.tar.gz &&
cd        virt-manager-3.2.0                &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf virt-manager-3.2.0
```

## GNOME Boxes

`GNOME` graphical VM manager.

```sh
curl https://download-fallback.gnome.org/sources/gnome-boxes/40/gnome-boxes-40.2.tar.xz -o /sources/gnome/gnome-boxes-40.2.tar.xz &&

tar xvf /sources/gnome/gnome-boxes-40.2.tar.xz &&
cd       gnome-boxes-40.2                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      -Dbuildtype=release \
      -Db_lto=true .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
sudo rm -rf gnome-boxes-40.2
```
