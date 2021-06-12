# System configuration

## kmod

Rebuild with `Python` bindings and ensure `pkg-config` picks up the `libkmod.pc` file, as `systemd` will not be able to find it otherwise.

```sh
tar xvf /sources/kmod-28.tar.xz &&
cd       kmod-28                &&

./configure --prefix=/usr          \
            --bindir=/bin          \
            --sysconfdir=/etc      \
            --with-rootlibdir=/lib \
            --with-xz              \
            --with-zstd            \
            --with-zlib            \
            --enable-python &&

make                                   &&
sudo make install-libLTLIBRARIES       &&
sudo make install-binPROGRAMS          &&
sudo make install-pkgconfigDATA        &&
sudo make install-includeHEADERS       &&
sudo make install-pkgpyexecLTLIBRARIES &&
sudo make install-dist_pkgpyexecPYTHON &&

sudo cp -iv man/*.8 /usr/share/man/man8/ &&
sudo cp -iv man/*.5 /usr/share/man/man5/ &&

cd .. &&
rm -rf kmod-28
```

## systemd rebuild

At this point, `systemd` should be rebuilt to support all of the additional tools and libraries now available that were not available when the base system was being created. It will also be configured to use `PAM` correctly for `systemd-logind`. We will also go ahead and take advantage of better compiler optimization levels and link-time optimization, which Linux From Scratch does not do.

```sh
tar xvf /sources/systemd-248.tar.gz &&
cd       systemd-248                &&

patch -Np1 -i /sources/systemd-248-upstream_fixes-1.patch                 &&
sed -i 's/GROUP="render"/GROUP="video"/' rules.d/50-udev-default.rules.in &&

mkdir -v build &&
cd       build &&

meson --prefix=/usr                       \
      --sysconfdir=/etc                   \
      --localstatedir=/var                \
      --buildtype=release                 \
      --optimization=2                    \
      -Dblkid=true                        \
      -Db_lto=true                        \
      -Dfirstboot=false                   \
      -Dinstall-tests=false               \
      -Dldconfig=false                    \
      -Dman=auto                          \
      -Dsysusers=false                    \
      -Drpmmacrosdir=no                   \
      -Dhomed=false                       \
      -Duserdb=false                      \
      -Dkmod=true                         \
      -Dkmod-path=/usr/bin/kmod           \
      -Dmode=release                      \
      -Dpamconfdir=/etc/pam.d             \
      -Ddocdir=/usr/share/doc/systemd-248 \
      .. &&

ninja              &&
sudo ninja install &&

cd .. &&
rm -rf systemd-248
```

Now, if you have a machine using WiFi to connect to the network, ensure that the kernel modules for WiFi drivers are loaded. Substitute in the appropriate driver for your WiFi card.

```sh
cat > /usr/lib/modules-load.d/iwd.conf << EOF
mac80211
cfg80211
iwlwifi
EOF
```

Update the `PAM` configuration for `systemd-logind`.

```sh
cat >> /etc/pam.d/system-session << "EOF"
# Begin Systemd addition

session  required    pam_loginuid.so
session  optional    pam_systemd.so

# End Systemd addition
EOF

cat > /etc/pam.d/systemd-user << "EOF"
# Begin /etc/pam.d/systemd-user

account  required    pam_access.so
account  include     system-account

session  required    pam_env.so
session  required    pam_limits.so
session  required    pam_unix.so
session  required    pam_loginuid.so
session  optional    pam_keyinit.so force revoke
session  optional    pam_systemd.so

auth     required    pam_deny.so
password required    pam_deny.so

# End /etc/pam.d/systemd-user
EOF
```

Now is a good time to reboot the system and see that everything starts up as expected.

## Console fonts

Install the psf fonts from `TerminusFont`.

```sh
curl http://sourceforge.net/projects/terminus-font/files/terminus-font-4.49/terminus-font-4.49.1.tar.gz/download -o /sources/terminus-font-4.49.1.tar.gz &&

tar xzvf /sources/terminus-font-4.49.1.tar.gz &&
cd        terminus-font-4.49.1                &&

make psf
```

Try out a few `sudo setfont <font>.psf` and install whichever you like:

```sh
gzip <font>.psf &&
sudo install -vm644 <font>.psf.gz /usr/share/consolefonts/
```

To install all of them:

```sh
find . -name "*.psf" -exec gzip {} ;\
sudo install -vm644 *.gz /usr/share/consolefonts/
```

Then set the one you like as the default:

```sh
cat > /etc/vconsole.conf << "EOF"
FONT=ter-h18b
EOF
```
