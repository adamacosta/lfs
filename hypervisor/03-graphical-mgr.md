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

## virt-manager



























