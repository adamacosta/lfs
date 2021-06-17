# General utility software

## rsync

Synchornize different file trees, indifferent to whether the target is local or remote. This will be built without support for the daemon.

```sh
curl https://www.samba.org/ftp/rsync/src/rsync-3.2.3.tar.gz -o /sources/rsync-3.2.3.tar.gz &&

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
curl https://zlib.net/pigz/pigz-2.6.tar.gz -o /sources/pigz-2.6.tar.gz &&

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
curl https://launchpad.net/pbzip2/1.1/1.1.13/+download/pbzip2-1.1.13.tar.gz -o /sources/pbzip2-1.1.13.tar.gz &&

tar xzvf /sources/pbzip2-1.1.13.tar.gz &&
cd        pbzip2-1.1.13                &&

make                               &&
sudo install -m755 pbzip2 /usr/bin &&

cd .. &&
rm -rf pbzip2-1.1.13
```

## Zip

```sh
curl https://downloads.sourceforge.net/infozip/zip30.tar.gz -o /sources/zip30.tar.gz &&

tar xzvf /sources/zip30.tar.gz &&
cd        zip30                &&

make -f unix/Makefile generic_gcc                                         &&
sudo make prefix=/usr MANDIR=/usr/share/man/man1 -f unix/Makefile install &&

cd .. &&
rm -rf zip30
```

## Unzip

```sh
curl https://downloads.sourceforge.net/infozip/unzip60.tar.gz -o /sources/unzip60.tar.gz &&
curl http://www.linuxfromscratch.org/patches/blfs/10.1/unzip-6.0-consolidated_fixes-1.patch -o /sources/unzip-6.0-consolidated_fixes-1.patch &&

tar xzvf /sources/unzip60.tar.gz &&
cd        unzip60                &&

patch -Np1 -i /sources/unzip-6.0-consolidated_fixes-1.patch &&

make -f unix/Makefile generic &&
sudo make prefix=/usr MANDIR=/usr/share/man/man1 \
 -f unix/Makefile install     &&

cd .. &&
rm -rf unzip60
```

## tree

```sh
curl http://mama.indstate.edu/users/ice/tree/src/tree-1.8.0.tgz -o /sources/tree-1.8.0.tgz &&

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
curl https://ftp.gnu.org/gnu/time/time-1.9.tar.gz -o /sources/time-1.9.tar.gz &&

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

First, get its dependency, `libevent`:

```sh
curl https://github.com/libevent/libevent/releases/download/release-2.1.12-stable/libevent-2.1.12-stable.tar.gz -o /sources/libevent-2.1.12-stable.tar.gz &&

tar xzvf /sources/libevent-2.1.12-stable.tar.gz &&
cd        libevent-2.1.12-stable                &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libevent-2.1.12-stable
```

Now install `tmux`:

```sh
curl https://github.com/tmux/tmux/releases/download/3.2/tmux-3.2.tar.gz -o /sources/tmux-3.2.tar.gz &&

tar xzvf /sources/tmux-3.2.tar.gz &&
cd        tmux-3.2                &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf tmux-3.2
```

## wget

Alternative web downloader to `curl`. Linux From Scratch relies upon this download files in bulk from a list.

```sh
curl https://ftp.gnu.org/gnu/wget/wget-1.21.1.tar.gz -o /sources/wget-1.21.1.tar.gz &&

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
