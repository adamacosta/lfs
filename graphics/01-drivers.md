# Drivers

## nvidia drivers

If you have an NVIDIA video card, you will likely want the proprietary drivers from NVIDIA. Find the latest drivers and download them using the NVIDIA search tool at [https://www.nvidia.com/en-us/geforce/drivers/](https://www.nvidia.com/en-us/geforce/drivers/).

*Note on upgrading:
The NVIDIA installer builds against the current running kernel by default. When you upgrade the kernel and need to rebuild NVIDIA modules, do so by passing the `--kernel-name=X.Y.Z` option, where `X.Y.Z` is the new kernel version you just built, and `--kernel-module-only` to build the modules but not unload the currently running module. That way you do not need to double reboot to pick up both the new kernel and rebuilt NVIDIA modules and it will not kill your video feed unloading the currently active modules.*

```sh
wget https://us.download.nvidia.com/XFree86/Linux-x86_64/465.31/NVIDIA-Linux-x86_64-465.31.run -P /sources &&

sh /sources/NVIDIA-Linux-x86_64-465.31.run --extract-only &&
cd  NVIDIA-Linux-x86_64-465.31                            &&

mkdir -v tmpbuild &&

sudo ./nvidia-installer                      \
        --no-questions                       \
        --no-install-compat32-libs           \
        --install-libglvnd                   \
        --log-file-name=${PWD}/installer.log \
        --tmpdir=${PWD}/tmpbuild             \
        --ui=none                            \
        --no-rpms                            \
        --disable-nouveau &&

echo "nvidia-uvm" |
    sudo install -vDm644 /dev/stdin /usr/lib/modules-load.d/nvidia-uvm.conf &&
echo "blacklist nouveau" |
    sudo install -vDm644 /dev/stdin /usr/lib/modprobe.d/nvidia.conf         &&
echo "options nvidia-drm modeset=1" |
    sudo install -vDm644 /dev/stdin /usr/lib/modprobe.d/nvidia-drm.conf     &&

cd .. &&
rm -rf NVIDIA-Linux-x86_64-465
```

Now is a good time to reboot and ensure the kernel modules are correctly loaded. Rebuild the `initrd` to ensure modules are loaded at the earliest possible time, but this may not be necessary.
