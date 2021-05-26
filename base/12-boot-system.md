# Make LFS system bootable

## Setup /etc/fstab

This is specific to a dual boot system, so using the common swap and boot partitions.

First, get the UUID for the root filesystem:

```sh
ROOT_UUID=$(blkid -s UUID -o value /dev/sdb1)
```

Then copy in the build system fstab but with this new filesystem replacing the root.

```sh
cat > /etc/fstab << EOF
# <file system> <dir> <type> <options> <dump> <pass>
# /dev/sdb1
UUID=${ROOT_UUID}       /               ext4            rw,relatime     0 1

# /dev/sda1
UUID=F378-ABD7          /boot           vfat            rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro       0 2

# /dev/sda2
UUID=3f1d6649-da80-4b7a-a150-1e0fdba130a0       none            swap            defaults        0 0
EOF
```

## Compile the Linux kernel

Again, from `$LFS/sources`:

```sh
tar xvf linux-5.10.17.tar.xz
cd linux-5.10.17
```

The kernel does not use a `./configure` script, instead relying on a menu-based config generator that is created by running `make`. First, create a default config with reasonable defaults for your detected system:

```sh
make mrproper && make defconfig
```

This can then be customized using the `ncurses` based graphical config editor (replace `LANG` with your own locale):

```sh
make LANG=en_US.UTF-8 make menuconfig
```

In order to run under Microsoft Hyper-V, all of the following options need to be set:

```
CONFIG_HYPERVISOR_GUEST: Processor type and featueres > Linux Guest Support
CONFIG_PARAVIRT: Processor type and features > Linux Guest Support > Enable paravirtualization code
CONFIG_PARAVIRT_SPINLOCKS: Processor type and features > Linux Guest Support > Paravirtualization layer for spinlocks
CONFIG_CONNECTOR: Device Drivers > Connector - unified userspace <-> kernelspase linker
CONFIG_SCSI_FC_ATTRS: Device Drivers > SCSI device support > SCSI Transports > FiberChannel Transport Attributes
CONFIG_HYPERV: Device Drivers > Microsoft Hyper-V guest support > Microsoft Hyper-V client drivers
CONFIG_HYPERV_UTILS: Device Drivers > Microsoft Hyper-V guest support > Microsoft Hyper-V Utilities driver
CONFIG_HYPERV_BALLOON: Device Drivers > Microsoft Hyper-V guest support > Microsoft Hyper-V Balloon driver
CONFIG_HYPERV_STORAGE: Device Drivers > SCSI device support > SCSI low-level drivers > Microsoft Hyper-V virtual storage driver
CONFIG_HYPERV_NET: Device Drivers > Network device support > Microsoft Hyper-V virtual network driver
CONFIG_HYPERV_KEYBOARD: Device Drivers > Input device support > Hardware I/O ports > Microsoft Synthetic Keyboard driver
CONFIG_FB_HYPERV: Device Drivers > Graphics support > Frame buffer Devices > Microsoft Hyper-V Synthetic Video support
CONFIG_HID_HYPERV_MOUSE: Device Drivers > HID support > Special HID drivers > Microsoft Hyper-V mouse driver
CONFIG_PCI_HYPERV: Device Driver > PCI Support > Hyper-V PCI Frontend
CONFIG_VSOCKETS: Networking support > Networking options > Virtual Socket protocol
CONFIG_HYPERV_VSOCKETS: Networking support > Networking options > Hyper-V transport for Virtual Sockets
```

Also ensure support for EFI is enabled:

```
CONFIG_EFI_VARS=y : Firmware Drivers > EFI (Extensible Firmware Interface) Support > EFI Variable Support via sysfs
CONFIG_EFI_RUNTIME_MAP=y : Firmware Drivers > EFI (Extensible Firmware Interface) Support > Export EFI runtime maps to sysfs
CONFIG_EFI_BOOTLOADER_CONTROL=y : Firmware Drivers > EFI (Extensible Firmware Interface) Support > EFI Bootloader Control
CONFIG_EFIVAR_FS=y : File systems > Pseudo filesystems > EFI Variable filesystem
```

