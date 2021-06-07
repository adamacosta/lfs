# Network utilities

## bind (client only)

`bind` depends upon `libuv`, which needs to be installed first:

```sh
curl https://dist.libuv.org/dist/v1.41.0/libuv-v1.41.0.tar.gz -o libuv-v1.41.0.tar.gz
tar xzvf libuv-v1.41.0.tar.gz
cd libuv-v1.41.0

sh autogen.sh &&
./configure --prefix=/usr --disable-static
make
sudo make install

cd ..
rm -rf libuv-v1.41.0
```

```sh
curl ftp://ftp.isc.org/isc/bind9/9.16.11/bind-9.16.11.tar.xz -o bind-9.16.11.tar.xz
tar xvf bind-9.16.11.tar.xz
cd bind-9.16.11

./configure --prefix=/usr --without-python
make -C lib/dns    &&
make -C lib/isc    &&
make -C lib/bind9  &&
make -C lib/isccfg &&
make -C lib/irs    &&
make -C bin/dig    &&
make -C doc        &&
sudo make -C bin/dig install &&
sudo cp -v doc/man/{dig.1,host.1,nslookup.1} /usr/share/man/man1
```

## traceroute

```sh
curl -L https://downloads.sourceforge.net/traceroute/traceroute-2.1.0.tar.gz -o traceroute-2.1.0.tar.gz
tar xzvf traceroute-2.1.0.tar.gz
cd traceroute-2.1.0

make
sudo make prefix=/usr install                                 &&
sudo mv /usr/bin/traceroute /bin                              &&
sudo ln -sv -f traceroute /bin/traceroute6                    &&
sudo ln -sv -f traceroute.8 /usr/share/man/man8/traceroute6.8 &&
sudo rm -fv /usr/share/man/man1/traceroute.1

cd ..
rm -rf traceroute-2.1.0
```

## whois

```sh
curl -L https://github.com/rfc1036/whois/archive/v5.4.3/whois-5.4.3.tar.gz -o whois-5.4.3.tar.gz
tar xzvf whois-5.4.3.tar.gz
cd whois-5.4.3

make
sudo make prefix=/usr install-whois

cd ..
rm -rf whois-5.4.3
```

## nmap

First, install the dependencies `libpcap`, `liblinear`, and `Lua`:

```sh
curl http://www.tcpdump.org/release/libpcap-1.10.0.tar.gz -o libpcap-1.10.0.tar.gz
tar xzvf libpcap-1.10.0.tar.gz
cd libpcap-1.10.0

./configure --prefix=/usr
make
sed -i '/INSTALL_DATA.*libpcap.a\|RANLIB.*libpcap.a/ s/^/#/' Makefile
sudo make install

cd ..
rm -rf libpcap-1.10.0

curl -L https://github.com/cjlin1/liblinear/archive/v242/liblinear-242.tar.gz -o liblinear-242.tar.gz
tar xzvf liblinear-242.tar.gz
cd liblinear-242

make lib
sudo install -vm644 linear.h /usr/include &&
sudo install -vm755 liblinear.so.4 /usr/lib &&
sudo ln -sfv liblinear.so.4 /usr/lib/liblinear.so

cd ..
rm -rf liblinear-242

curl http://www.lua.org/ftp/lua-5.4.2.tar.gz -o lua-5.4.2.tar.gz
curl -L http://www.linuxfromscratch.org/patches/blfs/10.1/lua-5.4.2-shared_library-1.patch -o lua-5.4.2-shared_library-1.patch
tar xzvf lua-5.4.2.tar.gz
cd lua-5.4.2

cat > lua.pc << "EOF"
V=5.4
R=5.4.2

prefix=/usr
INSTALL_BIN=${prefix}/bin
INSTALL_INC=${prefix}/include
INSTALL_LIB=${prefix}/lib
INSTALL_MAN=${prefix}/share/man/man1
INSTALL_LMOD=${prefix}/share/lua/${V}
INSTALL_CMOD=${prefix}/lib/lua/${V}
exec_prefix=${prefix}
libdir=${exec_prefix}/lib
includedir=${prefix}/include

Name: Lua
Description: An Extensible Extension Language
Version: ${R}
Requires:
Libs: -L${libdir} -llua -lm -ldl
Cflags: -I${includedir}
EOF

patch -Np1 -i ../lua-5.4.2-shared_library-1.patch &&
make linux
sudo make INSTALL_TOP=/usr                \
          INSTALL_DATA="cp -d"            \
          INSTALL_MAN=/usr/share/man/man1 \
          TO_LIB="liblua.so liblua.so.5.4 liblua.so.5.4.2" \
          install &&
sudo mkdir -pv                      /usr/share/doc/lua-5.4.2 &&
sudo cp -v doc/*.{html,css,gif,png} /usr/share/doc/lua-5.4.2 &&
sudo install -v -m644 -D lua.pc /usr/lib/pkgconfig/lua.pc

cd ..
rm -rf lua-5.4.2
```

