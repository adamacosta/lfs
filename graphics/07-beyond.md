# Beyond GNOME

## GIMP and dependencies

### libmypaint

```sh
wget https://github.com/mypaint/libmypaint/releases/download/v1.6.1/libmypaint-1.6.1.tar.xz -P /sources &&

tar xvf /sources/libmypaint-1.6.1.tar.xz &&
cd       libmypaint-1.6.1                &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf libmypaint-1.6.1
```

### mypaint-brushes

```sh
wget https://github.com/Jehan/mypaint-brushes/archive/v1.3.0/mypaint-brushes-v1.3.0.tar.gz -P /sources &&
wget https://www.linuxfromscratch.org/patches/blfs/svn/mypaint-brushes-1.3.0-automake_1.16-1.patch -P /sources/patches &&

tar xzvf /sources/mypaint-brushes-v1.3.0.tar.gz &&
cd        mypaint-brushes-1.3.0                 &&

patch -Np1 -i /sources/patches/mypaint-brushes-1.3.0-automake_1.16-1.patch &&

./autogen.sh              &&
./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf mypaint-brushes-1.3.0
```

### GIMP

The `GNU` Image Manipulation Program. Note that I am choosing to disable `Python` as it appears that `GIMP` does not yet support `Python3` and I do not wish to install `Python2`, which is now more than a year and a half past its EOL.

```sh
wget https://download.gimp.org/pub/gimp/v2.10/gimp-2.10.24.tar.bz2 -P /sources &&

tar xvf /sources/gimp-2.10.24.tar.bz2 &&
cd       gimp-2.10.24                 &&

./configure --prefix=/usr     \
            --sysconfdir=/etc \
            --disable-python &&

make              &&
sudo make install &&

sudo gtk-update-icon-cache -qtf /usr/share/icons/hicolor &&
sudo update-desktop-database -q &&

cd .. &&
rm -rf gimp-2.10.24
```

## VLC Player and dependencies

### lua-5.2

`VLC Player` does not yet support building with `lua-5.4`, so we need to install an older version first.

```sh
wget https://www.lua.org/ftp/lua-5.2.4.tar.gz -P /sources &&
wget https://www.linuxfromscratch.org/patches/blfs/svn/lua-5.2.4-shared_library-1.patch -P /sources/patches &&

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

### VLC Player

```sh
wget https://download.videolan.org/vlc/3.0.15/vlc-3.0.15.tar.xz -P /sources &&

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
wget https://archive.mozilla.org/pub/firefox/releases/89.0.1/source/firefox-89.0.1.source.tar.xz -P /sources &&

tar xvf /sources/firefox-89.0.1.source.tar.xz &&
cd       firefox-89.0.1                       &&
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
rm -rf firefox-89.0.1
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
