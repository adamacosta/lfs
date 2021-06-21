# Make LFS system bootable

## Setup /etc/fstab

This is specific to a dual boot system, so using the common swap and boot partitions.

First, from outside the chroot, copy in the host system `/etc/fstab` to the lfs `/etc/fstab` and grab the UUID for your root partition:

```sh
sudo cp /etc/fstab $LFS/etc/fstab
blkid -s UUID -o value /dev/<root device>
```

Get the UUID for the root filesystem (insert whatever you earlier chose as the root device), then edit the `fstab` to use your new root device instead of the host system's:

```sh
sudo vi /etc/fstab
```

## Compile the Linux kernel

First, change the owner of the source packages to the new user:

```sh
sudo chown -R lfs:lfs /sources
```

From `$LFS/sources/build_dir`:

```sh
tar xvf /sources/linux-5.12.12.tar.xz &&
cd       linux-5.12.12
```

The kernel does not use a `./configure` script, instead relying on a menu-based config generator that is created by running `make`. First, create a default config with reasonable defaults for your detected system:

```sh
make mrproper && make defconfig
```

This can then be customized using the `ncurses` based graphical config editor (replace `LANG` with your own locale):

```sh
LANG=en_US.UTF-8 make menuconfig
```

### UEFI Booting

Set the following to ensure support for EFI is enabled if you have a UEFI motherboard or are booting on a UEFI virtual device:

```
Firmware Drivers --->
  EFI (Extensible Firmware Interface) Support --->
    <*> EFI Variable Support via sysfs
    [*] Export efi runtime maps to sysfs
    <*> EFI Bootloader Control

File systems --->
  Pseudo filesystems --->
    <*> EIF Variable filesystem
```

### Microcode

To enable support for loading microcode updates:

#### AMD

```
General Setup --->
  [*] Initial RAM filesystem and RAM disk (initramfs/initrd) support [CONFIG_BLK_DEV_INITRD]
Processor type and features  --->
  [*] CPU microcode loading support  [CONFIG_MICROCODE]
  [*]      AMD microcode loading support [CONFIG_MICROCODE_AMD]
```

#### Intel

```
General Setup --->
  [*] Initial RAM filesystem and RAM disk (initramfs/initrd) support [CONFIG_BLK_DEV_INITRD]
Processor type and features  --->
  [*] CPU microcode loading support  [CONFIG_MICROCODE]
  [*]      Intel microcode loading support [CONFIG_MICROCODE_INTEL]
```

### LVM and RAID

If LVM and/or raid are desired, those will need to be enabled as well:

```
Device Drivers --->
  [*] Multiple devices driver support (RAID and LVM) ---> [CONFIG_MD]
    <*> RAID support                                      [CONFIG_BLK_DEV_MD]
    [*]   Autodetect RAID arrays during kernel boot       [CONFIG_MD_AUTODETECT]
    <*/M>  Linear (append) mode                           [CONFIG_MD_LINEAR]
    <*/M>  RAID-0 (striping) mode                         [CONFIG_MD_RAID0]
    <*/M>  RAID-1 (mirroring) mode                        [CONFIG_MD_RAID1]
    <*/M>  RAID-10 (mirrored striping) mode               [CONFIG_MD_RAID10]
    <*/M>  RAID-4/RAID-5/RAID-6 mode                      [CONFIG_MD_RAID456]

Device Drivers --->
  [*] Multiple devices driver support (RAID and LVM) ---> [CONFIG_MD]
    <*/M>   Device mapper support                         [CONFIG_BLK_DEV_DM]
    <*/M>   Crypt target support                          [CONFIG_DM_CRYPT]
    <*/M>   Snapshot target                               [CONFIG_DM_SNAPSHOT]
    <*/M>   Thin provisioning target                      [CONFIG_DM_THIN_PROVISIONING]
    <*/M>   Mirror target                                 [CONFIG_DM_MIRROR]

Kernel hacking --->
  Generic Kernel Debugging Instruments --->
    [*] Magic SysRq key                                   [CONFIG_MAGIC_SYSRQ]
```