Now install `nmap`:

```sh
curl -L http://nmap.org/dist/nmap-7.91.tar.bz2 -o nmap-7.91.tar.bz2
tar xvf nmap-7.91.tar.bz2
cd nmap-7.91

./configure --prefix=/usr
make
sed -i 's/lib./lib/' zenmap/test/run_tests.py
make -k check
sudo make install

cd ..
rm -rf nmap-7.91
```

## nspr

```sh
curl https://archive.mozilla.org/pub/nspr/releases/v4.29/src/nspr-4.29.tar.gz -o nspr-4.29.tar.gz
tar xzvf nspr-4.29.tar.gz
cd nspr-4.29

cd nspr                                                     &&
sed -ri 's#^(RELEASE_BINS =).*#\1#' pr/src/misc/Makefile.in &&
sed -i 's#$(LIBRARY) ##'            config/rules.mk         &&

./configure --prefix=/usr \
            --with-mozilla \
            --with-pthreads \
            $([ $(uname -m) = x86_64 ] && echo --enable-64bit)
make
sudo make install

cd ../..
rm -rm nspr-4.29
```

## nss

```sh
curl https://archive.mozilla.org/pub/security/nss/releases/NSS_3_61_RTM/src/nss-3.61.tar.gz -o nss-3.61.tar.gz
curl -L http://www.linuxfromscratch.org/patches/blfs/10.1/nss-3.61-standalone-1.patch -o nss-3.61-standalone-1.patch
tar xzvf nss-3.61.tar.gz

patch -Np1 -i ../nss-3.61-standalone-1.patch &&
cd nss &&
make -j1 BUILD_OPT=1                  \
  NSPR_INCLUDE_DIR=/usr/include/nspr  \
  USE_SYSTEM_ZLIB=1                   \
  ZLIB_LIBS=-lz                       \
  NSS_ENABLE_WERROR=0                 \
  $([ $(uname -m) = x86_64 ] && echo USE_64=1) \
  $([ -f /usr/include/sqlite3.h ] && echo NSS_USE_SYSTEM_SQLITE=1)
cd tests &&
HOST=localhost DOMSUF=localdomain ./all.sh
cd ../
cd ../dist
sudo install -v -m755 Linux*/lib/*.so              /usr/lib              &&
sudo install -v -m644 Linux*/lib/{*.chk,libcrmf.a} /usr/lib              &&
sudo install -v -m755 -d                           /usr/include/nss      &&
sudo cp -v -RL {public,private}/nss/*              /usr/include/nss      &&
sudo chmod -v 644                                  /usr/include/nss/*    &&
sudo install -v -m755 Linux*/bin/{certutil,nss-config,pk12util} /usr/bin &&
sudo install -v -m644 Linux*/lib/pkgconfig/nss.pc  /usr/lib/pkgconfig

cd ../..
rm -rf nss-3.61
```

Point libnssckbi.so to pkcs11/p11-kit-trust.so to use `p11-kit`:

```sh
sudo ln -sfv ./pkcs11/p11-kit-trust.so /usr/lib/libnssckbi.so
```

## Rebuild cURL again

Now that we also have `nss`, we can rebuild `cURL` to use it:

```sh
tar xzvf curl-7.76.1.tar.gz
cd curl-7.76.1

./configure --prefix=/usr                                         \
            --with-openssl                                        \
            --with-gnutls                                         \
            --with-nss                                            \
            --disable-static                                      \
            --with-zsh-functions-dir=/usr/share/zsh/5.8/functions \
            --with-ca-bundle=/etc/pki/tls/certs/ca-bundle.crt
make
make test
sudo make install
sudo rm -v /usr/lib/libcurl.la

cd ..
rm -rf curl-7.76.1
```