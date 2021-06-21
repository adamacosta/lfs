# General utility software

## rsync

Synchornize different file trees, indifferent to whether the target is local or remote. This will be built without support for the daemon.

```sh
wget https://www.samba.org/ftp/rsync/src/rsync-3.2.3.tar.gz -P /sources &&

tar xzvf /sources/rsync-3.2.3.tar.gz &&
cd        rsync-3.2.3                &&

./configure --prefix=/usr    \
            --disable-lz4    \
            --disable-xxhash \
            --without-included-zlib &&

make              &&
sudo make install &&

cd .. &&
rm -rf rsync-3.2.3
```

## pigz

Parallel drop-in replacement for `gzip`.

```sh
wget https://zlib.net/pigz/pigz-2.6.tar.gz -P /sources &&

tar xzvf /sources/pigz-2.6.tar.gz &&
cd        pigz-2.6                &&

make                               &&
make test                          &&
sudo install -m755 pigz /usr/bin   &&
sudo install -m755 unpigz /usr/bin &&

cd .. &&
rm -rf pigz-2.6
```

## pbzip2

Parallel drop-in replacement for `bzip2`.

```sh
wget https://launchpad.net/pbzip2/1.1/1.1.13/+download/pbzip2-1.1.13.tar.gz -P /sources &&

tar xzvf /sources/pbzip2-1.1.13.tar.gz &&
cd        pbzip2-1.1.13                &&

make                               &&
sudo install -m755 pbzip2 /usr/bin &&

cd .. &&
rm -rf pbzip2-1.1.13
```

## Zip

```sh
wget https://downloads.sourceforge.net/infozip/zip30.tar.gz -P /sources &&

tar xzvf /sources/zip30.tar.gz &&
cd        zip30                &&

make -f unix/Makefile generic_gcc                                         &&
sudo make prefix=/usr MANDIR=/usr/share/man/man1 -f unix/Makefile install &&

cd .. &&
rm -rf zip30
```

## Unzip

```sh
wget https://downloads.sourceforge.net/infozip/unzip60.tar.gz -P /sources &&
wget http://www.linuxfromscratch.org/patches/blfs/10.1/unzip-6.0-consolidated_fixes-1.patch -P /sources/patches &&

tar xzvf /sources/unzip60.tar.gz &&
cd        unzip60                &&

patch -Np1 -i /sources/unzip-6.0-consolidated_fixes-1.patch &&

make -f unix/Makefile generic &&
sudo make prefix=/usr MANDIR=/usr/share/man/man1 \
 -f unix/Makefile install     &&

cd .. &&
rm -rf unzip60
```

## p7zip

This is the POSIX implementation of the Windows `7-Zip`, a single general-purpose compression tool that handles all widely-used algorithms

```sh
wget https://github.com/jinfeihan57/p7zip/archive/v17.04/p7zip-17.04.tar.gz -P /sources &&

tar xzvf /sources/p7zip-17.04.tar.gz &&
cd        p7zip-17.04                &&

sed '/^gzip/d' -i install.sh &&
sed -i '160a if(_buffer == nullptr || _size == _pos) return E_FAIL;' CPP/7zip/Common/StreamObjects.cpp &&

make all3 &&
sudo make DEST_HOME=/usr \
          DEST_MAN=/usr/share/man \
          DEST_SHARE_DOC=/usr/share/doc/p7zip-17.04 install &&

cd .. &&
rm -rf p7zip-17.04
```

## tree

```sh
wget http://mama.indstate.edu/users/ice/tree/src/tree-1.8.0.tgz -P /sources &&

tar xzvf /sources/tree-1.8.0.tgz &&
cd        tree-1.8.0             &&

make                                         &&
sudo make MANDIR=/usr/share/man/man1 install &&
sudo chmod -v 644 /usr/share/man/man1/tree.1 &&

cd .. &&
rm -rf tree-1.8.0
```

## time

```sh
wget https://ftp.gnu.org/gnu/time/time-1.9.tar.gz -P /sources &&

tar xzvf /sources/time-1.9.tar.gz &&
cd        time-1.9                &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf time-1.9
```

## tmux

Terminal multiplexer. Allows running multiple sessions from a single terminal.

```sh
wget https://github.com/tmux/tmux/releases/download/3.2/tmux-3.2.tar.gz -P /sources &&

tar xzvf /sources/tmux-3.2.tar.gz &&
cd        tmux-3.2                &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf tmux-3.2
```

## logrotate

Daemon to compress and rotate system logs so errant logging doesn't fill up your filesystem.

