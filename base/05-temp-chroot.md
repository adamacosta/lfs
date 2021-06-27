# Prepare the chroot environment for temporary system

At this point, we are ready to compile everything needed for the LFS system from within the system itself, so we will generate a `chroot` environment with all of the needed virtual filesystems underneath the `$LFS` mount and perform the remainder of our build and setup inside of this isolated root.

First, switch back to root: `sudo su -` or `exit`, depending on if you were root before or switched from another user.

## chown all the system files

Right now the `lfs` user only exists on the host system, so we want to avoid having files owned by an invalid uid in the new system. uid 0 will always be valid, so change ownership of the new system's directory tree back to `root`:

```sh
chown -R root:root $LFS/{usr,lib,lib64,var,etc,bin,sbin,tools}
```

## Prepare the kernel virtual filesystems

We now need to create and mount all of the virtual filesystems exported from the kernel to userspace, as we would otherwise not be able to communicate with the kernel from inside our chroot.

### Create the root directories

```sh
mkdir -pv $LFS/{dev,proc,sys,run}
```

### Create the initial device nodes

```sh
mknod -m 600 $LFS/dev/console c 5 1 &&
mknod -m 666 $LFS/dev/null    c 1 3
```

### Bind mount /dev

```sh
mount -v --bind /dev     $LFS/dev &&
mount -v --bind /dev/pts $LFS/dev/pts
```

### Mount the virtual kernel filesystems

```sh
mount -vt proc proc   $LFS/proc &&
mount -vt sysfs sysfs $LFS/sys  &&
mount -vt tmpfs tmpfs $LFS/run
```

### Mount the efivars

```sh
mount -vt efivarfs efivarfs $LFS/sys/firmware/efi/efivars/
```

## Enter the chroot environment

Note that `+h` argument to `bash`, telling it not to hash the paths to executables. This prevents it from using old versions of tools we are reinstalling in-place and allows picking up new tools instantly.

```sh
chroot "$LFS" /usr/bin/env -i          \
    HOME=/root                         \
    TERM="$TERM"                       \
    PS1='(lfs chroot) \u:\w\$ '        \
    PATH=/bin:/usr/bin:/sbin:/usr/sbin \
    /bin/bash --login +h
```

For the sake of convenience when moving back and forth between the host system and the chroot, we can create a script that does all of this, similar to the `arch-chroot` tool included with the Arch Linux installation media.

```sh
cat > lfs-chroot << EOF
#!/bin/sh

export LFS=/mnt/lfs

mkdir -p $LFS/sources/build_dir

mount -v --bind /dev              $LFS/dev
mount -v --bind /dev/pts          $LFS/dev/pts
mount -vt       proc proc         $LFS/proc
mount -vt       sysfs sysfs       $LFS/sys
mount -vt       tmpfs tmpfs       $LFS/run
mount -vt       tmpfs tmpfs       $LFS/sources/build_dir
mount -vt       efivarfs efivarfs $LFS/sys/firmware/efi/efivars/

chroot "$LFS" /usr/bin/env -i          \
    HOME=/root                         \
    TERM="$TERM"                       \
    PS1='(lfs chroot) \u:\w\$ '        \
    PATH=/bin:/usr/bin:/sbin:/usr/sbin \
    /bin/bash --login +h
EOF
chmod +x lfs-chroot
```

Then use that to enter the LFS chroot:

```sh
./lfs-chroot
```