If you have `NVMe` devices you need to read, ensure you enable support for `NVMe`:

```
Device Drivers --->
  NVME Support --->
    <*> NVM Expess block device
    [*] NVMe multipath support
    [*] NVMe hardware monitoring
```

There will be options for NVMe over Fabric, which allows you expose a disk over the network. You will likely know if you need that.

If you have an `nvidia` video card and wish to use the proprietary drivers, ensure you do not enable the `nouveau` device drivers.

### Running LFS in a VM

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

### Running as a hypervisor

To include support for virtualization in the kernel, include the following options, noting that only one of `CONFIG_KVM_INTEL` or `CONFIG_KVM_AMD` is needed, depending on which processor type you have.

```
[*] Virtualization:  --->                                             [CONFIG_VIRTUALIZATION]
  <*/M>   Kernel-based Virtual Machine (KVM) support [CONFIG_KVM]
  <*/M>     KVM for Intel (and compatible) processors support         [CONFIG_KVM_INTEL]
  <*/M>     KVM for AMD processors support                            [CONFIG_KVM_AMD]

[*] Networking support  --->                         [CONFIG_NET]
  Networking options  --->
    <*/M> 802.1d Ethernet Bridging                   [CONFIG_BRIDGE]
Device Drivers  --->
  [*] Network device support  --->                   [CONFIG_NETDEVICES]
    <*/M>    Universal TUN/TAP device driver support [CONFIG_TUN]
```

To see if your processor supports virtualization (vmx is Intel, svm is AMD):

```sh
egrep --color=auto '(vmx|svm)' /proc/cpuinfo
```

Beware that this will print out the information for every processor core, so if you have a many core processor, you may want to filter to just look at the first one:

```sh
cat /proc/cpuinfo | grep -A19 'processor.*: 0' | egrep --color=auto '(vmx|svm)'
```

### Filesystem support

Ensure whichever filesystem(s) you chose to create on your partition(s) are enabled. The filesystem driver for your root filesystem and for FAT32 to read the ESP should be built-in, but you may want to build modules for other filesystems in order to be able to mount them at runtime.

### WiFi support

To be able to connect to a WiFi network, at minimum, you will need enable the following:

```
[*] Networking support  --->
    [*] Wireless  --->
        <M>   cfg80211 - wireless configuration API
        <M>   Generic IEEE 802.11 Networking Stack (mac80211)
```

You will also need the device drivers for your WiFi card, but which specific driver(s) you need entirely depends on your WiFi card. You may also need firmware. One important detail to note is that, if you need firmware, then you need to compile the drivers as modules, not built-in.

### Building

Once the final `.config` file has been generated, build the kernel and install modules (provided you enabled module support), you can perform the build. If you are updating or changing the kernel, you may want to diff the `.config` file with the options saved off from your last kernel build, or even just use the same file if you didn't intend to change anything. Note that if the update is more than a patch, config options may have changed, so be careful blindly applying the same config.

```sh
make
sudo make modules_install
```

From outside of the chroot, bind mount the host `/boot` directory to `/boot` inside the chroot:

```sh
sudo mount --bind /boot /mnt/lfs/boot
```

Now install the kernel image into the boot partition:

```sh
sudo cp -iv arch/x86/boot/bzImage /boot/vmlinuz-5.12.12-lfs
```

To also install the symbol map and `.config` file for debugging purposes:

```sh
sudo cp -iv System.map /boot/System.map-5.12.12 &&
sudo cp -iv .config /boot/config-5.12.12
```

To install the kernel documentation:

```sh
sudo install -d /usr/share/doc/linux-5.12.12 &&
sudo cp -r Documentation/* /usr/share/doc/linux-5.12.12
```

### Microsoft Hyper-V integration services for Linux