Finally, ensure whichever filesystem(s) you chose to create on your partitions are enabled.

Once the final `.config` file has been generated, build the kernel and install modules (provided you enabled module support):

```sh
make
make modules_install
```

From outside of the chroot, bind mount the host `/boot` directory to `/boot` inside the chroot:

```sh
mount --bind /boot /mnt/lfs/boot
```

Now install the kernel image into the boot partition:

```sh
cp -iv arch/x86/boot/bzImage /boot/vmlinuz-5.10.17-lfs-10.1-systemd
```

To also install the symbol map and `.config` file for debugging purposes:

```sh
cp -iv System.map /boot/System.map-5.10.17
cp -iv .config /boot/config-5.10.17
```

Finally, install the kernel documentation:

```sh
install -d /usr/share/doc/linux-5.10.17
cp -r Documentation/* /usr/share/doc/linux-5.10.17
```

### Microsoft Hyper-V integration services for Linux

This is only needed if you intend to run your system in a VM on a Microsoft Hyper-V hypervisor.

```sh
cd linux-5.10.17/tools/hv

CFLAGS+=' -DKVP_SCRIPTS_PATH=\"/usr/lib/hyperv/kvp_scripts/\"' make
make install
install -dm755 /usr/lib/hyperv/kvp_scripts
```

The `systemd` units will need to be manually created:

```sh
cat > /usr/lib/systemd/system/hv_fcopy_daemon.service << EOF
[Unit]
Description=Hyper-V file copy service (FCOPY)
ConditionPathExists=/dev/vmbus/hv_fcopy

[Service]
ExecStart=/usr/bin/hv_fcopy_daemon -n

[Install]
WantedBy=multi-user.target
EOF
cat > /usr/lib/systemd/system/hv_kvp_daemon.service << EOF
[Unit]
Description=Hyper-V key-value pair (KVP)
ConditionPathExists=/dev/vmbus/hv_kvp

[Service]
ExecStart=/usr/bin/hv_kvp_daemon -n

[Install]
WantedBy=multi-user.target
EOF
cat > /usr/lib/systemd/system/hv_vss_daemon.service << EOF
[Unit]
Description=Hyper-V volume shadow copy service (VSS)
ConditionPathExists=/dev/vmbus/hv_vss

[Service]
ExecStart=/usr/bin/hv_vss_daemon -n

[Install]
WantedBy=multi-user.target
EOF

systemctl enable hv_fcopy_daemon.service
systemctl enable hv_kvp_daemon.service
systemctl enable hv_vss_daemon.service
```

## Bootloader configuration

This will be specific to your bootloader, but I am using `systemd-boot` and building from an Arch Linux system, which is the default. For this, the `PARTUUID` is needed to pass to the kernel instead of the `UUID`, which will not yet be available in early userspace.

```sh
ROOT_UUID=$(blkid -s PARTUUID -o value /dev/sdb1)

cat > /boot/loader/entries/lfs.conf << EOF
title   Linux From Scratch
linux   /vmlinuz-5.10.17-lfs-10.1-systemd
options root="PARTUUID=$ROOT_UUID" rw loglevel=7 earlyprintk=vga,keep rootfstype=ext4
EOF
```

It is important to use the PARTUUID here and not the UUID or a label. These can only be used if you create an `initrd`, which will have tools for detecting those. That is outside the scope of base Linux From Scratch, as the main purpose of `initrd` is providing the ability to distribute generic kernels where everything needed is autodetected at boot time and loaded as modules. Here we have a fully customized system where that should not be necessary.

For `systemd-boot`, it is also important to not have a timeout of 0 in the `/boot/loader/loader.conf` file, or to make `lfs` the default entry. Otherwise, it will always boot into whatever system is the default entry.

## Reboot into LFS

Exit the chroot environment, unmount `/mnt/lfs`, and reboot the system. When the boot menu appears, select `Linux From Scratch`.
