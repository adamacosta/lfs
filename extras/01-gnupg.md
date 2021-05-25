# GnuPG and dependencies

This is going to be critical to use the system for any sort of development and for package upgrades, as it is needed to verify the provenance of any packages being installed.

## libgpgerror

```sh
curl https://www.gnupg.org/ftp/gcrypt/libgpg-error/libgpg-error-1.41.tar.bz2 -o libgpg-error-1.41.tar.bz2
tar xvf libgpg-error-1.41.tar.bz2
cd libgpg-error-1.41

./configure --prefix=/usr
make -j
sudo make install
sudo install -v -m644 -D README /usr/share/doc/libgpg-error-1.41/README

cd ..
rm -rf libgpg-error-1.41
```

## libassuan

```sh
curl https://www.gnupg.org/ftp/gcrypt/libassuan/libassuan-2.5.4.tar.bz2 -o libassuan-2.5.4.tar.bz2
tar xvf libassuan-2.5.4
cd libassuan-2.5.4

./configure --prefix=/usr
make -j
make -C doc html                                                       &&
makeinfo --html --no-split -o doc/assuan_nochunks.html doc/assuan.texi &&
makeinfo --plaintext       -o doc/assuan.txt           doc/assuan.texi
sudo make install &&
sudo install -v -dm755   /usr/share/doc/libassuan-2.5.4/html &&
sudo install -v -m644 doc/assuan.html/* \
                    /usr/share/doc/libassuan-2.5.4/html &&
sudo install -v -m644 doc/assuan_nochunks.html \
                    /usr/share/doc/libassuan-2.5.4      &&
sudo install -v -m644 doc/assuan.{txt,texi} \
                    /usr/share/doc/libassuan-2.5.4

cd ..
rm -rf libassuan-2.5.4
```

## libgcrypt

```sh
curl https://www.gnupg.org/ftp/gcrypt/libgcrypt/libgcrypt-1.9.2.tar.bz2 -o libgcrypt-1.9.2.tar.bz2
tar xvf libgcrypt-1.9.2.tar.bz2
cd libgcrypt-1.9.2

./configure --prefix=/usr
make -j
make -C doc html                                                       &&
makeinfo --html --no-split -o doc/gcrypt_nochunks.html doc/gcrypt.texi &&
makeinfo --plaintext       -o doc/gcrypt.txt           doc/gcrypt.texi
make -j check
sudo make install &&
sudo install -v -dm755   /usr/share/doc/libgcrypt-1.9.2 &&
sudo install -v -m644    README doc/{README.apichanges,fips*,libgcrypt*} \
                    /usr/share/doc/libgcrypt-1.9.2 &&

sudo install -v -dm755   /usr/share/doc/libgcrypt-1.9.2/html &&
sudo install -v -m644 doc/gcrypt.html/* \
                    /usr/share/doc/libgcrypt-1.9.2/html &&
sudo install -v -m644 doc/gcrypt_nochunks.html \
                    /usr/share/doc/libgcrypt-1.9.2      &&
sudo install -v -m644 doc/gcrypt.{txt,texi} \
                    /usr/share/doc/libgcrypt-1.9.2

cd ..
rm -rf libgcrypt-1.9.2
```

## libksba

```sh
curl https://www.gnupg.org/ftp/gcrypt/libksba/libksba-1.5.0.tar.bz2 -o libksba-1.5.0.tar.bz2
tar xvf libksba-1.5.0.tar.bz2
cd libksba-1.5.0

./configure --prefix=/usr
make -j
sudo make install

cd ..
rm -rf libksba-1.5.0
```

## NPth

```sh
curl https://www.gnupg.org/ftp/gcrypt/npth/npth-1.6.tar.bz2 -o npth-1.6.tar.bz2
tar xvf npth-1.6.tar.bz2
cd npth-1.6

./configure --prefix=/usr
make -j
sudo make install

cd ..
rm -rf npth-1.6
```

## pinentry

```sh
curl https://www.gnupg.org/ftp/gcrypt/pinentry/pinentry-1.1.1.tar.bz2 -o pinentry-1.1.1.tar.bz2
tar xvf pinentry-1.1.1.tar.bz2
cd pinentry-1.1.1

./configure --prefix=/usr --enable-pinentry-tty
make -j
sudo make install

cd ..
rm -rf pinentry-1.1.1
```

## GnuPG

```sh
curl https://www.gnupg.org/ftp/gcrypt/gnupg/gnupg-2.2.27.tar.bz2 -o gnupg-2.2.27.tar.bz2
tar xvf gnupg-2.2.27.tar.bz2
cd gnupg-2.2.27

./configure --prefix=/usr            \
            --localstatedir=/var     \
            --docdir=/usr/share/doc/gnupg-2.2.27
make -j
makeinfo --html --no-split -o doc/gnupg_nochunks.html doc/gnupg.texi &&
makeinfo --plaintext       -o doc/gnupg.txt           doc/gnupg.texi &&
make -C doc html
sudo make install &&
sudo install -v -m755 -d /usr/share/doc/gnupg-2.2.27/html            &&
sudo install -v -m644    doc/gnupg_nochunks.html \
                    /usr/share/doc/gnupg-2.2.27/html/gnupg.html &&
sudo install -v -m644    doc/*.texi doc/gnupg.txt \
                    /usr/share/doc/gnupg-2.2.27 &&
sudo install -v -m644    doc/gnupg.html/* \
                    /usr/share/doc/gnupg-2.2.27/html

cd ..
rm -rf gnupg-2.2.27
```

## GPGME

```sh
curl https://www.gnupg.org/ftp/gcrypt/gpgme/gpgme-1.12.0.tar.bz2 -o gpgme-1.12.0.tar.bz2
tar xvf gpgme-1.12.0.tar.bz2
cd gpgme-1.12.0

./configure --prefix=/usr
make -j
make -k check
sudo make install

cd ..
rm -rf gpgme-1.12.0
```
