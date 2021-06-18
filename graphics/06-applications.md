# Desktop applications

These are all optional and `GNOME` and other vendors offer plenty of other graphical desktop applications, but these are the minimum I personally prefer to have. In additional to the file manager, at least a terminal emulator and web browser is needed for most tasks.

To get instructions for obtaining and building desktop applications from Beyond Linux From Scratch for anything I have not included, check [GNOME Applications](https://www.linuxfromscratch.org/blfs/view/systemd/gnome/applications.html) and [X Software](https://www.linuxfromscratch.org/blfs/view/systemd/xsoft/xsoft.html).

## GNOME Tweaks

```sh
curl https://download.gnome.org/sources/gnome-tweaks/40/gnome-tweaks-40.0.tar.xz -o /sources/gnome/gnome-tweaks-40.0.tar.xz &&

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
curl https://download.gnome.org/sources/gnome-color-manager/3.36/gnome-color-manager-3.36.0.tar.xz -o /sources/gnome/gnome-color-manager-3.36.0.tar.xz &&

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

This assumes you copied in the transparency patch, copied from the Arch User Repository. Remove the lines to apply the patch run `autoreconf` if you don't want this.

```sh
curl https://download.gnome.org/sources/gnome-terminal/3.40/gnome-terminal-3.40.2.tar.xz -o /sources/gnome/gnome-terminal-3.40.2.tar.xz &&

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
curl https://github.com/nicklan/pnmixer/releases/download/v0.7.2/pnmixer-v0.7.2.tar.gz -o /sources/pnmixer-v0.7.2.tar.gz &&

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

## Baobab

Graphical file tree analyzer.

```sh
curl https://download.gnome.org/sources/baobab/40/baobab-40.0.tar.xz -o /sources/gnome/baobab-40.0.tar.xz &&

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
curl https://download.gnome.org/sources/file-roller/3.40/file-roller-3.40.0.tar.xz -o /sources/gnome/file-roller-3.40.0.tar.xz &&

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
curl https://download.gnome.org/sources/eog/40/eog-40.2.tar.xz -o /sources/gnome/eog-40.2.tar.xz &&

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
curl https://download.gnome.org/sources/gnome-screenshot/40/gnome-screenshot-40.0.tar.xz -o /sources/gnome/gnome-screenshot-40.0.tar.xz &&

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
curl https://download.gnome.org/sources/gtksourceview/4.8/gtksourceview-4.8.1.tar.xz -o /sources/gnome/gtksourceview-4.8.1.tar.xz &&
curl https://www.linuxfromscratch.org/patches/blfs/svn/gtksourceview4-4.8.1-buildfix-1.patch -o /sources/patches/gtksourceview-4.8.1-buildfix-1.patch &&

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
curl https://download.gnome.org/sources/gspell/1.8/gspell-1.8.4.tar.xz -o /sources/gnome/gspell-1.8.4.tar.xz &&

tar xvf /sources/gnome/gspell-1.8.4.tar.xz &&
cd       gspell-1.8.4                      &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf gspell-1.8.4
```

## Evince

Document viewer

```sh
curl https://download.gnome.org/sources/evince/40/evince-40.2.tar.xz -o /sources/gnome/evince-40.2.tar.xz &&

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
curl https://download.gnome.org/sources/gnome-calculator/40/gnome-calculator-40.1.tar.xz -o /sources/gnome/gnome-calculator-40.1.tar.xz &&

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
curl https://download.gnome.org/sources/gnome-disk-utility/40/gnome-disk-utility-40.1.tar.xz -o /sources/gnome/gnome-disk-utility-40.1.tar.xz &&

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
curl https://download.gnome.org/sources/gnome-books/40/gnome-books-40.0.tar.xz -o /sources/gnome/gnome-books-40.0.tar.xz &&

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
curl https://download.gnome.org/sources/gnome-calendar/40/gnome-calendar-40.2.tar.xz -o /sources/gnome/gnome-calendar-40.2.tar.xz &&

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
curl https://download.gnome.org/sources/gnome-music/40/gnome-music-40.1.1.tar.xz -o /sources/gnome/gnome-music-40.1.1.tar.xz &&

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
curl https://download.gnome.org/sources/gnome-photos/40/gnome-photos-40.0.tar.xz -o /sources/gnome/gnome-photos-40.0.tar.xz &&

tar xvf /sources/gnome/gnome-photos-40.0.tar.xz &&
cd       gnome-photos-40.0                      &&

mkdir build &&
cd    build &&

meson --prefix=/usr  .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf gnome-photos-40.0
```

## GNOME TODO

```sh
curl https://download.gnome.org/sources/gnome-todo/40/gnome-todo-40.0.tar.xz -o /sources/gnome/gnome-todo-40.0.tar.xz &&

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
curl https://download.gnome.org/sources/gnome-weather/40/gnome-weather-40.0.tar.xz -o /sources/gnome/gnome-weather-40.0.tar.xz &&

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
curl https://github.com/lathiat/avahi/releases/download/v0.8/avahi-0.8.tar.gz -o /sources/avahi-0.8.tar.gz &&
curl https://www.linuxfromscratch.org/patches/blfs/svn/avahi-0.8-ipv6_race_condition_fix-1.patch -o /sources/patches/avahi-0.8-ipv6_race_condition_fix-1.patch &&

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
curl https://download.gnome.org/sources/seahorse/40/seahorse-40.0.tar.xz -o /sources/gnome/seahorse-40.0.tar.xz &&

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
curl https://download.gnome.org/sources/epiphany/40/epiphany-40.2.tar.xz -o /sources/gnome/epiphany-40.2.tar.xz &&

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
curl https://download-fallback.gnome.org/sources/totem/3.38/totem-3.38.1.tar.xz -o /sources/gnome/totem-3.38.1.tar.xz &&

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

## VLC Player

Media player that recognizes virtually every format known to man. Does not yet support building with `lua-5.4`, so we need to install an older version first. Also requires `Qt5` for the graphical interface.

```sh
curl https://www.lua.org/ftp/lua-5.2.4.tar.gz -o /sources/lua-5.2.4.tar.gz &&
curl https://www.linuxfromscratch.org/patches/blfs/svn/lua-5.2.4-shared_library-1.patch -o /sources/patches/lua-5.2.4-shared_library-1.patch &&

tar xzvf /sources/lua-5.2.4.tar.gz &&
cd        lua-5.2.4                &&

patch -Np1 -i /sources/patches/lua-5.2.4-shared_library-1.patch &&

sed -i '/#define LUA_ROOT/s:/usr/local/:/usr/:' src/luaconf.h &&

sed -r -e '/^LUA_(SO|A|T)=/ s/lua/lua5.2/' \
       -e '/^LUAC_T=/ s/luac/luac5.2/'     \
       -i src/Makefile &&

make MYCFLAGS="-fPIC" linux &&

make TO_BIN='lua5.2 luac5.2'                     \
     TO_LIB="liblua5.2.so liblua5.2.so.5.2 liblua5.2.so.5.2.4" \
     INSTALL_DATA="cp -d"                        \
     INSTALL_TOP=$PWD/install/usr                \
     INSTALL_INC=$PWD/install/usr/include/lua5.2 \
     INSTALL_MAN=$PWD/install/usr/share/man/man1 \
     install &&

mkdir -pv install/usr/share/doc/lua-5.2.4 &&
cp -v doc/*.{html,css,gif,png} install/usr/share/doc/lua-5.2.4 &&

ln -s liblua5.2.so install/usr/lib/liblua.so.5.2   &&
ln -s liblua5.2.so install/usr/lib/liblua.so.5.2.4 &&

mv install/usr/share/man/man1/{lua.1,lua5.2.1}   &&
mv install/usr/share/man/man1/{luac.1,luac5.2.1} &&

sudo chown -R root:root install  &&
sudo cp    -a install/* /        &&

cd .. &&
rm -rf lua-5.2.4
```

Then create a `pkg-config` file:

```sh
sudo su -
cat > /usr/lib/pkgconfig/lua52.pc << "EOF"
V=5.2
R=5.2.4

prefix=/usr
INSTALL_BIN=${prefix}/bin
INSTALL_INC=${prefix}/include/lua5.2
INSTALL_LIB=${prefix}/lib
INSTALL_MAN=${prefix}/share/man/man1
INSTALL_LMOD=${prefix}/share/lua/${V}
INSTALL_CMOD=${prefix}/lib/lua/${V}
exec_prefix=${prefix}
libdir=${exec_prefix}/lib
includedir=${prefix}/include/lua5.2

Name: Lua
Description: An Extensible Extension Language
Version: ${R}
Requires:
Libs: -L${libdir} -llua5.2 -lm -ldl
Cflags: -I${includedir}
EOF
exit
```

Now intall `vlc-player`:

```sh
curl https://download.videolan.org/vlc/3.0.15/vlc-3.0.15.tar.xz -o /sources/vlc-3.0.15.tar.xz &&

tar xvf /sources/vlc-3.0.15.tar.xz &&
cd       vlc-3.0.15                &&

export LUAC=/usr/bin/luac5.2                   &&
export LUA_LIBS="$(pkg-config --libs lua52)"   &&
export CPPFLAGS="$(pkg-config --cflags lua52)" &&

BUILDCC=gcc ./configure --prefix=/usr \
                        --disable-opencv \
                        --disable-vpx &&

make                                               &&
sudo make docdir=/usr/share/doc/vlc-3.0.15 install &&

cd .. &&
rm -rf vlc-3.0.15
```

To update the icon and desktop caches:

```sh
sudo gtk-update-icon-cache   -qtf /usr/share/icons/hicolor &&
sudo update-desktop-database -q
```

## Firefox

```sh
curl https://archive.mozilla.org/pub/firefox/releases/89.0/source/firefox-89.0.source.tar.xz -o /sources/firefox-89.0.source.tar.xz &&

tar xvf /sources/firefox-89.0.source.tar.xz &&
cd       firefox-89.0                       &&
```

Beware that the `Firefox` tarball is jacked up and changes ownership of the directory it is run in and changes the sticky bit. To restore:

```sh
sudo chown lfs:lfs /sources/build_dir &&
chmod      0755    /sources/build_dir
```

`Firefox` requires a special build configuration file:

```sh
cat > mozconfig << "EOF"
mk_add_options MOZ_OBJDIR=@TOPSRCDIR@/obj-release-dir

# If you have installed (or will install) wireless-tools, and you wish
# to use geolocation web services, comment out this line
ac_add_options --disable-necko-wifi

# API Keys for geolocation APIs - necko-wifi (above) is required for MLS
# Uncomment the following line if you wish to use Mozilla Location Service
#ac_add_options --with-mozilla-api-keyfile=$PWD/mozilla-key

# Uncomment the following line if you wish to use Google's geolocaton API
# (needed for use with saved maps with Google Maps)
#ac_add_options --with-google-location-service-api-keyfile=$PWD/google-key

# You cannot distribute the binary if you do this
ac_add_options --enable-official-branding

# The BLFS editors recommend not changing anything below this line:
ac_add_options --prefix=/usr
ac_add_options --enable-application=browser

ac_add_options --enable-release
ac_add_options --enable-optimize
ac_add_options --enable-hardening
ac_add_options --enable-rust-simd

ac_add_options --disable-crashreporter
ac_add_options --disable-updater
ac_add_options --disable-tests
ac_add_options --disable-debug
ac_add_options --disable-elf-hack

ac_add_options --enable-system-ffi
ac_add_options --enable-system-pixman

ac_add_options --with-system-libevent
ac_add_options --with-system-webp
ac_add_options --with-system-nspr
ac_add_options --with-system-nss
ac_add_options --with-system-icu
ac_add_options --with-system-jpeg
ac_add_options --with-system-png
ac_add_options --with-system-zlib

# The following option unsets Telemetry Reporting. With the Addons Fiasco,
# Mozilla was found to be collecting user's data, including saved passwords and
# web form data, without users consent. Mozilla was also found shipping updates
# to systems without the user's knowledge or permission.
# As a result of this, use the following command to permanently disable
# telemetry reporting in Firefox.
unset MOZ_TELEMETRY_REPORTING
EOF
```

Then build and install. I have opted to use a mix of settings from both BLFS and the Arch Linux official `pacman` package build. In particular, I have not gotten `Firefox-89.0` to build successfully with `gcc-11.1.0` due to an appparent incompatibility in `libstdc++`.

```sh
export MOZBUILD_STATE_PATH=${PWD}/mozbuild &&
export MACH_USE_SYSTEM_PYTHON=1            &&
./mach configure                           &&
./mach build                               &&
sudo ./mach install                        &&
unset CC CXX MOZBUILD_STATE_PATH           &&

cd .. &&
rm -rf firefox-89.0
```

Create the .desktop file for your desktop environment:

```sh
sudo su -
mkdir -pv /usr/share/applications &&
mkdir -pv /usr/share/pixmaps      &&

cat > /usr/share/applications/firefox.desktop << "EOF" &&
[Desktop Entry]
Encoding=UTF-8
Name=Firefox Web Browser
Comment=Browse the World Wide Web
GenericName=Web Browser
Exec=firefox %u
Terminal=false
Type=Application
Icon=firefox
Categories=GNOME;GTK;Network;WebBrowser;
MimeType=application/xhtml+xml;text/xml;application/xhtml+xml;application/vnd.mozilla.xul+xml;text/mml;x-scheme-handler/http;x-scheme-handler/https;
StartupNotify=true
EOF

ln -sfv /usr/lib/firefox/browser/chrome/icons/default/default128.png \
        /usr/share/pixmaps/firefox.png
exit
```

Create some default application config settings:

```sh
sudo install -Dvm644 /dev/stdin \
    /usr/lib/firefox/browser/defaults/preferences/vendor.js <<EOF
// Use LANG environment variable to choose locale
pref("intl.locale.requested", "");
// Use system-provided dictionaries
pref("spellchecker.dictionary_path", "/usr/share/dict/words");
// Disable default browser checking.
pref("browser.shell.checkDefaultBrowser", false);
// Don't disable extensions in the application directory
pref("extensions.autoDisableScopes", 11);
EOF
```

Install the official Firefox icons:

```sh
for i in 16 22 24 32 48 64 128 256; do
sudo install -Dvm644 browser/branding/official/default$i.png \
  /usr/share/icons/hicolor/${i}x${i}/apps/firefox.png
done
sudo install -Dvm644 browser/branding/official/content/about-logo.png \
    /usr/share/icons/hicolor/192x192/apps/firefox.png
sudo install -Dvm644 browser/branding/official/content/about-logo@2x.png \
    /usr/share/icons/hicolor/384x384/apps/firefox.png
sudo install -Dvm644 browser/branding/official/content/identity-icons-brand.svg \
    /usr/share/icons/hicolor/symbolic/apps/firefox-symbolic.svg
```
