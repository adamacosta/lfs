# Kernel configuration

First, you need to check if your processor has virtualization extensions enabled:

```sh
egrep '^flags.*(vmx|svm)' /proc/cpuinfo
```

If available, you still may need to go into your motherboard's BIOS to enable it. On my Gigabyte Aorus Extreme TRX40, virtualization extensions are disabled by default. The way to do this will differ by vendor, so please consult your motherboard's user guide.

## Kernel settings

Enable one of these, depending on whether you have an Intel or AMD processor.

```
[*] Virtualization:  --->                                             [CONFIG_VIRTUALIZATION]
  <*/M>   Kernel-based Virtual Machine (KVM) support [CONFIG_KVM]
  <*/M>     KVM for Intel (and compatible) processors support         [CONFIG_KVM_INTEL]
  <*/M>     KVM for AMD processors support                            [CONFIG_KVM_AMD]
```

You will also need to enable ethernet bridging and TUN/TAP device support.

```
[*] Networking support  --->                         [CONFIG_NET]
  Networking options  --->
    <*/M> 802.1d Ethernet Bridging                   [CONFIG_BRIDGE]
Device Drivers  --->
  [*] Network device support  --->                   [CONFIG_NETDEVICES]
    <*/M>    Universal TUN/TAP device driver support [CONFIG_TUN]
```

After the kernel is built with the required drivers, you will still need `bridge-utils`. Ensure you regenerate your `initrd` if you are using one after rebuilding the kernel.

## bridge-utils

Create and manage bridge devices.

```sh
curl https://www.kernel.org/pub/linux/utils/net/bridge-utils/bridge-utils-1.7.1.tar.xz -o /sources/bridge-utils-1.7.1.tar.xz &&

tar xvf /sources/bridge-utils-1.7.1.tar.xz &&
cd       bridge-utils-1.7.1                &&

autoconf                  &&
./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf bridge-utils-1.7.1
```
