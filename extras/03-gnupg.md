# GnuPG and dependencies

## libgpgerror

```sh
curl https://www.gnupg.org/ftp/gcrypt/libgpg-error/libgpg-error-1.42.tar.bz2 -o /sources/libgpg-error-1.42.tar.bz2 &&

tar xvf /sources/libgpg-error-1.42.tar.bz2 &&
cd       libgpg-error-1.42                 &&

./configure --prefix=/usr &&

make                                                                    &&
sudo make install                                                       &&
sudo install -v -m644 -D README /usr/share/doc/libgpg-error-1.42/README &&

cd .. &&
rm -rf libgpg-error-1.42
```

## libassuan

```sh
curl https://www.gnupg.org/ftp/gcrypt/libassuan/libassuan-2.5.5.tar.bz2 -o /sources/libassuan-2.5.5.tar.bz2 &&

tar xvf /sources/libassuan-2.5.5.tar.bz2 &&
cd       libassuan-2.5.5                 &&

./configure --prefix=/usr &&

make                                                                   &&
make -C doc html                                                       &&
makeinfo --html --no-split -o doc/assuan_nochunks.html doc/assuan.texi &&
makeinfo --plaintext       -o doc/assuan.txt           doc/assuan.texi &&
sudo make install                                                      &&
sudo install -v -dm755   /usr/share/doc/libassuan-2.5.5/html           &&
sudo install -v -m644 doc/assuan.html/* \
                    /usr/share/doc/libassuan-2.5.5/html                &&
sudo install -v -m644 doc/assuan_nochunks.html \
                    /usr/share/doc/libassuan-2.5.5                     &&
sudo install -v -m644 doc/assuan.{txt,texi} \
                    /usr/share/doc/libassuan-2.5.5                     &&

cd .. &&
rm -rf libassuan-2.5.5
```

## libgcrypt

```sh
curl https://www.gnupg.org/ftp/gcrypt/libgcrypt/libgcrypt-1.9.3.tar.bz2 -o /sources/libgcrypt-1.9.3.tar.bz2 &&

tar xvf /sources/libgcrypt-1.9.3.tar.bz2 &&
cd       libgcrypt-1.9.3                 &&

./configure --prefix=/usr &&

make                                                                   &&
make -C doc html                                                       &&
makeinfo --html --no-split -o doc/gcrypt_nochunks.html doc/gcrypt.texi &&
makeinfo --plaintext       -o doc/gcrypt.txt           doc/gcrypt.texi &&
make -k check                                                          &&
sudo make install                                                      &&
sudo install -v -dm755   /usr/share/doc/libgcrypt-1.9.3                &&
sudo install -v -m644    README doc/{README.apichanges,fips*,libgcrypt*} \
                    /usr/share/doc/libgcrypt-1.9.3                     &&

sudo install -v -dm755   /usr/share/doc/libgcrypt-1.9.3/html           &&
sudo install -v -m644 doc/gcrypt.html/* \
                    /usr/share/doc/libgcrypt-1.9.3/html                &&
sudo install -v -m644 doc/gcrypt_nochunks.html \
                    /usr/share/doc/libgcrypt-1.9.3                     &&
sudo install -v -m644 doc/gcrypt.{txt,texi} \
                    /usr/share/doc/libgcrypt-1.9.3                     &&

cd .. &&
rm -rf libgcrypt-1.9.3
```

## libksba

```sh
curl https://www.gnupg.org/ftp/gcrypt/libksba/libksba-1.6.0.tar.bz2 -o /sources/libksba-1.6.0.tar.bz2 &&

tar xvf /sources/libksba-1.6.0.tar.bz2 &&
cd       libksba-1.6.0                 &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf libksba-1.6.0
```

## NPth

```sh
curl https://www.gnupg.org/ftp/gcrypt/npth/npth-1.6.tar.bz2 -o /sources/npth-1.6.tar.bz2 &&

tar xvf /sources/npth-1.6.tar.bz2 &&
cd       npth-1.6                 &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf npth-1.6
```

## pinentry

```sh
curl https://www.gnupg.org/ftp/gcrypt/pinentry/pinentry-1.1.1.tar.bz2 -o /sources/pinentry-1.1.1.tar.bz2 &&

tar xvf /sources/pinentry-1.1.1.tar.bz2 &&
cd       pinentry-1.1.1                 &&

./configure --prefix=/usr \
            --enable-pinentry-tty &&

make              &&
sudo make install &&

cd .. &&
rm -rf pinentry-1.1.1
```

## GnuPG

```sh
curl https://www.gnupg.org/ftp/gcrypt/gnupg/gnupg-2.2.28.tar.bz2 -o /sources/gnupg-2.2.28.tar.bz2 &&

tar xvf /sources/gnupg-2.2.28.tar.bz2 &&
cd       gnupg-2.2.28                 &&

./configure --prefix=/usr            \
            --localstatedir=/var     \
            --docdir=/usr/share/doc/gnupg-2.2.28 &&

make                                                                 &&
makeinfo --html --no-split -o doc/gnupg_nochunks.html doc/gnupg.texi &&
makeinfo --plaintext       -o doc/gnupg.txt           doc/gnupg.texi &&
make -C doc html                                                     &&
sudo make install                                                    &&
sudo install -v -m755 -d /usr/share/doc/gnupg-2.2.28/html            &&
sudo install -v -m644    doc/gnupg_nochunks.html \
                    /usr/share/doc/gnupg-2.2.28/html/gnupg.html      &&
sudo install -v -m644    doc/*.texi doc/gnupg.txt \
                    /usr/share/doc/gnupg-2.2.28                      &&
sudo install -v -m644    doc/gnupg.html/* \
                    /usr/share/doc/gnupg-2.2.28/html                 &&

cd .. &&
rm -rf gnupg-2.2.28
```

## GPGME

```sh
curl https://www.gnupg.org/ftp/gcrypt/gpgme/gpgme-1.15.1.tar.bz2 -o /sources/gpgme-1.15.1.tar.bz2 &&

tar xvf /sources/gpgme-1.15.1.tar.bz2 &&
cd       gpgme-1.15.1                 &&

./configure --prefix=/usr &&

make              &&
make -k check     &&
sudo make install &&

cd .. &&
rm -rf gpgme-1.15.1
```
