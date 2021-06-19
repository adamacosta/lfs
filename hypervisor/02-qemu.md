# QEMU and libvirt

## QEMU

Add any users you want to manage guests to the `kvm` group:

```sh
sudo usermod -a -G kvm <username>
```

Note that I have chosen to enable support for both `alsa` and `PulseAudio`, contra Beyond Linux From Scratch.

```sh
wget https://download.qemu-project.org/qemu-6.0.0.tar.xz -P /sources/qemu-6.0.0.tar.xz &&

tar xvf /sources/qemu-6.0.0.tar.xz &&
cd       qemu-6.0.0                &&

mkdir build &&
cd    build &&

../configure --prefix=/usr                \
             --sysconfdir=/etc            \
             --target-list=x86_64-softmmu \
             --audio-drv-list=alsa,pa     \
             --enable-lto                 \
             --docdir=/usr/share/doc/qemu-6.0.0 &&

make              &&
ninja test        &&
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

Provide the `kvm` convenenience script that provides a shortcut to launch `kvm`-accelerated guests with `QEMU`.

```sh
sudo su -
cat > /usr/bin/kvm <<"EOF" &&
#!/bin/sh
exec qemu-system-x86_64 -enable-kvm "$@"
EOF
chmod 0755 /usr/bin/kvm
exit
```

## libnfnetlink

```sh
wget https://www.netfilter.org/pub/libnfnetlink/libnfnetlink-1.0.1.tar.bz2 -P /sources/libnfnetlink-1.0.1.tar.bz2 &&

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
wget https://www.netfilter.org/pub/libmnl/libmnl-1.0.4.tar.bz2 -P /sources/libmnl-1.0.4.tar.bz2 &&

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
wget https://www.netfilter.org/pub/libnetfilter_conntrack/libnetfilter_conntrack-1.0.8.tar.bz2 -P /sources/libnetfilter_conntrack-1.0.8.tar.bz2 &&

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
wget https://www.netfilter.org/pub/libnftnl/libnftnl-1.2.0.tar.bz2 -P /sources/libnftnl-1.2.0.tar.bz2 &&

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
wget https://www.netfilter.org/pub/nftables/nftables-0.9.9.tar.bz2 -P /sources/nftables-0.9.9.tar.bz2 &&

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
wget http://www.netfilter.org/projects/iptables/files/iptables-1.8.7.tar.bz2 -P /sources/iptables-1.8.7.tar.bz2 &&

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
wget https://download.savannah.nongnu.org/releases/dmidecode/dmidecode-3.3.tar.xz -P /sources/dmidecode-3.3.tar.xz &&

tar xvf /sources/dmidecode-3.3.tar.xz &&
cd       dmidecode-3.3                &&

sed -i "s:sbin:bin:g" Makefile &&

make prefix=/usr                                                  &&
sudo make prefix=/usr docdir=/usr/share/doc/dmidecode-3.3 install &&

cd .. &&
rm -rf dmidecode-3.3
```

## yajl2

"Yet another JSON library." Required by `libvirt` in order to support `QEMU` as a driver.

```sh
wget https://github.com/lloyd/yajl/archive/2.1.0.tar.gz -P /sources/yajl-2.1.0.tar.gz &&

tar xzvf /sources/yajl-2.1.0.tar.gz &&
cd        yajl-2.1.0                &&

mkdir build &&
cd    build &&

cmake -DCMAKE_INSTALL_PREFIX=/usr \
      -DCMAKE_BUILD_TYPE=Release .. &&

make              &&
sudo make install &&

sudo rm /usr/lib/libyajl_s.a &&

cd ../.. &&
rm -rf yajl-2.1.0
```

## libvirt

A portable, long-term stable C API for managing virtualization systems on many operating systems, including `QEMU` and `KVM` on Linux.

Create a user to own the socket:

```sh
sudo groupadd libvirt
```

```sh
wget https://libvirt.org/sources/libvirt-7.4.0.tar.xz -P /sources/libvirt-7.4.0.tar.xz &&

tar xvf /sources/libvirt-7.4.0.tar.xz &&
cd       libvirt-7.4.0                &&

mkdir build &&
cd    build &&

meson --prefix=/usr            \
      --sysconfdir=/etc        \
      --buildtype=release      \
      --libexecdir=lib/libvirt \
      -Db_lto=true             \
      -Drunstatedir=/run       \
      -Ddriver_qemu=enabled    \
      -Dqemu_group=kvm .. &&

ninja              &&
ninja test         &&
sudo ninja install &&

sudo mv /usr/share/doc/libvirt /usr/share/doc/libvirt-7.4.0 &&

cd ../.. &&
rm -rf libvirt-7.4.0
```

Fix some permissions and create a default configuration:

```sh
sudo su -
echo "z /var/lib/libvirt/qemu 0751" > /usr/lib/tmpfiles.d/libvirt.conf
chmod 600 /etc/libvirt/nwfilter/*.xml \
    /etc/libvirt/qemu/networks/default.xml
cat >> /etc/libvirt/libvirtd.conf <<"EOF"
uri_default = "qemu:///system"
unix_sock_group = "libvirt"
unix_sock_ro_perms = "0777" # set to 0770 to deny non-group libvirt users
unix_sock_rw_perms = "0770"
auth_unix_ro = "none"
auth_unix_rw = "none"
EOF
exit
```

Add any users you want to manage the socket to the new group:

```sh
usermod -a -G libvirt <user>
```

To enable the libvirt daemon and logging daemon:

```sh
sudo systemctl enable libvirtd &&
sudo systemctl enable virtlogd
```