```sh
wget https://github.com/logrotate/logrotate/releases/download/3.18.1/logrotate-3.18.1.tar.xz -P /sources &&

tar xvf /sources/logrotate-3.18.1.tar.xz &&
cd       logrotate-3.18.1                &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf logrotate-3.18.1
```

`logrotate` requires configuration files. The base configuration from Beyond Linux From Scratch is this:

```sh
cat > /etc/logrotate.conf << EOF
# Begin /etc/logrotate.conf

# Rotate log files weekly
weekly

# Don't mail logs to anybody
nomail

# If the log file is empty, it will not be rotated
notifempty

# Number of backups that will be kept
# This will keep the 2 newest backups only
rotate 2

# Create new empty files after rotating old ones
# This will create empty log files, with owner
# set to root, group set to sys, and permissions 664
create 0664 root sys

# Compress the backups with gzip
compress

# No packages own lastlog or wtmp -- rotate them here
/var/log/wtmp {
    monthly
    create 0664 root utmp
    rotate 1
}

/var/log/lastlog {
    monthly
    rotate 1
}

# Some packages drop log rotation info in this directory
# so we include any file in it.
include /etc/logrotate.d

# End /etc/logrotate.conf
EOF

chmod -v 0644 /etc/logrotate.conf
```

Then create the structure for additional configs:

```sh
mkdir -p /etc/logrotate.d
```

```sh
cat > /etc/logrotate.d/sys.log << EOF
/var/log/sys.log {
   # If the log file is larger than 100kb, rotate it
   size   100k
   rotate 5
   weekly
   postrotate
      /bin/killall -HUP syslogd
   endscript
}
EOF

chmod -v 0644 /etc/logrotate.d/sys.log
```

Then a `systemd` one-shot service is created, along with a timer to run it daily:

```sh
cat > /usr/lib/systemd/system/logrotate.service << "EOF" &&
[Unit]
Description=Runs the logrotate command
Documentation=man:logrotate(8)
DefaultDependencies=no
After=local-fs.target
Before=shutdown.target

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/sbin/logrotate /etc/logrotate.conf
EOF
cat > /usr/lib/systemd/system/logrotate.timer << "EOF" &&
[Unit]
Description=Runs the logrotate command daily at 3:00 AM

[Timer]
OnCalendar=*-*-* 3:00:00
Persistent=true

[Install]
WantedBy=timers.target
EOF
systemctl enable logrotate.timer
```

## D-Bus

Although installed as part of base Linux From Scratch, here we rebuild with support for per-user sessions. You will need to add `--disable-doxygen-docs` if you don't have `doxygen` installed and `--disable-xml-docs` if you don't have `xmlto` installed. These will disable the API documentation and the html documentation, respectively.

```sh
tar xzvf /sources/base/dbus-1.12.20.tar.gz &&
cd        dbus-1.12.20                     &&

./configure --prefix=/usr                        \
            --sysconfdir=/etc                    \
            --localstatedir=/var                 \
            --enable-user-session                \
            --disable-static                     \
            --docdir=/usr/share/doc/dbus-1.12.20 \
            --with-console-auth-dir=/run/console \
            --with-system-pid-file=/run/dbus/pid \
            --with-system-socket=/run/dbus/system_bus_socket &&

make              &&
sudo make install &&

cd .. &&
rm -rf dbus-1.12.20
```

## patchelf

A utility from `NixOS` that allows changing the dynamic linker and RPATH of `ELF` executables.

```sh
wget https://github.com/NixOS/patchelf/archive/0.12/patchelf-0.12.tar.gz -P /sources &&

tar xzvf /sources/patchelf-0.12.tar.gz &&
cd        patchelf-0.12                &&

autoreconf -ifv &&

./configure --prefix=/usr \
            --docdir=/usr/share/doc/patchelf-0.12 &&

make              &&
make -k check     &&
sudo make install &&

cd .. &&
rm -rf patchelf-0.12
```

## wget

Alternative web downloader to `curl`. Linux From Scratch relies upon this download files in bulk from a list.

```sh
wget https://ftp.gnu.org/gnu/wget/wget-1.21.1.tar.gz -P /sources/wget-1.21.1.tar.gz &&

tar xzvf /sources/wget-1.21.1.tar.gz &&
cd        wget-1.21.1                &&

./configure --prefix=/usr      \
            --sysconfdir=/etc  \
            --with-linux-crypto &&

make              &&
sudo make install &&

cd .. &&
rm -rf wget-1.21.1
```
