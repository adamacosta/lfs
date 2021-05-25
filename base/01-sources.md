# Obtaining the sources

```sh
wget https://www.linuxfromscratch.org/lfs/view/stable-systemd/wget-list
wget https://www.linuxfromscratch.org/lfs/view/stable-systemd/md5sums

mkdir -v $LFS/sources
chmod -v a+wt $LFS/sources

wget --input-file=wget-list --continue --directory-prefix=$LFS/sources
pushd $LFS/sources
  md5sum -c md5sums
popd
```

These are not in the default LFS list, but are needed for `systemd-boot` to work, which we will be using in lieu of `GRUB`:

```sh
wget https://github.com/kontais/gnu-efi/archive/refs/tags/3.0.12.tar.gz -O $LFS/sources/gnu-efi-3.0.12.tar.gz
wget https://github.com/rhboot/efivar/releases/download/37/efivar-37.tar.bz2 -O $LFS/sources/efivar-37.tar.bz2
wget https://www.linuxfromscratch.org/patches/blfs/svn/efivar-37-gcc_9-1.patch -O $LFS/sources/efivar-37-gcc_9-1.patch
```

We will add `curl`, `openssh`, `pam`, and `sudo` to make life a bit easier for using the new system as a headless VM.

```sh
wget https://curl.se/download/curl-7.76.1.tar.gz -O $LFS/sources/curl-7.76.1.tar.gz
wget https://mirror.esc7.net/pub/OpenBSD/OpenSSH/portable/openssh-8.6p1.tar.gz -O $LFS/sources/openssh-8.6p1.tar.gz
wget https://github.com/linux-pam/linux-pam/releases/download/v1.5.1/Linux-PAM-1.5.1.tar.xz -O $LFS/sources/Linux-PAM-1.5.1.tar.xz
wget https://www.sudo.ws/dist/sudo-1.9.7.tar.gz -O $LFS/sources/sudo-1.9.7.tar.gz
```

We will also be adding `libtasn`, `p11-kit`, `make-ca`, `nettle`, `libunistring`, `GnuTLS`, `PCRE`, and `zsh`:

```sh
wget https://ftp.gnu.org/gnu/libtasn1/libtasn1-4.16.0.tar.gz -O $LFS/sources/libtasn1-4.16.0.tar.gz
wget https://github.com/p11-glue/p11-kit/releases/download/0.23.22/p11-kit-0.23.22.tar.xz -O $LFS/sources/p11-kit-0.23.22.tar.xz
wget https://github.com/djlucas/make-ca/releases/download/v1.7/make-ca-1.7.tar.xz -O $LFS/sources/make-ca-1.7.tar.xz
wget https://ftp.gnu.org/gnu/nettle/nettle-3.7.1.tar.gz -O $LFS/sources/nettle-3.7.1.tar.gz
wget https://ftp.gnu.org/gnu/libunistring/libunistring-0.9.10.tar.xz -O $LFS/sources/libunistring-0.9.10.tar.gz
wget https://www.gnupg.org/ftp/gcrypt/gnutls/v3.7/gnutls-3.7.0.tar.xz -O $LFS/sources/gnutls-3.7.0.tar.xz
wget https://ftp.pcre.org/pub/pcre/pcre-8.44.tar.bz2 -O $LFS/sources/pcre-8.44.tar.bz2
wget http://www.zsh.org/pub/zsh-5.8.tar.xz -O $LFS/sources/zsh-5.8.tar.xz
```

Get the `BLFS` `systemd` units:

```sh
wget http://www.linuxfromscratch.org/blfs/downloads/10.1-systemd/blfs-systemd-units-20210122.tar.xz -O $LFS/sources/blfs-systemd-units-20210122.tar.xz
```

Finally, get the `which` utility, which is handy for finding paths to executables:

```sh
wget https://carlowood.github.io/which/which-2.21.tar.gz -O $LFS/sources/which-2.21.tar.gz
```