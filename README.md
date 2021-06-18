# Linux from Scratch

This document pulls a bit from [Linux From Scratch systemd](https://www.linuxfromscratch.org/lfs/view/systemd/index.html) and a bit from [Beyond Linux From Scratch systemd](https://www.linuxfromscratch.org/blfs/view/systemd/index.html) to create a fully-customized Linux distro.

Several libraries, tools, and features are also introduced that are not in Linux From Scratch or Beyond Linux From Scratch, so you can think of this as one of many projects out there attempting "Beyond Beyond Linux From Scratch." This includes additional libraries and tools for numerical and scientific computing, machine learning, running containers, and building out Kubernetes clusters on customized distros. There are also instructions here for creating Linux systems that will run as a guest on a hypervisor, in particular Microsoft's `Hyper-V`, which I could not find much documentation on.

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

Beware that some of the crypto tools and compilers take a very long time to build and test. If you choose to attempt building `ATLAS`, it can potentially take days and may fail even after running for a very long time.

### Extending the toolchain

These are not ordered. Libraries are presented first because many tools in later sections depend on them. However, at least a few libraries depend upon build tools and compilers not included with basic Linux From Scratch. In particular, you may need to build `CMake`, `LLVM`, and `rustc` before building many libraries. Many libraries and tools also use `doxygen` to generate API documentation, which you would need to build first. `GraphViz` and `FreeType` have optional dependencies on tools that also depend on them, so you may find yourself building each multiple times to incorporate optional features when they become available.

1. [General purpose libraries](extras/01-libraries.md)
2. [Basic system utilities](extras/02-basic-system-utils.md)
3. [GnuPG](extras/03-gnupg.md)
4. [Network utilities](extras/04-network-utils.md)
5. [Additional system utiltiies](extras/05-addt-system-utils.md)
6. [Text processing utilities](extras/06-text-utils.md)
7. [Multimedia tools](extras/07-media-utils.md)
8. [Documentation generators](extras/08-doc-utils.md)
9. [Developer tools](extras/09-devtools.md)
10. [Numerical and geospatial libraries](extras/10-numerical.md)
11. [System configuration](extras/11-sysconfig.md)

### Graphical desktop

This focuses on building the `GNOME` desktop, which is enormous and takes a very long time, but I just like the way it looks and have gotten used to it. Support for using `NVIDIA` proprietary drivers and libraries is also included here.

1. [Drivers](graphics/01-drivers.md)
2. [XOrg](graphics/02-xorg.md)
3. [Audio and video libraries](graphics/03-audio-video.md)
4. [GNOME desktop](graphics/04-gnome.md)
5. [Desktop applications](graphics/05-applications.md)

### Scientific and Statistical Computing

1. [Additional native libraries](scicomp/01-libraries.md)
2. [Python scientific computing stack](scicomp/02-python.md)
3. [Alternative scientific and statistical toolchains](scicomp/03-alternatives.md)

### Running a hypervisor

1. [Kernel configuration](hypervisor/01-kernel.md)
2. [QEMU and libvirt](hypervisor/02-qemu.md)
3. [Graphical guest managers](hypervisor/03-graphical-mgr.md)
4. [Terraform libvirt provider](hypervisor/04-terraform.md)

### Container tools

1. [Foundational libraries and runtimes](containers/01-foundations.md)
2. [Docker](containers/02-docker.md)
3. [Redhat container tools](containers/03-redhat-tools.md)
4. [Serving an image registry](containers/04-registry.md)

### System hardening

1. [Kernel configuration](hardening/01-kernel.md)
2. [SELinux](hardening/02-selinux.md)
3. [iptables](hardening/03-iptables.md)
4. [Setting up a firewall](hardening/04-firewall.md)
5. [Tripwire](hardening/05-tripwire.md)
