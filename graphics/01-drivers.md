# Drivers

## nvidia drivers

If you have an `nvidia` video card, you will likely want the proprietary drivers from `nvidia`. Find the latest drivers and download them using the NVIDIA search tool at [https://www.nvidia.com/en-us/geforce/drivers/](https://www.nvidia.com/en-us/geforce/drivers/).

As of now, the latest version is `465.31` and that is the link in the snippet.

```sh
curl https://us.download.nvidia.com/XFree86/Linux-x86_64/465.31/NVIDIA-Linux-x86_64-465.31.run -o /sources/NVIDIA-Linux-x86_64-465.31.run &&

sh /sources/NVIDIA-Linux-x86_64-465.31.run --extract-only &&
cd  NVIDIA-Linux-x86_64-465.31                            &&

sudo ./nvidia-installer                      \
        --no-questions                       \
        --no-install-compat32-libs           \
        --log-file-name=${PWD}/installer.log \
        --tmp-dir=${PWD}/tmpbuild            \
        --ui=none                            \
        --no-rpms                            \
        --disable-nouveau                    \
        --run-nvidia-xonfig &&

echo "nvidia-uvm" |
    sudo install -vDm644 /dev/stdin /usr/lib/modules-load.d/nvidia-uvm.conf &&
echo "blacklist nouveau" |
    sudo install -vDm644 /dev/stdin /usr/lib/modprobe.d/nvidia.conf         &&
echo "options nvidia-drm modeset=1" |
    sudo install -vDm644 /dev/stdin /usr/lib/modprobe.d/nvidia-drm.conf     &&

cd .. &&
rm -rf NVIDIA-Linux-x86_64-465
```


```sh
tar xf nvidia-persistenced-init.tar.bz2 &&
cd     kernel                           &&

sed -i "s/__VERSION_STRING/465.31/" dkms.conf              &&
sed -i 's/__JOBS/`nproc`/' dkms.conf                       &&
sed -i 's/__DKMS_MODULES//' dkms.conf                      &&
sed -i '$iBUILT_MODULE_NAME[0]="nvidia"\
DEST_MODULE_LOCATION[0]="/kernel/drivers/video"\
BUILT_MODULE_NAME[1]="nvidia-uvm"\
DEST_MODULE_LOCATION[1]="/kernel/drivers/video"\
BUILT_MODULE_NAME[2]="nvidia-modeset"\
DEST_MODULE_LOCATION[2]="/kernel/drivers/video"\
BUILT_MODULE_NAME[3]="nvidia-drm"\
DEST_MODULE_LOCATION[3]="/kernel/drivers/video"' dkms.conf &&

# Kernel modules
make                      &&
sudo make modules_install &&

cd .. &&

# OpenCL
sudo install -vDm644 nvidia.icd /etc/OpenCL/vendors/                     &&
sudo install -vDm755 libnvidia-compiler.so.465.31 /usr/lib/              &&
sudo install -vDm755 libnvidia-opencl.so.465.31 /usr/lib/                &&

# Firmware
sudo install -vdm644 /usr/lib/firmware/nvidia/465.31/                   &&
sudo install -vDm755 firmware/gsp.bin /usr/lib/firmware/nvidia/465.31/  &&

# nvidia-utils
sudo install -vDm755 nvidia-bug-report.sh /usr/bin/    &&
sudo install -vDm755 nvidia-cuda-mps-control /usr/bin/ &&
sudo install -vDm755 nvidia-cuda-mps-server /usr/bin/  &&
sudo install -vDm755 nvidia-debugdump /usr/bin/        &&
sudo install -vDm755 nvidia-modprobe /usr/bin/         &&
sudo install -vDm755 nvidia-persistenced /usr/bin/     &&
sudo install -vDm755 systemd/nvidia-sleep.sh /usr/bin/ &&
sudo install -vDm755 nvidia-smi /usr/bin/              &&
sudo install -vDm755 nvidia-xconfig /usr/bin/          &&

# nvidia-settings
sudo install -vDm755 nvidia-settings /usr/bin/                        &&
sudo install -vDm755 libnvidia-gtk3.so.465.31 /usr/lib/               &&
sudo install -vDm644 nvidia-settings.desktop /usr/share/applications/ &&
sudo install -vDm644 nvidia-*.1.gz /usr/share/man/man1/               &&
sudo install -vDm644 nvidia-settings.png /usr/share/pixmaps/          &&

# X11
sudo install -vdm644 /usr/lib/xorg/modules/drivers/                     &&
sudo install -vDm755 nvidia_drv.so /usr/lib/xorg/modules/drivers/       &&
sudo install -vdm644 /usr/share/X11/xorg.conf.d/                        &&
sudo install -vDm644 nvidia-drm-outputclass.conf \
    /usr/share/X11/xorg.conf.d/10-nvidia-drm-outputclass.conf           &&
sudo install -vdm644 /usr/lib/nvidia/xorg                               &&
sudo install -vDm755 libglxserver_nvidia.so.465.31 /usr/lib/nvidia/xorg &&

# App profiles
sudo install -vdm644 /usr/share/nvidia                                &&
sudo install -vDm644 nvidia-application-profiles-* /usr/share/nvidia/ &&

# Vulkan
sudo install -vdm644 /usr/share/vulkan/icd.d                               &&
sudo install -vDm644 nvidia_icd.json /usr/share/vulkan/icd.d/              &&
sudo install -vdm644 /usr/share/vulkan/implicit_layer.d                    &&
sudo install -vDm644 nvidia_layers.json /usr/share/vulkan/implicit_layer.d &&

# glvnd
sudo install -vdm644 /usr/share/glvnd/egl_vendor.d                        &&
sudo install -vDm644 10_nvidia.json /usr/share/glvnd/egl_vendor.d         &&
sudo install -vDm644 10_nvidia_wayland.json /usr/share/glvnd/egl_vendor.d &&

# systemd
sudo install -vDm644 systemd/system/* /usr/lib/systemd/system/                  &&
sudo install -vDm755 systemd/system-sleep/nvidia /usr/lib/systemd/system-sleep/ &&
sudo install -vDm644                                                     \
    nvidia-persistenced-init/systemd/nvidia-persistenced.service.template \
    /usr/lib/systemd/system/nvidia-persistenced.service                         &&
sudo sed -i 's/__USER__/nvidia-persistenced/' \
    /usr/lib/systemd/system/nvidia-persistenced.service                         &&
sudo useradd -r -M nvidia-persistenced \
             -s /bin/false -d /run/nvidia-persistenced                          &&

# nvidia libs
sudo install -vDm755 libnvidia-cfg.so.465.31            /usr/lib &&
sudo install -vDm755 libGLESv1_CM_nvidia.so.465.31      /usr/lib &&
sudo install -vDm755 libnvidia-ml.so.465.31             /usr/lib &&
sudo install -vDm755 libnvidia-ptxjitcompiler.so.465.31 /usr/lib &&
sudo install -vDm755 libvdpau_nvidia.so.465.31          /usr/lib &&
sudo install -vDm755 libnvidia-glsi.so.465.31           /usr/lib &&
sudo install -vDm755 libnvidia-glvkspirv.so.465.31      /usr/lib &&
sudo install -vDm755 libnvcuvid.so.465.31               /usr/lib &&
sudo install -vDm755 libnvidia-eglcore.so.465.31        /usr/lib &&
sudo install -vDm755 libnvidia-ifr.so.465.31            /usr/lib &&
sudo install -vDm755 libnvidia-rtcore.so.465.31         /usr/lib &&
sudo install -vDm755 libnvidia-ngx.so.465.31            /usr/lib &&
sudo install -vDm755 libGLESv2_nvidia.so.465.31         /usr/lib &&
sudo install -vDm755 libnvidia-opticalflow.so.465.31    /usr/lib &&
sudo install -vDm755 libnvidia-encode.so.465.31         /usr/lib &&
sudo install -vDm755 libGLX_nvidia.so.465.31            /usr/lib &&
sudo install -vDm755 libcuda.so.465.31                  /usr/lib &&
sudo install -vDm755 libnvoptix.so.465.31               /usr/lib &&
sudo install -vDm755 libnvidia-glcore.so.465.31         /usr/lib &&
sudo install -vDm755 libnvidia-fbc.so.465.31            /usr/lib &&
sudo install -vDm755 libnvidia-gtk2.so.465.31           /usr/lib &&
sudo install -vDm755 libnvidia-allocator.so.465.31      /usr/lib &&
sudo install -vDm755 libnvidia-egl-wayland.so.1.1.5     /usr/lib &&
sudo install -vDm755 libnvidia-cbl.so.465.31            /usr/lib &&
sudo install -vDm755 libEGL_nvidia.so.465.31            /usr/lib &&
sudo install -vDm755 libnvidia-tls.so.465.31            /usr/lib &&
sudo install -vDm755 libEGL.so.465.31                   /usr/lib &&

# other libs
sudo install -vDm755 libEGL.so.1.1.0                    /usr/lib &&
sudo install -vDm755 libGLdispatch.so.0                 /usr/lib &&
sudo install -vDm755 libGLX.so.0                        /usr/lib &&
sudo install -vDm755 libGLESv1_CM.so.1.2.0              /usr/lib &&
sudo install -vDm755 libGL.so.1.7.0                     /usr/lib &&
sudo install -vDm755 libGLESv2.so.2.1.0                 /usr/lib &&
sudo install -vDm755 libOpenCL.so.1.0.0                 /usr/lib &&
sudo install -vDm755 libOpenGL.so.0                     /usr/lib &&

# docs
sudo install -vdm644 /usr/share/doc/nvidia-465.31/html         &&
sudo install -vDm644 html/* /usr/share/doc/nvidia-465.31/html/ &&

sudo ldconfig &&

# module load options
echo "blacklist nouveau" |
    sudo install -Dm644 /dev/stdin /usr/lib/modprobe.d/nvidia.conf     &&
echo "options nvidia-drm modeset=1" |
    sudo install -Dm644 /dev/stdin /usr/lib/modprobe.d/nvidia-drm.conf &&

cd .. &&
rm -rf NVIDIA-Linux-x86_64-465
```

Now is a good time to reboot and ensure the kernel modules are correctly loaded. Rebuild the `initrd` to ensure modules are loaded at the earliest possible time, but this may not be necessary.

`Xorg` should work out of the box, but NVIDIA provides a tool for auto-generating a config anyway:

```sh
sudo nvidia-xconfig
```