This is only needed if you intend to run your system in a VM on a Microsoft Hyper-V hypervisor.

```sh
cd linux-5.12.12/tools/hv

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

### Installation to the EFI system partition

This will be specific to your bootloader, but I am using `systemd-boot`. To install the bootloader:

```sh
bootctl install
```

Now create a configuration for the bootloader. To see all available options you might want to set, read [Systemd-boot#Loader_configuration](https://wiki.archlinux.org/title/Systemd-boot#Loader_configuration) from the Arch Wiki.

```sh
cat > /boot/loader/loader.conf << EOF
default  lfs.conf
timeout  5
console-mode max
editor   no
EOF
```

For booting without `initrd`, the `PARTUUID` is needed to pass to the kernel instead of the `UUID`, which will not yet be available in early userspace.

```sh
ROOT_UUID=$(sudo blkid -s PARTUUID -o value /dev/<root device>)
FSTYPE=$(sudo blkid -s TYPE -o value /dev/<root device>)
```

Then add a bootloader configuration for the new system:

```sh
cat > /boot/loader/entries/lfs.conf << EOF
title   Linux From Scratch
linux   /vmlinuz-5.12.9-lfs-10.1-systemd
options root="PARTUUID=$ROOT_UUID" rw loglevel=7 earlyprintk=vga,keep rootfstype=<filesystem type>
EOF
```

It is important to use the PARTUUID here and not the UUID or a label. These can only be used if you create an `initrd`, which will have tools for detecting those. That is outside the scope of base Linux From Scratch, as the main purpose of `initrd` is providing the ability to distribute generic kernels where everything needed is autodetected at boot time and loaded as modules. Here we have a fully customized system where that should not be necessary.

For `systemd-boot`, it is also important to not have a timeout of 0 in the `/boot/loader/loader.conf` file, or to make `lfs` the default entry. Otherwise, it will always boot into whatever system is the default entry.

### LVM and RAID

If your root filesystem is a logical or raid volume, the above will not work. There will be no PARTUUID and the kernel in early userspace will not have the tools necessary to find your root volume. Instead, you will need to create an `initrd` to load before loading the kernel. You can create and use the scripts provided by Linux From Scratch. First, however, you will want to install `cpio` and possibly microcode from your processor vendor.

#### cpio

First, grab the source from [https://ftp.gnu.org/gnu/cpio/cpio-2.13.tar.bz2](https://ftp.gnu.org/gnu/cpio/cpio-2.13.tar.bz2). From outside the chroot (since LFS will not have network capability):

```sh
wget https://ftp.gnu.org/gnu/cpio/cpio-2.13.tar.bz2 -O $LFS/sources/cpio-2.13.tar.bz2
```

Then to build it:

```sh
tar xvf /sources/cpio-2.13.tar.bz2 &&
cd       cpio-2.13                 &&

sed -i '/The name/,+2 d' src/global.c   &&
./configure --prefix=/usr \
            --enable-mt   \
            --with-rmt=/usr/libexec/rmt &&

make                                                      &&
makeinfo --html            -o doc/html      doc/cpio.texi &&
makeinfo --html --no-split -o doc/cpio.html doc/cpio.texi &&
makeinfo --plaintext       -o doc/cpio.txt  doc/cpio.texi &&

