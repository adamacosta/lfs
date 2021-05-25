# Clean up the temporary system

```sh
find /usr/{lib,libexec} -name \*.la -delete
```

## Exit the chroot and unmount VFS

```sh
exit
umount $LFS/dev{/pts,}
umount $LFS/{sys,proc,run}
```

## Strip unneeded symbols

```sh
strip --strip-debug $LFS/usr/lib/*
strip --strip-unneeded $LFS/usr/{,s}bin/*
strip --strip-unneeded $LFS/tools/bin/*
```

## Backup the temporary system

```sh
cd $LFS
tar cJpvf $HOME/lfs-temp-tools-10.1-systemd.tar.xz .
```

If a restore is needed, run:

```sh
cd $LFS
rm -rf ./*
tar xpvf $HOME/lfs-temp-tools-10.1-systemd.tar.xz
```