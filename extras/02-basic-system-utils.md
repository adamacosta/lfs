# General utility software

## rsync

This will be built without support for the daemon.

```sh
curl https://www.samba.org/ftp/rsync/src/rsync-3.2.3.tar.gz -o rsync-3.2.3.tar.gz
tar xzvf rsync-3.2.3.tar.gz
cd rsync-3.2.3

./configure --prefix=/usr    \
            --disable-lz4    \
            --disable-xxhash \
            --without-included-zlib
make -j
sudo make install

cd ..
rm -rf rsync-3.2.3
```

## pigz

```sh
curl https://zlib.net/pigz/pigz-2.6.tar.gz -o pigz-2.6
tar xzvf pigz-2.6
cd pigz-2.6

make
make test
sudo install -m755 pigz /usr/bin &&
sudo install -m755 unpigz /usr/bin

cd ..
rm -rf pigz-2.6
```

## tree

```sh
curl http://mama.indstate.edu/users/ice/tree/src/tree-1.8.0.tgz -o tree-1.8.0.tgz
tar xzvf tree-1.8.0.tgz
cd tree-1.8.0

make -j
sudo make MANDIR=/usr/share/man/man1 install &&
sudo chmod -v 644 /usr/share/man/man1/tree.1

cd ..
rm -rf tree-1.8.0
```

## time

```sh
curl https://ftp.gnu.org/gnu/time/time-1.9.tar.gz -o time-1.9.tar.gz
tar xzvf time-1.9.tar.gz
cd time-1.9

./configure --prefix=/usr
make -j
sudo make install

cd ..
rm -rf time-1.9
```

## tmux

First, get its dependency, `libevent`:

```sh
curl https://github.com/libevent/libevent/releases/download/release-2.1.12-stable/libevent-2.1.12-stable.tar.gz -o libevent-2.1.12-stable.tar.gz
tar xzvf libevent-2.1.12-stable.tar.gz
cd libevent-2.1.12

./configure --prefix=/usr --disable-static
make -j
sudo make install

cd ..
rm -rf libevent-2.1.12
```

Now install `tmux`:

```sh
curl https://github.com/tmux/tmux/releases/download/3.2/tmux-3.2.tar.gz -o tmux-3.2.tar.gz
tar xzvf tmux-3.2.tar.gz
cd tmux-3.2

./configure --prefix=/usr
make -j
sudo make install

cd ..
rm -rf tmux-3.2
```

## wget

```sh
curl https://ftp.gnu.org/gnu/wget/wget-1.21.1.tar.gz -o wget-1.21.1.tar.gz
tar xzvf wget-1.21.1.tar.gz
cd wget-1.21.1

./configure --prefix=/usr      \
            --sysconfdir=/etc  \
            --with-linux-crypto
make -j
sudo make install

cd ..
rm -rf wget-1.21.1
```