# Getting started

## Note on terminology

I will be using the term `host` to refer to an already installed Linux system from which Linux From Scratch is being bootstrapped, and `guest` to refer to the Linux From Scratch system. This is not standard usage, in which these terms typically refer to hypervisors and VMs running on them. Linux From Scratch can be built inside a VM and I will include instructions on how to do this, but even inside of a VM, a base Linux distro from which to bootstrap is required, with Linux From Scratch created inside of a separate partition or disk. Later, the original disk can be discarded if the bootstrap system was only being used for bootstrapping.

## Reserving space

Linux From Scratch recommends setting aside a 30 GB partition to build the system on. I would recommend more than that, noting that building `rustc` takes up about 20 GB on its own and building the tests for `boost` takes nearly 50 GB.

You can dump the partition table for a disk using the `fdisk` utility.

For example, here is an unformatted disk on a SATA controller with 4TiB of space:

```sh
sudo fdisk -l /dev/sda
Disk /dev/sda: 3.64 TiB, 4000787030016 bytes, 7814037168 sectors
Disk model: ST4000NE001-2MA1
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
```

Here is a 2TiB SSD with an EFI System Partition and a member of a Linux RAID array:

```sh
sudo fdisk -l /dev/nvme0n1
Disk /dev/nvme0n1: 1.82 TiB, 2000398934016 bytes, 3907029168 sectors
Disk model: Sabrent Rocket Q4
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 48929F80-748D-6047-9072-49E35C552708

Device          Start        End    Sectors  Size Type
/dev/nvme0n1p1   2048     534527     532480  260M EFI System
/dev/nvme0n1p2 534528 3907029134 3906494607  1.8T Linux RAID
```

You will want to find or obtain a disk with spare room on which you can create a separate partition in which to build and install Linux From Scratch. This can be a true partition or a logical volume. For some advice on how to do this, see the following resources from the Arch Wiki:

