# Linux from Scratch

This document pulls a bit from [Linux From Scratch 10.1-systemd](https://www.linuxfromscratch.org/lfs/view/stable-systemd/index.html) and a bit from [Beyond Linux From Scratch 10.1-systemd](https://www.linuxfromscratch.org/blfs/view/10.1-systemd/index.html) to create a Linux system that can optionally be run as a VM on a Microsoft `Hyper-V` hypervisor.

## Basic System

The first part follows Linux From Scratch pretty closely, creating a base headless system, adding in support for `ssh`, `sudo`, and `PAM`, a non-root user, changing the default shell to `zsh`, and optionally adding support for running under `Hyper-V`.

1. [Getting started](base/01-get-started.md)
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

At this point, you have a perfectly functional system you can use. Chances are, you'll want to specialize it for some specific use. Many options are provided for what can be done.

Each optional extra will be downloaded into the `/sources` directory and built in `/sources/build_dir`. Since we will no longer be building as `root`, installation is done using `sudo`.

Beware that some of the crypto tools and compilers take a very long time to build and test.

### Extending the toolchain

1. [GnuPG](extras/01-gnupg.md)
2. [Basic system utilities](extras/02-basic-system-utils.md)
3. [Additional system utiltiies](extras/03-addt-system-utils.md)
4. [Network utilities](extras/04-network-utils.md)
5. [General purpose libraries](extras/05-gen-libs.md)
6. [Text processing utilities](extras/06-text-utils.md)
7. [Multimedia and documentation generation](extras/07-media-doc-utils.md)
8. [Developer tools](extras/08-devtools.md)
9. [Numerical and geospatial libraries](09-numerical.md)
10. [System configuration](extras/10-sysconfig.md)

### Graphical desktop

1. [Drivers](graphics/01-drivers.md)
2. [XOrg](graphics/02-xorg.md)
3. [Audio and video libraries](graphics/03-audio-video.md)
4. [GNOME desktop](graphica/04-gnome.md)

### System hardening

1. [Kernel configuration](hardening/01-kernel.md)
2. [SELinux](hardening/02-selinux.md)
3. [iptables](hardening/03-iptables.md)
4. [Setting up a firewall](hardening/04-firewall.md)
5. [Tripwire](hardening/05-tripwire.md)

### Container tools

1. [Foundational libraries and runtimes](containers/01-foundations.md)
2. [Docker](containers/02-docker.md)
3. [Redhat container tools](containers/03-redhat-tools.md)
4. [Serving an image registry](containers/04-registry.md)

### Running a hypervisor

1. [Kernel configuration](hypervisor/01-kernel.md)
2. [libvirt](hypervisor/02-libvirt.md)
3. [virt-manager](hypervisor/03-virt-manager.md)
4. [Terraform libvirt provider](hypervisor/04-terraform.md)