sudo make install                                      &&
sudo install -v -m755 -d /usr/share/doc/cpio-2.13/html &&
sudo install -v -m644    doc/html/* \
                         /usr/share/doc/cpio-2.13/html &&
sudo install -v -m644    doc/cpio.{html,txt} \
                         /usr/share/doc/cpio-2.13      &&

cd .. &&
rm -rf cpio-2.13
```

#### mkinitramfs scripts

```sh
cat > /usr/bin/mkinitramfs << "EOF"
#!/bin/bash
# This file based in part on the mkinitramfs script for the LFS LiveCD
# written by Alexander E. Patrakov and Jeremy Huntwork.

copy()
{
  local file

  if [ "$2" = "lib" ]; then
    file=$(PATH=/usr/lib type -p $1)
  else
    file=$(type -p $1)
  fi

  if [ -n "$file" ] ; then
    cp $file $WDIR/usr/$2
  else
    echo "Missing required file: $1 for directory $2"
    rm -rf $WDIR
    exit 1
  fi
}

if [ -z $1 ] ; then
  INITRAMFS_FILE=initrd.img-no-kmods
else
  KERNEL_VERSION=$1
  INITRAMFS_FILE=initrd.img-$KERNEL_VERSION
fi

if [ -n "$KERNEL_VERSION" ] && [ ! -d "/usr/lib/modules/$1" ] ; then
  echo "No modules directory named $1"
  exit 1
fi

printf "Creating $INITRAMFS_FILE... "

binfiles="sh cat cp dd killall ls mkdir mknod mount "
binfiles="$binfiles umount sed sleep ln rm uname"
binfiles="$binfiles readlink basename"

# Systemd installs udevadm in /bin. Other udev implementations have it in /sbin
if [ -x /usr/bin/udevadm ] ; then binfiles="$binfiles udevadm"; fi

sbinfiles="modprobe blkid switch_root"

# Optional files and locations
for f in mdadm mdmon udevd udevadm; do
  if [ -x /usr/bin/$f ] ; then sbinfiles="$sbinfiles $f"; fi
done

# Add lvm if present (cannot be done with the others because it
# also needs dmsetup
if [ -x /usr/bin/lvm ] ; then sbinfiles="$sbinfiles lvm dmsetup"; fi

unsorted=$(mktemp /tmp/unsorted.XXXXXXXXXX)

DATADIR=/usr/share/mkinitramfs
INITIN=init.in

# Create a temporary working directory
WDIR=$(mktemp -d /tmp/initrd-work.XXXXXXXXXX)

# Create base directory structure
mkdir -p $WDIR/{dev,run,sys,proc,usr/{bin,lib/{firmware,modules},sbin}}
if [ -d "/usr/lib/modules/$1" ] ; then
  mkdir -p $WDIR/usr/lib/modules/$1
fi
mkdir -p $WDIR/etc/{modprobe.d,udev/rules.d}
touch $WDIR/etc/modprobe.d/modprobe.conf
ln -s usr/bin  $WDIR/bin
ln -s usr/lib  $WDIR/lib
ln -s usr/sbin $WDIR/sbin
ln -s lib      $WDIR/lib64

# Create necessary device nodes
mknod -m 640 $WDIR/dev/console c 5 1
mknod -m 664 $WDIR/dev/null    c 1 3

# Install the udev configuration files
if [ -f /etc/udev/udev.conf ]; then
  cp /etc/udev/udev.conf $WDIR/etc/udev/udev.conf
fi

for file in $(find /etc/udev/rules.d/ -type f) ; do
  cp $file $WDIR/etc/udev/rules.d
done

# Install any firmware present
cp -a /usr/lib/firmware $WDIR/usr/lib

# Copy the RAID configuration file if present
if [ -f /etc/mdadm.conf ] ; then
  cp /etc/mdadm.conf $WDIR/etc
fi

# Install the init file
install -m0755 $DATADIR/$INITIN $WDIR/init

if [  -n "$KERNEL_VERSION" ] ; then
  if [ -x /usr/bin/kmod ] ; then
    binfiles="$binfiles kmod"
  else
    binfiles="$binfiles lsmod"
    sbinfiles="$sbinfiles insmod"
  fi
fi

# Install basic binaries
for f in $binfiles ; do
  ldd /usr/bin/$f | sed "s/\t//" | cut -d " " -f1 >> $unsorted
  copy /usr/bin/$f bin
done

for f in $sbinfiles ; do
  ldd /usr/sbin/$f | sed "s/\t//" | cut -d " " -f1 >> $unsorted
  copy $f sbin
done

# Add udevd libraries if not in /usr/sbin
if [ -x /usr/lib/udev/udevd ] ; then
  ldd /usr/lib/udev/udevd | sed "s/\t//" | cut -d " " -f1 >> $unsorted
elif [ -x /usr/lib/systemd/systemd-udevd ] ; then
  ldd /usr/lib/systemd/systemd-udevd | sed "s/\t//" | cut -d " " -f1 >> $unsorted
fi

# Add module symlinks if appropriate
if [ -n "$KERNEL_VERSION" ] && [ -x /usr/bin/kmod ] ; then
  ln -s kmod $WDIR/usr/bin/lsmod
  ln -s kmod $WDIR/usr/bin/insmod
fi

# Add lvm symlinks if appropriate
# Also copy the lvm.conf file
if  [ -x /usr/sbin/lvm ] ; then
  ln -s lvm $WDIR/usr/sbin/lvchange
  ln -s lvm $WDIR/usr/sbin/lvrename
  ln -s lvm $WDIR/usr/sbin/lvextend
  ln -s lvm $WDIR/usr/sbin/lvcreate
  ln -s lvm $WDIR/usr/sbin/lvdisplay
  ln -s lvm $WDIR/usr/sbin/lvscan

  ln -s lvm $WDIR/usr/sbin/pvchange
  ln -s lvm $WDIR/usr/sbin/pvck
  ln -s lvm $WDIR/usr/sbin/pvcreate
  ln -s lvm $WDIR/usr/sbin/pvdisplay
  ln -s lvm $WDIR/usr/sbin/pvscan

  ln -s lvm $WDIR/usr/sbin/vgchange
  ln -s lvm $WDIR/usr/sbin/vgcreate
  ln -s lvm $WDIR/usr/sbin/vgscan
  ln -s lvm $WDIR/usr/sbin/vgrename
  ln -s lvm $WDIR/usr/sbin/vgck
  # Conf file(s)
  cp -a /etc/lvm $WDIR/etc
fi

# Install libraries
sort $unsorted | uniq | while read library ; do
# linux-vdso and linux-gate are pseudo libraries and do not correspond to a file
# libsystemd-shared is in /lib/systemd, so it is not found by copy, and
# it is copied below anyway
  if [[ "$library" == linux-vdso.so.1 ]] ||
     [[ "$library" == linux-gate.so.1 ]] ||
     [[ "$library" == libsystemd-shared* ]]; then
    continue
  fi

  copy $library lib
done

if [ -d /usr/lib/udev ]; then
  cp -a /usr/lib/udev $WDIR/usr/lib
fi
if [ -d /usr/lib/systemd ]; then
  cp -a /usr/lib/systemd $WDIR/usr/lib
fi
if [ -d /usr/lib/elogind ]; then
  cp -a /usr/lib/elogind $WDIR/usr/lib
fi

# Install the kernel modules if requested
if [ -n "$KERNEL_VERSION" ]; then
  find                                                                            \
     /usr/lib/modules/$KERNEL_VERSION/kernel/{crypto,fs,lib}                      \
     /usr/lib/modules/$KERNEL_VERSION/kernel/drivers/{block,ata,md,firewire}      \
     /usr/lib/modules/$KERNEL_VERSION/kernel/drivers/{scsi,message,pcmcia,virtio} \
     /usr/lib/modules/$KERNEL_VERSION/kernel/drivers/usb/{host,storage}           \
     -type f 2> /dev/null | cpio --make-directories -p --quiet $WDIR

  cp /usr/lib/modules/$KERNEL_VERSION/modules.{builtin,order}                     \
            $WDIR/usr/lib/modules/$KERNEL_VERSION

  depmod -b $WDIR $KERNEL_VERSION
fi

( cd $WDIR ; find . | cpio -o -H newc --quiet | gzip -9 ) > $INITRAMFS_FILE

# Prepare early loading of microcode if available
if ls /usr/lib/firmware/intel-ucode/* >/dev/null 2>&1 ||
   ls /usr/lib/firmware/amd-ucode/*   >/dev/null 2>&1; then

# first empty WDIR to reuse it
  rm -r $WDIR/*

  DSTDIR=$WDIR/kernel/x86/microcode
  mkdir -p $DSTDIR

  if [ -d /usr/lib/firmware/amd-ucode ]; then
    cat /usr/lib/firmware/amd-ucode/microcode_amd*.bin > $DSTDIR/AuthenticAMD.bin
  fi

  if [ -d /usr/lib/firmware/intel-ucode ]; then
    cat /usr/lib/firmware/intel-ucode/* > $DSTDIR/GenuineIntel.bin
  fi

  ( cd $WDIR; find . | cpio -o -H newc --quiet ) > microcode.img
  cat microcode.img $INITRAMFS_FILE > tmpfile
  mv tmpfile $INITRAMFS_FILE
  rm microcode.img
fi

# Remove the temporary directories and files
rm -rf $WDIR $unsorted
printf "done.\n"

EOF
```

```sh
chmod 0755 /usr/bin/mkinitramfs &&
mkdir -p /usr/share/mkinitramfs
```

```sh
cat > /usr/share/mkinitramfs/init.in << "EOF"
#!/bin/sh

PATH=/usr/bin:/usr/sbin
export PATH

problem()
{
   printf "Encountered a problem!\n\nDropping you to a shell.\n\n"
   sh
}

no_device()
{
   printf "The device %s, which is supposed to contain the\n" $1
   printf "root file system, does not exist.\n"
   printf "Please fix this problem and exit this shell.\n\n"
}

no_mount()
{
   printf "Could not mount device %s\n" $1
   printf "Sleeping forever. Please reboot and fix the kernel command line.\n\n"
   printf "Maybe the device is formatted with an unsupported file system?\n\n"
   printf "Or maybe filesystem type autodetection went wrong, in which case\n"
   printf "you should add the rootfstype=... parameter to the kernel command line.\n\n"
   printf "Available partitions:\n"
}

do_mount_root()
{
   mkdir /.root
   [ -n "$rootflags" ] && rootflags="$rootflags,"
   rootflags="$rootflags$ro"

   case "$root" in
      /dev/*    ) device=$root ;;
      UUID=*    ) eval $root; device="/dev/disk/by-uuid/$UUID" ;;
      PARTUUID=*) eval $root; device="/dev/disk/by-partuuid/$PARTUUID" ;;
      LABEL=*   ) eval $root; device="/dev/disk/by-label/$LABEL" ;;
      ""        ) echo "No root device specified." ; problem ;;
   esac

   while [ ! -b "$device" ] ; do
       no_device $device
       problem
   done

   if ! mount -n -t "$rootfstype" -o "$rootflags" "$device" /.root ; then
       no_mount $device
       cat /proc/partitions
       while true ; do sleep 10000 ; done
   else
       echo "Successfully mounted device $root"
   fi
}

do_try_resume()
{
   case "$resume" in
      UUID=* ) eval $resume; resume="/dev/disk/by-uuid/$UUID"  ;;
      LABEL=*) eval $resume; resume="/dev/disk/by-label/$LABEL" ;;
   esac

   if $noresume || ! [ -b "$resume" ]; then return; fi

   ls -lH "$resume" | ( read x x x x maj min x
       echo -n ${maj%,}:$min > /sys/power/resume )
}

init=/sbin/init
root=
rootdelay=
rootfstype=auto
ro="ro"
rootflags=
device=
resume=
noresume=false

mount -n -t devtmpfs devtmpfs /dev
mount -n -t proc     proc     /proc
mount -n -t sysfs    sysfs    /sys
mount -n -t tmpfs    tmpfs    /run

read -r cmdline < /proc/cmdline

for param in $cmdline ; do
  case $param in
    init=*      ) init=${param#init=}             ;;
    root=*      ) root=${param#root=}             ;;
    rootdelay=* ) rootdelay=${param#rootdelay=}   ;;
    rootfstype=*) rootfstype=${param#rootfstype=} ;;
    rootflags=* ) rootflags=${param#rootflags=}   ;;
    resume=*    ) resume=${param#resume=}         ;;
    noresume    ) noresume=true                   ;;
    ro          ) ro="ro"                         ;;
    rw          ) ro="rw"                         ;;
  esac
done

# udevd location depends on version
if [ -x /sbin/udevd ]; then
  UDEVD=/sbin/udevd
elif [ -x /lib/udev/udevd ]; then
  UDEVD=/lib/udev/udevd
elif [ -x /lib/systemd/systemd-udevd ]; then
  UDEVD=/lib/systemd/systemd-udevd
else
  echo "Cannot find udevd nor systemd-udevd"
  problem
fi

${UDEVD} --daemon --resolve-names=never
udevadm trigger
udevadm settle

if [ -f /etc/mdadm.conf ] ; then mdadm -As                       ; fi
if [ -x /sbin/vgchange  ] ; then /sbin/vgchange -a y > /dev/null ; fi
if [ -n "$rootdelay"    ] ; then sleep "$rootdelay"              ; fi

do_try_resume # This function will not return if resuming from disk
do_mount_root

killall -w ${UDEVD##*/}

