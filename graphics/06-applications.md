# Desktop applications

These are all optional and `GNOME` and other vendors offer plenty of other graphical desktop applications, but these are the minimum I personally prefer to have. In additional to the file manager, at least a terminal emulator and web browser is needed for most tasks.

To get instructions for obtaining and building desktop applications from Beyond Linux From Scratch for anything I have not included, check [GNOME Applications](https://www.linuxfromscratch.org/blfs/view/systemd/gnome/applications.html) and [X Software](https://www.linuxfromscratch.org/blfs/view/systemd/xsoft/xsoft.html).

You can also go direct to the source at https://download.gnome.org/sources/, which contains everything GNOME has ever released. All of the discontinued applications are also there, so if you really love old GNOME and want to give it a go, you can find older versions of everything and try to get them to build.

## GNOME Tweaks

Exposes settings no longer exposed through the standard GNOME settings app.

```sh
wget https://download.gnome.org/sources/gnome-tweaks/40/gnome-tweaks-40.0.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/gnome-tweaks-40.0.tar.xz &&
cd       gnome-tweaks-40.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gnome-tweaks-40.0
```

## GNOME Color Manager

```sh
wget https://download.gnome.org/sources/gnome-color-manager/3.36/gnome-color-manager-3.36.0.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/gnome-color-manager-3.36.0.tar.xz &&
cd       gnome-color-manager-3.36.0                      &&

sed /subdir\(\'man/d -i meson.build &&

mkdir build &&
cd    build &&

meson --prefix=/usr .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gnome-color-manager-3.36.0
```

## GNOME Terminal

This assumes you copied in the transparency patch, copied from the Arch User Repository. Remove the lines to apply the patch and run `autoreconf` if you don't want this.

```sh
wget https://download.gnome.org/sources/gnome-terminal/3.40/gnome-terminal-3.40.2.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/gnome-terminal-3.40.2.tar.xz &&
cd       gnome-terminal-3.40.2                      &&

patch -Np1 -i /sources/patches/gnome-terminal-3.40.2-transparency.patch &&

autoreconf -fiv &&

./configure --prefix=/usr    \
            --disable-static \
            --with-nautilus-extension &&

make              &&
sudo make install &&

cd .. &&
rm -rf gnome-terminal-3.40.2
```

Beware that the `LANG` environment variable has to be set to a UTF-8 value before `gnome-terminal` is launched or it will crash.

## pnmixer

Volume control with tray icon.

```sh
wget https://github.com/nicklan/pnmixer/releases/download/v0.7.2/pnmixer-v0.7.2.tar.gz -P /sources &&

tar xzvf /sources/pnmixer-v0.7.2.tar.gz &&
cd        pnmixer-v0.7.2                &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr .. &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf pnmixer-v0.7.2
```

## pavucontrol

`GTK+` volume control for `PulseAudio` that allows you to separately mix channels for multiple output devices.

```sh
wget https://freedesktop.org/software/pulseaudio/pavucontrol/pavucontrol-4.0.tar.xz -P /sources &&

tar xvf /sources/pavucontrol-4.0.tar.xz &&
cd       pavucontrol-4.0                &&

./configure --prefix=/usr \
            --docdir=/usr/share/doc/pavucontrol-4.0 &&

make              &&
sudo make install &&

cd .. &&
rm -rf pavucontrol-4.0
```

## Baobab

Graphical file tree analyzer.

```sh
wget https://download.gnome.org/sources/baobab/40/baobab-40.0.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/baobab-40.0.tar.xz &&
cd       baobab-40.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf baobab-40.0
```

## File Roller

Archive manager for `GNOME`.

```sh
wget https://download.gnome.org/sources/file-roller/3.40/file-roller-3.40.0.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/file-roller-3.40.0.tar.xz &&
cd       file-roller-3.40.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Dpackagekit=false .. &&

ninja              &&
sudo ninja install &&

sudo chmod -v 0755 /usr/libexec/file-roller/isoinfo.sh     &&
sudo gtk-update-icon-cache   -qtf /usr/share/icons/hicolor &&
sudo update-desktop-database -q                            &&

cd ../.. &&
rm -rf file-roller-3.40.0
```

## EOG

Image viewer with basic editing capabilities.

```sh
wget https://download.gnome.org/sources/eog/40/eog-40.2.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/eog-40.2.tar.xz &&
cd       eog-40.2                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Dlibportal=false .. &&

ninja              &&
sudo ninja install &&

sudo update-desktop-database &&

cd ../.. &&
rm -rf eog-40.2
```

## GNOME Screenshot

```sh
wget https://download.gnome.org/sources/gnome-screenshot/40/gnome-screenshot-40.0.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/gnome-screenshot-40.0.tar.xz &&
cd       gnome-screenshot-40.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gnome-screenshot-40.0
```

## GtkSourceView

Syntax highlighting for GTK text functions.

```sh
wget https://download.gnome.org/sources/gtksourceview/4.8/gtksourceview-4.8.1.tar.xz -P /sources/gnome &&
wget https://www.linuxfromscratch.org/patches/blfs/svn/gtksourceview4-4.8.1-buildfix-1.patch -P /sources/patches &&

tar xvf /sources/gnome/gtksourceview-4.8.1.tar.xz &&
cd       gtksourceview-4.8.1                      &&

patch -Np1 -i /sources/patches/gtksourceview-4.8.1-buildfix-1.patch &&

mkdir build &&
cd    build &&

meson --prefix=/usr .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gtksourceview-4.8.1
```

## gspell

Libraries for adding spell checking to GTK+ applications.

```sh
wget https://download.gnome.org/sources/gspell/1.8/gspell-1.8.4.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/gspell-1.8.4.tar.xz &&
cd       gspell-1.8.4                      &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf gspell-1.8.4
```

## Evince

Document viewer.

```sh
wget https://download.gnome.org/sources/evince/40/evince-40.2.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/evince-40.2.tar.xz &&
cd       evince-40.2                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr       \
      --buildtype=release \
      -Dgtk_doc=false .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf evince-40.2
```

## GNOME Calculator

```sh
wget https://download.gnome.org/sources/gnome-calculator/40/gnome-calculator-40.1.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/gnome-calculator-40.1.tar.xz &&
cd       gnome-calculator-40.1                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gnome-calculator-40.1
```

## GNOME Disk Utility

```sh
wget https://download.gnome.org/sources/gnome-disk-utility/40/gnome-disk-utility-40.1.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/gnome-disk-utility-40.1.tar.xz &&
cd       gnome-disk-utility-40.1                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gnome-disk-utility-40.1
```

## GNOME Books

```sh
wget https://download.gnome.org/sources/gnome-books/40/gnome-books-40.0.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/gnome-books-40.0.tar.xz &&
cd       gnome-books-40.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gnome-books-40.0
```

## GNOME Calendar

```sh
wget https://download.gnome.org/sources/gnome-calendar/40/gnome-calendar-40.2.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/gnome-calendar-40.2.tar.xz &&
cd       gnome-calendar-40.2                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gnome-calendar-40.2
```

## GNOME Music

```sh
wget https://download.gnome.org/sources/gnome-music/40/gnome-music-40.1.1.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/gnome-music-40.1.1.tar.xz &&
cd       gnome-music-40.1.1                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gnome-music-40.1.1
```

## GNOME Photos

```sh
wget https://download.gnome.org/sources/gnome-photos/40/gnome-photos-40.0.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/gnome-photos-40.0.tar.xz &&
cd       gnome-photos-40.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr  .. &&

ninja              &&
sudo ninja install &&

sudo mv /usr/share/doc/gnome-photos /usr/share/doc/gnome-photos-40.0 &&

cd ../.. &&
rm -rf gnome-photos-40.0
```

## GNOME TODO

```sh
wget https://download.gnome.org/sources/gnome-todo/40/gnome-todo-40.0.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/gnome-todo-40.0.tar.xz &&
cd       gnome-todo-40.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr  .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gnome-todo-40.0
```

## GNOME Weather

```sh
wget https://download.gnome.org/sources/gnome-weather/40/gnome-weather-40.0.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/gnome-weather-40.0.tar.xz &&
cd       gnome-weather-40.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr  .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gnome-weather-40.0
```

## Avahi Client

Link-local service discovery. Since `systemd-resolved` handles `mDNS` perfectly well on its own, I am only installing the client since `Seahorse` requires it.

Create a group for privileged access to `Avahi` clients:

```sh
sudo groupadd -fg 86 netdev
```

```sh
wget https://github.com/lathiat/avahi/releases/download/v0.8/avahi-0.8.tar.gz -P /sources &&
wget https://www.linuxfromscratch.org/patches/blfs/svn/avahi-0.8-ipv6_race_condition_fix-1.patch -P /sources/patches &&

tar xzvf /sources/avahi-0.8.tar.gz &&
cd        avahi-0.8                &&

patch -Np1 -i /sources/patches/avahi-0.8-ipv6_race_condition_fix-1.patch &&

./configure --prefix=/usr        \
            --sysconfdir=/etc    \
            --localstatedir=/var \
            --disable-static     \
            --disable-libdaemon  \
            --disable-mono       \
            --disable-monodoc    \
            --disable-python     \
            --disable-qt3        \
            --disable-qt4        \
            --disable-qt5        \
            --enable-core-docs   \
            --with-distro=none   &&

make              &&
sudo make install &&

cd .. &&
rm -rf avahi-0.8
```

## Seahorse

Graphical interface for managing and using encryption keys.

```sh
wget https://download.gnome.org/sources/seahorse/40/seahorse-40.0.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/seahorse-40.0.tar.xz &&
cd       seahorse-40.0                      &&

sed -i -r 's:"(/apps):"/org/gnome\1:' data/*.xml &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf seahorse-40.0
```

## Epiphany

The `GNOME` web browser.

```sh
wget https://download.gnome.org/sources/epiphany/40/epiphany-40.2.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/epiphany-40.2.tar.xz &&
cd       epiphany-40.2                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf epiphany-40.2
```

## Totem

The `GNOME` video player.

```sh
wget https://download-fallback.gnome.org/sources/totem/3.38/totem-3.38.1.tar.xz -P /sources/gnome &&

tar xvf /sources/gnome/totem-3.38.1.tar.xz &&
cd       totem-3.38.1                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
sudo rm -rf totem-3.38.1
```
