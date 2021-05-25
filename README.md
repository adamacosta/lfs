# Linux from Scratch

This document pulls a bit from Linux From Scratch and a bit from Beyond Linux From Scratch to create a basic LFS system that can optionally be run as a VM on a Microsoft Hyper-V hypervisor.

## Basic System

1. [Obtaining the sources](base/01-sources.md)
2. [Compiling the cross-toolchain pass 1](base/02-cross-toolchain-pass1.md)
3. [Cross compiling temporary tools](base/03-cross-compile-temp-tools.md)
4. [Compiling the cross-toolchain pass 2](base/04-cross-toolchain-pass2.md)
5. [Prepare chroot environment for temporary system](base/05-temp-chroot.md)
6. [Create system skeleton in chroot](base/06-chroot-skeleton.md)
7. [Install temporary tools](base/07-install-temp-tools.md)
8. [Clean up temporary system](base/08-temp-cleanup.md)
9. [Build the base system](base/09-base-build.md)
10. [Clean up the base system](base/10-base-cleanup.md)
11. [Configure the LFS system](base/11-configure-system.md)
12. [Make system bootable](base/12-boot-system.md)
13. [Post-installation setup](base/13-post-install.md)

## Optional extras

At this point, you have a perfectly functional system you can use. Chances are, you'll want to specialize it for some specific use, but I prefer to have at least a few additional utilities no matter what.

Each optional extra will be downloaded and built in the `/sources` directory. Since we will no longer be building as `root`, installation is done using `sudo`.

Beware that some of the crypto tools take a very long time to build and test.

1. [GnuPG](extras/01-gnupg.md)
2. [Basic system utilities](extras/02-basic-system-utils.md)
3. [Additional system utiltiies](extras/03-addt-system-utils.md)
4. [Network utilities](extras/04-network-utils.md)
5. [Text processing utilities](extras/05-text-utils.md)
6. [Developer tools](extras/06-devtools.md)

## System hardening

1. [Kernel configuration](hardening/01-kernel.md)
2. [SELinux](hardening/02-selinux.md)
3. [iptables](hardening/03-iptables.md)
4. [Setting up a firewall](hardening/04-firewall.md)
5. [Tripwire](hardening/05-tripwire.md)

## Container tools

1. [Foundational libraries and runtimes](containers/01-foundations.md)
2. [Docker](containers/02-docker.md)
3. [Redhat container tools](containers/03-redhat-tools.md)
4. [Serving an image registry](containers/04-registry.md)

## Running a hypervisor

1. [Kernel configuration](hypervisor/01-kernel.md)
2. [libvirt](hypervisor/02-libvirt.md)
3. [virt-manager](hypervisor/03-virt-manager.md)
4. [Terraform libvirt provider](hypervisor/04-terraform.md)