exec switch_root /.root "$init" "$@"

EOF
```

#### Microcode

Download the appropriate microcode for your processor. Run `grep -F -m 1 "cpu family" /proc/cpuinfo` to get the family. I happen to have an AMD processor family 23, which is hex 17h for the Zen line. 

```sh
curl https://anduin.linuxfromscratch.org/BLFS/linux-firmware/amd-ucode/microcode_amd_fam17h.bin -o /sources/microcode_amd_fam17h.bin &&

sudo mkdir -pv /usr/lib/firmware/amd-ucode &&
sudo cp    -v /sources/microcode_amd_fam17h.bin /usr/lib/firmware/amd-ucode
``` 

Creating the `initrd` will automatically include the microcode:

```sh
sudo mkinitramfs 5.12.11
sudo mv -iv initrd.img-5.12.11 /boot
```

Now copy this into the loader configuration:

```sh
ROOT_UUID=$(blkid -s UUID -o value /dev/<root device>)
ROOT_FSTYPE=$(blkid -s TYPE -o value /dev/<root device>)

cat > /boot/loader/entries/lfs.conf << EOF
title   Linux From Scratch
linux   /vmlinuz-5.12.11-lfs
initrd  /initrd.img-5.12.11
options root="UUID=$ROOT_UUID" rw loglevel=7 earlyprintk=vga,keep rootfstype=$ROOT_FSTYPE
EOF
```

If your root filesystem is on LVM, I have not had luck using a UUID and instead use the device path directly:

```sh
ROOT_FSTYPE=$(blkid -s TYPE -o value /dev/<root device>)

cat > /boot/loader/entries/lfs.conf << EOF
title   Linux From Scratch
linux   /vmlinuz-5.12.11-lfs
initrd  /initrd.img-5.12.11
options root=/dev/mapper/<volGroup>-<volId> rw loglevel=7 earlyprintk=vga,keep rootfstype=$ROOT_FSTYPE
EOF
```

If your root filesystem is on raid0, you will also need to add the kernel option `raid0.default_layout=2`.

Feel free to change the kernel options. Maximum loglevel will generate a lot of logging messages, which is useful when first attempting to boot in case you run into problems, but you may want to quiet it down later. Likewise with directing `printk` during boot to the console and retaining those lines.

## Reboot into LFS

Exit the chroot environment, unmount `/mnt/lfs`, and reboot the system. When the boot menu appears, select `Linux From Scratch`.
