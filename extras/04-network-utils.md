# Network utilities

## nghttp2

Implementation of HTTP/2.

```sh
wget https://github.com/nghttp2/nghttp2/releases/download/v1.43.0/nghttp2-1.43.0.tar.xz -P /sources &&

tar xvf /sources/nghttp2-1.43.0.tar.xz &&
cd       nghttp2-1.43.0                &&

./configure --prefix=/usr     \
            --disable-static  \
            --enable-lib-only \
            --docdir=/usr/share/doc/nghttp2-1.43.0 &&

make              &&
sudo make install &&

cd .. &&
rm -rf nghttp2-1.43.0
```

## bind (client only)

Building just the client provides the `dig` utility for querying DNS servers.


```sh
curl ftp://ftp.isc.org/isc/bind9/9.16.16/bind-9.16.16.tar.xz -P /sources &&

tar xvf /sources/bind-9.16.16.tar.xz &&
cd       bind-9.16.16                &&

./configure --prefix=/usr \
            --without-python &&

make -C lib/dns                                                  &&
make -C lib/isc                                                  &&
make -C lib/bind9                                                &&
make -C lib/isccfg                                               &&
make -C lib/irs                                                  &&
make -C bin/dig                                                  &&
make -C doc                                                      &&
sudo make -C bin/dig install                                     &&
sudo cp -v doc/man/{dig.1,host.1,nslookup.1} /usr/share/man/man1 &&

cd .. &&
rm -rf bind-9.16.16
```

## traceroute

Trace the network hops of your packets.

```sh
wget https://downloads.sourceforge.net/traceroute/traceroute-2.1.0.tar.gz -P /sources &&

tar xzvf /sources/traceroute-2.1.0.tar.gz &&
cd        traceroute-2.1.0                &&

make                                                          &&
sudo make prefix=/usr install                                 &&
sudo ln -sv -f traceroute /bin/traceroute6                    &&
sudo ln -sv -f traceroute.8 /usr/share/man/man8/traceroute6.8 &&
sudo rm -fv /usr/share/man/man1/traceroute.1                  &&

cd .. &&
rm -rf traceroute-2.1.0
```

## whois

Query the domain registry database.

```sh
wget https://github.com/rfc1036/whois/archive/v5.4.3/whois-5.4.3.tar.gz -P /sources &&

tar xzvf /sources/whois-5.4.3.tar.gz &&
cd        whois-5.4.3                &&

make                                &&
sudo make prefix=/usr install-whois &&

cd .. &&
rm -rf whois-5.4.3
```

## nmap

Port scanning tool.

```sh
wget http://nmap.org/dist/nmap-7.91.tar.bz2 -P /sources &&

tar xvf /sources/nmap-7.91.tar.bz2 &&
cd       nmap-7.91                 &&

./configure --prefix=/usr &&

make                                          &&
sed -i 's/lib./lib/' zenmap/test/run_tests.py &&
make -k check                                 &&
sudo make install                             &&

cd .. &&
rm -rf nmap-7.91
```

Some scripts or Internet guides may refer to `nc` rather than `ncat`, so you can create a symlink for compatibility:

```sh
sudo ln -s ncat /usr/bin/nc
```

## nspr

Netscape Portable Runtime

```sh
wget https://archive.mozilla.org/pub/nspr/releases/v4.31/src/nspr-4.31.tar.gz -P /sources &&

tar xzvf /sources/nspr-4.31.tar.gz &&
cd        nspr-4.31                &&

cd nspr                                                     &&
sed -ri 's#^(RELEASE_BINS =).*#\1#' pr/src/misc/Makefile.in &&
sed -i 's#$(LIBRARY) ##'            config/rules.mk         &&

./configure --prefix=/usr \
            --with-mozilla \
            --with-pthreads \
            --enable-64bit &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf nspr-4.31
```

## nss

Network Security Services.

```sh
wget https://archive.mozilla.org/pub/security/nss/releases/NSS_3_67_RTM/src/nss-3.67.tar.gz -P /sources &&
wget http://www.linuxfromscratch.org/patches/blfs/svn/nss-3.67-standalone-1.patch -P /sources/patches &&

tar xzvf /sources/nss-3.67.tar.gz &&
cd       nss-3.67                 &&

patch -Np1 -i /sources/nss-3.67-standalone-1.patch &&
cd nss                                             &&

make -j1 BUILD_OPT=1                  \
  NSPR_INCLUDE_DIR=/usr/include/nspr  \
  USE_SYSTEM_ZLIB=1                   \
  ZLIB_LIBS=-lz                       \
  NSS_ENABLE_WERROR=0                 \
  USE_64=1                            \
  NSS_USE_SYSTEM_SQLITE=1 &&

cd tests                                   &&
HOST=localhost DOMSUF=localdomain ./all.sh &&
cd ../../dist                              &&

sudo install -v -m755 Linux*/lib/*.so              /usr/lib              &&
sudo install -v -m644 Linux*/lib/{*.chk,libcrmf.a} /usr/lib              &&
sudo install -v -m755 -d                           /usr/include/nss      &&
sudo cp -v -RL {public,private}/nss/*              /usr/include/nss      &&
sudo chmod -v 644                                  /usr/include/nss/*    &&
sudo install -v -m755 Linux*/bin/{certutil,nss-config,pk12util} /usr/bin &&
sudo install -v -m644 Linux*/lib/pkgconfig/nss.pc  /usr/lib/pkgconfig

cd ../.. &&
rm -rf nss-3.67
```

Point libnssckbi.so to pkcs11/p11-kit-trust.so to use `p11-kit`:

```sh
sudo ln -sfv ./pkcs11/p11-kit-trust.so /usr/lib/libnssckbi.so
```

## Rebuild cURL again

Now that we also have `nss`, we can rebuild `cURL` to use it:

```sh
tar xzvf /sources/curl-7.76.1.tar.gz &&
cd        curl-7.76.1                &&

./configure --prefix=/usr                                         \
            --with-openssl                                        \
            --with-gnutls                                         \
            --with-nss                                            \
            --disable-static                                      \
            --with-zsh-functions-dir=/usr/share/zsh/5.8/functions \
            --with-ca-bundle=/etc/pki/tls/certs/ca-bundle.crt &&

make                           &&
make test                      &&
sudo make install              &&
sudo rm -v /usr/lib/libcurl.la &&

cd .. &&
rm -rf curl-7.76.1
```

## iw

Interact with wireless devices.

```sh
wget https://www.kernel.org/pub/software/network/iw/iw-5.9.tar.xz -P /sources &&

tar xvf /sources/iw-5.9.tar.xz &&
cd       iw-5.9                &&

sed -i "/INSTALL.*gz/s/.gz//" Makefile &&

make              &&
sudo make install &&

cd .. &&
rm -rf iw-5.9
```
