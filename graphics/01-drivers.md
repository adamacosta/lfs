# Drivers

## nvidia drivers

If you have an NVIDIA video card, you will likely want the proprietary drivers from NVIDIA. Find the latest drivers and download them using the NVIDIA search tool at [https://www.nvidia.com/en-us/geforce/drivers/](https://www.nvidia.com/en-us/geforce/drivers/).

As of now, the latest version is `465.31` and that is the link in the snippet.

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
