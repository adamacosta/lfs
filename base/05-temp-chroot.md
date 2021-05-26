## Prepare the chroot environment for temporary system

First, switch back to root: `sudo su -`

### chown all the system files

```sh
chown -R root:root $LFS/{usr,lib,lib64,var,etc,bin,sbin,tools}
```

### Prepare the kernel virtual filesystems

#### Create the root directories

```sh
mkdir -pv $LFS/{dev,proc,sys,run}
```

#### Create the initial device nodes

```sh
mknod -m 600 $LFS/dev/console c 5 1 &&
mknod -m 666 $LFS/dev/null    c 1 3
```

#### Bind mount /dev

```sh
mount -v --bind /dev     $LFS/dev &&
mount -v --bind /dev/pts $LFS/dev/pts
```

#### Mount the virtual kernel filesystems

```sh
mount -vt proc proc   $LFS/proc &&
mount -vt sysfs sysfs $LFS/sys  &&
mount -vt tmpfs tmpfs $LFS/run
```

#### Mount the efivars

```sh
mount -vt efivarfs efivarfs $LFS/sys/firmware/efi/efivars/
```