* [Partitioning](https://wiki.archlinux.org/title/Partitioning)
* [LVM](https://wiki.archlinux.org/title/LVM)

### Note on virtual disks

If you're building Linux From Scratch inside a VM, all major hypervisors include a simple way to create a dynamic disk that only takes up the space actually being used but allocates more as a maximum size. The exception I know of is Azure, which requires statically sized disks. It is possible other cloud providers have a restriction like this as well, but I do not recommend building Linux From Scratch in the cloud. Any Linux system can use  `QEMU` and Windows as of Windows 10 comes with `Hyper-V` out of the box.

### EFI versus BIOS

Linux From Scratch provides instructions for using BIOS with the `GRUB` bootloader. Given virtually all motherboards at this point support and prefer UEFI, I will instead be booting from UEFI and utilizing `systemd-boot`, which comes for free with `systemd` if you build it with the GNU EFI libraries present. There are some sections in Beyond Linux From Scratch explaining how to set up UEFI with GRUB, but they are somewhat incomplete for some use cases, specifically building inside a VM, which you are likely to want to do at first if you don't have spare disks and empty PCs laying around.

## Building from a bootstrap host

There are two possible ways to accomplish the building of a new system:

1. On an unused partition mounted to a separate Linux installation.
2. On an empty system from a live rescue or installation medium.

The first method is what Linux From Scratch uses, and the editors no longer provide a live installation image. However, the Arch installation media has all the tools needed, so you can run a live system from a USB stick and install Linux From Scratch onto a completely brand new build that way as well.

### Directory layout

Linux From Scratch recommends a layout like this:

```
/
/mnt/
/mnt/lfs/
        /<new system>...
```

In other words, create a directory `/mnt/lfs` in your host Linux system, and mount the partition you intend to install Linux From Scratch onto there.

Inside the root of the `<new system>` will be a directory tree conforming to the [Filesystem Hierarchy Standard](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html), plus a directory `/sources` in which to download and unpack all of the required source tarballs. This results in a layout like so from the host (not including the virtual filesystems from the kernel and `tmpfs` mounts):

```
/
/boot
/lib
/usr
/include
/local
/opt
/var
/home
/mnt
    /lfs
        /sources
        /boot
        /lib
        /usr
        /include
        /local
        /opt
        /var
        /home
        /mnt
```

Creating the full directory layout in the Linux From Scratch guest happens later. For now, just create a `/sources` directory and mount it, ensuring the stick bit is set:

```sh
sudo su -
export LFS=/mnt/lfs
sudo mkdir -p       $LFS
sudo mkfs.ext4      /dev/<your lfs partition>
sudo mount          /dev/<your lfs partition> $LFS
sudo mkdir -v       $LFS/sources/{patches,base}
sudo chmod -v  a+wt $LFS/sources
```

Then set the environment variable `LFS` to `/mnt/lfs` to follow the instructions for obtaining and building the initial set of bootstrapping packages.

## Create the lfs user

In order to minimize the risk of damaging your host system, Linux From Scratch recommends creating a user `lfs` and giving it ownership of the sources directory:

```sh
sudo groupadd lfs
sudo useradd -m -k /dev/null -g lfs -s /bin/bash lfs
sudo chown lfs:lfs /mnt/lfs/sources
su lfs
```

### Setup user environment

See [Setting up the environment](https://www.linuxfromscratch.org/lfs/view/stable-systemd/chapter04/settingenvironment.html) for the rationale, but Linux From Scratch has your `lfs` user setting the following variables in dotfiles:

```sh
cat > ~/.bash_profile << "EOF"
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF

cat > ~/.bashrc << "EOF"
set +h
umask 022
LFS=/mnt/lfs
LC_ALL=POSIX
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/usr/bin
if [ ! -L /bin ]; then PATH=/bin:$PATH; fi
PATH=$LFS/tools/bin:$PATH
CONFIG_SITE=$LFS/usr/share/config.site
export LFS LC_ALL LFS_TGT PATH CONFIG_SITE
EOF
```

## Obtaining the sources

From the host system, download the list of packages and checksums provided by Linux From Scratch and use it to download the required packages into the `$LFS/sources` directory:

```sh
wget https://www.linuxfromscratch.org/lfs/view/systemd/wget-list &&
wget https://www.linuxfromscratch.org/lfs/view/systemd/md5sums   &&

wget --input-file=wget-list --continue --directory-prefix=$LFS/sources/base &&

cp md5sums $LFS/sources/base &&
pushd $LFS/sources/base      &&
  md5sum -c md5sums          &&
popd
```

### Updates from LFS

We will be using packages from the development version of Linux From Scratch, rather than stable LFS, as well as `linux-5.12.13`, which is the latest stable version of the Linux kernel. Download these and replace the versions provided by `LFS`:

```sh
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.12.13.tar.xz -P $LFS/sources/base &&
rm  $LFS/sources/base/linux-5.12.10.tar.xz
```

### Add-ons

These additional packages are not in the default LFS list, but are needed for `systemd-boot` to work, which we will be using in lieu of `GRUB`:

```sh
wget https://github.com/kontais/gnu-efi/archive/download/3.0.12/gnuefi-3.0.12.tar.gz -P $LFS/sources &&
wget https://github.com/rhboot/efivar/releases/download/37/efivar-37.tar.bz2 -P $LFS/sources &&
wget https://www.linuxfromscratch.org/patches/blfs/svn/efivar-37-gcc_9-1.patch -P $LFS/sources &&
rm grub-2.06~rc1.tar.xz
```

We will also download `isl-0.24`, which is an optional dependency of `gcc` for manipulating constrained integer sets:

```sh
wget http://isl.gforge.inria.fr/isl-0.24.tar.xz -O $LFS/sources/isl-0.24.tar.xz
```

We will also add `curl`, `openssh`, `pam`, and `sudo` to make life a bit easier for using the new system as a headless VM, allowing connection without needing to use a management console. Even for a bare metal headless system, this can be handy since you can utilize a graphical web browser and cut and paste features via `ssh` from a terminal emulator rather than the bare console.

```sh
wget https://curl.se/download/curl-7.76.1.tar.gz -P $LFS/sources &&
wget https://mirror.esc7.net/pub/OpenBSD/OpenSSH/portable/openssh-8.6p1.tar.gz -P $LFS/sources &&
wget https://github.com/linux-pam/linux-pam/releases/download/v1.5.1/Linux-PAM-1.5.1.tar.xz -P $LFS/sources &&
wget https://www.sudo.ws/dist/sudo-1.9.7p1.tar.gz -P $LFS/sources
```

We will also be adding `libtasn`, `p11-kit`, `make-ca`, `nettle`, `libunistring`, `GnuTLS`, `PCRE`, and `zsh`:

```sh
wget https://ftp.gnu.org/gnu/libtasn1/libtasn1-4.16.0.tar.gz -P $LFS/sources &&
wget https://github.com/p11-glue/p11-kit/releases/download/0.24.0/p11-kit-0.24.0.tar.xz -P $LFS/sources &&
wget https://github.com/djlucas/make-ca/releases/download/v1.7/make-ca-1.7.tar.xz -P $LFS/sources &&
wget https://ftp.gnu.org/gnu/nettle/nettle-3.7.3.tar.gz -P $LFS/sources &&
wget https://ftp.gnu.org/gnu/libunistring/libunistring-0.9.10.tar.xz -P $LFS/sources &&
wget https://www.gnupg.org/ftp/gcrypt/gnutls/v3.7/gnutls-3.7.0.tar.xz -P $LFS/sources &&
wget https://ftp.pcre.org/pub/pcre/pcre-8.44.tar.bz2 -P $LFS/sources &&
wget http://www.zsh.org/pub/zsh-5.8.tar.xz -P $LFS/sources
```

Get the `BLFS` `systemd` units:

```sh
wget http://www.linuxfromscratch.org/blfs/downloads/10.1-systemd/blfs-systemd-units-20210122.tar.xz -P $LFS/sources
```

Finally, get the `which` utility, which is handy for finding paths to executables. Many packages in Beyond Linux From Scratch require this for their configure and build processes to detect the presence of dependencies and you are likely to be used to using it anyway.

```sh
wget https://carlowood.github.io/which/which-2.21.tar.gz -P $LFS/sources
```
