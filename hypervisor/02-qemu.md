# QEMU and libvirt

## QEMU

Add any users you want to manage guests to the `kvm` group:

```sh
sudo usermod -a -G kvm <username>
```

Note that I have chosen to enable support for both `alsa` and `PulseAudio`, contra Beyond Linux From Scratch.

```sh
curl https://download.qemu-project.org/qemu-6.0.0.tar.xz -o /sources/qemu-6.0.0.tar.xz &&

tar xvf /sources/qemu-6.0.0.tar.xz &&
cd       qemu-6.0.0                &&

mkdir -vp build &&
cd        build &&

../configure --prefix=/usr                \
             --sysconfdir=/etc            \
             --target-list=x86_64-softmmu \
             --audio-drv-list=alsa,pa     \
             --docdir=/usr/share/doc/qemu-6.0.0 &&

make              &&
sudo make install &&

cd ../.. &&
rm -rf qemu-6.0.0
```

Add a `udev` rule so the `kvm` device gets the correct permissions:

```sh
sudo su -
cat > /lib/udev/rules.d/65-kvm.rules << "EOF"
KERNEL=="kvm", GROUP="kvm", MODE="0660"
EOF
exit
```

Change the permissions and ownership of a helper script and add a convenience link to the `qemu` executable:

```sh
sudo chgrp kvm  /usr/libexec/qemu-bridge-helper &&
sudo chmod 4750 /usr/libexec/qemu-bridge-helper &&
sudo ln -sv qemu-system-`uname -m` /usr/bin/qemu
```

## libnfnetlink

```sh
curl https://www.netfilter.org/pub/libnfnetlink/libnfnetlink-1.0.1.tar.bz2 -o /sources/libnfnetlink-1.0.1.tar.bz2 &&

tar xvf /sources/libnfnetlink-1.0.1.tar.bz2 &&
cd       libnfnetlink-1.0.1                 &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf libnfnetlink-1.0.1
```

## libmnl

```sh
curl https://www.netfilter.org/pub/libmnl/libmnl-1.0.4.tar.bz2 -o /sources/libmnl-1.0.4.tar.bz2 &&

tar xvf /sources/libmnl-1.0.4.tar.bz2 &&
cd       libmnl-1.0.4                 &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf libmnl-1.0.4
```

## libnetfilter_conntrack

```sh
curl https://www.netfilter.org/pub/libnetfilter_conntrack/libnetfilter_conntrack-1.0.8.tar.bz2 -o /sources/libnetfilter_conntrack-1.0.8.tar.bz2 &&

tar xvf /sources/libnetfilter_conntrack-1.0.8.tar.bz2 &&
cd       libnetfilter_conntrack-1.0.8                 &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf libnetfilter_conntrack-1.0.8
```

## libnftnl

```sh
curl https://www.netfilter.org/pub/libnftnl/libnftnl-1.2.0.tar.bz2 -o /sources/libnftnl-1.2.0.tar.bz2 &&

tar xvf /sources/libnftnl-1.2.0.tar.bz2 &&
cd       libnftnl-1.2.0                 &&

./configure --prefix=/usr &&

make              &&
sudo make install &&

cd .. &&
rm -rf libnftnl-1.2.0
```

## nftables

```sh
curl https://www.netfilter.org/pub/nftables/nftables-0.9.9.tar.bz2 -o /sources/nftables-0.9.9.tar.bz2 &&

tar xvf /sources/nftables-0.9.9.tar.bz2 &&
cd       nftables-0.9.9                 &&

./configure --prefix=/usr    \
            --disable-static \
            --enable-python  \
            --with-json      \
            --docdir=/usr/share/doc/nftables-0.9.9 &&

make              &&
sudo make install &&

cd .. &&
rm -rf nftables-0.9.9
```

## iptables

```sh
curl http://www.netfilter.org/projects/iptables/files/iptables-1.8.7.tar.bz2 -o /sources/iptables-1.8.7.tar.bz2 &&

tar xvf /sources/iptables-1.8.7.tar.bz2 &&
cd       iptables-1.8.7                 &&

./configure --prefix=/usr    \
            --enable-libipq &&

make              &&
sudo make install &&

cd .. &&
rm -rf iptables-1.8.7
```

Additional kernel settings are needed to control netfilter:

```
[*] Networking support  --->                                          [CONFIG_NET]
      Networking Options  --->
        [*] Network packet filtering framework (Netfilter) --->       [CONFIG_NETFILTER]
          [*] Advanced netfilter configuration                        [CONFIG_NETFILTER_ADVANCED]
          Core Netfilter Configuration --->
            <*/M> Netfilter connection tracking support               [CONFIG_NF_CONNTRACK]
            <*/M> Netfilter Xtables support (required for ip_tables)  [CONFIG_NETFILTER_XTABLES]
            <*/M> LOG target support                                  [CONFIG_NETFILTER_XT_TARGET_LOG]
          IP: Netfilter Configuration --->
            <*/M> IP tables support (required for filtering/masq/NAT) [CONFIG_IP_NF_IPTABLES]
```

## dmidecode

Desktop management interface decoder. Read information straight from the hardware.

```sh
curl https://download.savannah.nongnu.org/releases/dmidecode/dmidecode-3.3.tar.xz -o /sources/dmidecode-3.3.tar.xz &&

tar xvf /sources/dmidecode-3.3.tar.xz &&
cd       dmidecode-3.3                &&

sed -i "s:sbin:bin:g" Makefile &&

make prefix=/usr                                                  &&
sudo make prefix=/usr docdir=/usr/share/doc/dmidecode-3.3 install &&

cd .. &&
rm -rf dmidecode-3.3
```

## libvirt

A portable, long-term stable C API for managing virtualization systems on many operating systems, including `QEMU` and `KVM` on Linux.

```sh
curl https://libvirt.org/sources/libvirt-7.4.0.tar.xz -o /sources/libvirt-7.4.0.tar.xz &&

tar xvf /sources/libvirt-7.4.0.tar.xz &&
cd       libvirt-7.4.0                &&

mkdir build &&
cd    build &&

meson --prefix=/usr                         \
      --sysconfdir=/etc                     \
      --docdir=/usr/share/doc/libvirt-7.4.0 \
      --buildtype=release                   \
      --libexecdir=lib/libvirt              \
      -Db_lto=true                          \
      -Drunstatedir=/run                    \
      -Dqemu_group=kvm .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf libvirt-7.4.0
```

To enable the daemon:

```sh
sudo systemctl enable libvirtd
```





























