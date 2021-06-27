# Libraries

## libimagequant

Adds image quantization features to `PILLOW`.

```sh
wget https://github.com/ImageOptim/libimagequant/archive/refs/tags/2.15.1.zip -P /sources &&

unzip /sources/libimagequant-2.15.1.zip &&
cd     libimagequant-2.15.1             &&

./configure --prefix=/usr \
            --with-openmp=static &&

make                  &&
sudo make install     &&

cd .. &&
rm -rf libimagequant-2.15.1
```

## lz4

Extremely fast compression.

```sh
wget https://github.com/lz4/lz4/archive/refs/tags/v1.9.3.tar.gz -P /sources &&

tar xzvf /sources/lz4-1.9.3.tar.gz &&
cd        lz4-1.9.3                &&

make BUILD_STATIC=no              &&
sudo make install PREFIX=/usr \
                  BUILD_STATIC=no &&

cd .. &&
rm -rf lz4-1.9.3
```

## hdf5

Hierarchical data format.

```sh
wget "https://www.hdfgroup.org/package/hdf5-1-12-0-tar-gz/?wpdmdl=14582&refresh=60ca8587d16d31623885191" -O /sources/hdf5-1.12.0.tar.gz &&

tar xzvf /sources/hdf5-1.12.0.tar.gz    &&
cd        hdf5-1.12.0                   &&

./configure --prefix=/usr                  \
            --disable-static               \
            --enable-optimization=high     \
            --enable-build-mode=production \
            --enable-parallel &&

make              &&
sudo make install &&

cd ../../.. &&
rm -rf hdf5-1.12.0
```

## NVIDIA CUDA toolkit

If you have an `NVIDIA` GPU, this will help you get the most out of your hardware, allowing many of the toolsets introduced here to offload embarrassingly parallel matrix operations to specialized cores on the GPU.

Unfortunately, `NVIDIA` does not provide any generic distribution for this, only packages for "supported platforms," so we'll instead download the `.deb` file intended for Ubuntu 20.04 and extract the files from it. Beware this is a pretty large (3.1 GB) download.

### The `.deb` package format

#### Top-level archive

`.deb` is a very simple file format, consisting of two tar archives, optionally compressed, inside of an ar archive. The top-level contents are always this:

```
<package-archive>
|-- debian-binary
|-- control.tar
`-- data.tar
```

Optionally, there may be other members, but they can safely be ignored and should be prefixed with an underscore `_`. 

These can be extracted using the Unix `ar` tool using the command `ar x [--output <directory>] <archive>` or listed using `ar t <archive>`.

#### Package format version

`debian-binary` is a plain-text, single-line file containing the version of the package format, presently `2.0`.

#### Control archive

`control.tar` may be optionally compressed using either `gzip` or `xz` and contains:

- `control`: description of the package and its dependencies
- `md5sums`: MD5 checksums of all files in the package

Optionally, it may also contain:

- `conffiles`: files that should not be overwritten during a package upgrade
- `preinst`, `postinst`, `prerm`, `postrm`: scripts to run before and after installation and removal
- `config`: script supporting the `debconf` configuration system
- `shlibs`: list of shared library dependencies
- `triggers`: notification events to send to other applications that should take an action upon installation of this package
- `symbols`: a table of symbols exported by library packages

#### Data archive

`data.tar` may be optionally compressed with `gzip`, `bzip2`, `lzma`, or `xz` and contains all of the installable files. If `md5sums` is present in the `control.tar` archive, then each file in the `data.tar` archive should match an entry.

#### Meta-packages

Packages may be meta-packages, which install nothing but simply depend on other packages in order to provide a simple way to specify a group of packages that should be installed together. If a package is a meta-package, then the data archive may be empty or contain only the empty directory structure that the dependent packages will install into.

#### Extracting files to install

`ar x <package>.deb` will extract the top-level contents into the current working directory.

`ar x --output <dir> <package>.deb` will extract into `<dir>`. 

`tar xfO control.tar(|.[gx]z) ./control` will print the contents of the `control` file from the archive onto standard output. Beware this is the capital letter O and *not* the number 0.

`tar tf data.tar(|.(bz2|lzma|[xg]z))` will list the members of the `data` archive.

`tar xf data.tar(|.(bz2|lzma|[xg]z))` will extract the members of the `data` archive to the current working directory.

`tar xf data.tar(|.(bz2|lzma|[xg]z)) -C <dir>` will extract the members of the `data` archive into `<dir>`, the equivalent of a `DESTDIR` installation.

`sudo tar xf data.tar(|.(bz2|lzma|[xg]z)) -C /` will extract the members of the `data` archive to the system root, effectively installing the package.

`sudo tar xkf data.tar(|.(bz2|lzma|[xg]z)) -C /` is the "no clobber" version of the installation, telling `tar` not to replace already existing files when extracting.

### CUDA Toolkit Installation

```sh
wget https://developer.download.nvidia.com/compute/cuda/11.3.1/local_installers/cuda-repo-ubuntu2004-11-3-local_11.3.1-465.19.01-1_amd64.deb -P /sources &&

mkdir cuda-11.3.1-465.19.01 &&
ar x --output cuda-11.3.1-465.19.01/ /sources/cuda-repo-ubuntu2004-11-3-local_11.3.1-465.19.01-1_amd64.deb &&

cd     cuda-11.3.1-465.19.01 &&
tar xf data.tar.xz
```

This creates a directory full of the `.deb` files for each individual component of the `CUDA` toolkit. This is the full list of packages:

```
xserver-xorg-video-nvidia-465_465.19.01-0ubuntu1_amd64.deb
xserver-xorg-video-nvidia-430_465.19.01-0ubuntu1_amd64.deb
nvidia-utils-465_465.19.01-0ubuntu1_amd64.deb
nvidia-utils-430_465.19.01-0ubuntu1_amd64.deb
nvidia-settings_465.19.01-0ubuntu1_amd64.deb
nvidia-modprobe_465.19.01-0ubuntu1_amd64.deb
nvidia-kernel-source-465_465.19.01-0ubuntu1_amd64.deb
nvidia-kernel-source-430_465.19.01-0ubuntu1_amd64.deb
nvidia-kernel-common-465_465.19.01-0ubuntu1_amd64.deb
nvidia-kernel-common-430_465.19.01-0ubuntu1_amd64.deb
nvidia-headless-no-dkms-465_465.19.01-0ubuntu1_amd64.deb
nvidia-headless-no-dkms-430_465.19.01-0ubuntu1_amd64.deb
nvidia-headless-465_465.19.01-0ubuntu1_amd64.deb
nvidia-headless-430_465.19.01-0ubuntu1_amd64.deb
nvidia-fabricmanager-dev-465_465.19.01-1_amd64.deb
nvidia-fabricmanager-465_465.19.01-1_amd64.deb
nvidia-driver-465_465.19.01-0ubuntu1_amd64.deb
nvidia-driver-430_465.19.01-0ubuntu1_amd64.deb
nvidia-dkms-465_465.19.01-0ubuntu1_amd64.deb
nvidia-dkms-430_465.19.01-0ubuntu1_amd64.deb
nvidia-compute-utils-465_465.19.01-0ubuntu1_amd64.deb
nvidia-compute-utils-430_465.19.01-0ubuntu1_amd64.deb
nsight-systems-2021.1.3_2021.1.3.14-1_amd64.deb
nsight-compute-2021.1.1_2021.1.1.5-1_amd64.deb
libxnvctrl0_465.19.01-0ubuntu1_amd64.deb
libxnvctrl-dev_465.19.01-0ubuntu1_amd64.deb
libnvjpeg-dev-11-3_11.5.0.109-1_amd64.deb
libnvjpeg-11-3_11.5.0.109-1_amd64.deb
libnvidia-nscq-465_465.19.01-1_amd64.deb
libnvidia-ifr1-465_465.19.01-0ubuntu1_amd64.deb
libnvidia-ifr1-430_465.19.01-0ubuntu1_amd64.deb
libnvidia-gl-465_465.19.01-0ubuntu1_amd64.deb
libnvidia-gl-430_465.19.01-0ubuntu1_amd64.deb
libnvidia-fbc1-465_465.19.01-0ubuntu1_amd64.deb
libnvidia-fbc1-430_465.19.01-0ubuntu1_amd64.deb
libnvidia-extra-465_465.19.01-0ubuntu1_amd64.deb
libnvidia-encode-465_465.19.01-0ubuntu1_amd64.deb
libnvidia-encode-430_465.19.01-0ubuntu1_amd64.deb
libnvidia-decode-465_465.19.01-0ubuntu1_amd64.deb
libnvidia-decode-430_465.19.01-0ubuntu1_amd64.deb
libnvidia-compute-465_465.19.01-0ubuntu1_amd64.deb
libnvidia-compute-430_465.19.01-0ubuntu1_amd64.deb
libnvidia-common-465_465.19.01-0ubuntu1_all.deb
libnvidia-common-430_465.19.01-0ubuntu1_all.deb
libnvidia-cfg1-465_465.19.01-0ubuntu1_amd64.deb
libnvidia-cfg1-430_465.19.01-0ubuntu1_amd64.deb
libnpp-dev-11-3_11.3.3.95-1_amd64.deb
libnpp-11-3_11.3.3.95-1_amd64.deb
libcusparse-dev-11-3_11.6.0.109-1_amd64.deb
libcusparse-11-3_11.6.0.109-1_amd64.deb
libcusolver-dev-11-3_11.1.2.109-1_amd64.deb
libcusolver-11-3_11.1.2.109-1_amd64.deb
libcurand-dev-11-3_10.2.4.109-1_amd64.deb
libcurand-11-3_10.2.4.109-1_amd64.deb
libcufft-dev-11-3_10.4.2.109-1_amd64.deb
libcufft-11-3_10.4.2.109-1_amd64.deb
libcublas-dev-11-3_11.5.1.109-1_amd64.deb
libcublas-11-3_11.5.1.109-1_amd64.deb
cuda_11.3.1-1_amd64.deb
cuda-visual-tools-11-3_11.3.1-1_amd64.deb
cuda-ubuntu2004.pin
cuda-tools-11-3_11.3.1-1_amd64.deb
cuda-toolkit-config-common_11.3.109-1_all.deb
cuda-toolkit-11-config-common_11.3.109-1_all.deb
cuda-toolkit-11-3_11.3.1-1_amd64.deb
cuda-toolkit-11-3-config-common_11.3.109-1_all.deb
cuda-thrust-11-3_11.3.109-1_amd64.deb
cuda-sanitizer-11-3_11.3.111-1_amd64.deb
cuda-samples-11-3_11.3.58-1_amd64.deb
cuda-runtime-11-3_11.3.1-1_amd64.deb
cuda-nvvp-11-3_11.3.111-1_amd64.deb
cuda-nvtx-11-3_11.3.109-1_amd64.deb
cuda-nvrtc-dev-11-3_11.3.109-1_amd64.deb
cuda-nvrtc-11-3_11.3.109-1_amd64.deb
cuda-nvprune-11-3_11.3.58-1_amd64.deb
cuda-nvprof-11-3_11.3.111-1_amd64.deb
cuda-nvml-dev-11-3_11.3.58-1_amd64.deb
cuda-nvdisasm-11-3_11.3.58-1_amd64.deb
cuda-nvcc-11-3_11.3.109-1_amd64.deb
cuda-nsight-systems-11-3_11.3.1-1_amd64.deb
cuda-nsight-compute-11-3_11.3.1-1_amd64.deb
cuda-nsight-11-3_11.3.109-1_amd64.deb
cuda-minimal-build-11-3_11.3.1-1_amd64.deb
cuda-memcheck-11-3_11.3.109-1_amd64.deb
cuda-libraries-dev-11-3_11.3.1-1_amd64.deb
cuda-libraries-11-3_11.3.1-1_amd64.deb
cuda-gdb-src-11-3_11.3.109-1_amd64.deb
cuda-gdb-11-3_11.3.109-1_amd64.deb
cuda-drivers_465.19.01-1_amd64.deb
cuda-drivers-fabricmanager_465.19.01-1_amd64.deb
cuda-drivers-fabricmanager-465_465.19.01-1_amd64.deb
cuda-drivers-465_465.19.01-1_amd64.deb
cuda-driver-dev-11-3_11.3.109-1_amd64.deb
cuda-documentation-11-3_11.3.111-1_amd64.deb
cuda-demo-suite-11-3_11.3.58-1_amd64.deb
cuda-cuxxfilt-11-3_11.3.58-1_amd64.deb
cuda-cupti-dev-11-3_11.3.111-1_amd64.deb
cuda-cupti-11-3_11.3.111-1_amd64.deb
cuda-cuobjdump-11-3_11.3.58-1_amd64.deb
cuda-cudart-dev-11-3_11.3.109-1_amd64.deb
cuda-cudart-11-3_11.3.109-1_amd64.deb
cuda-compiler-11-3_11.3.1-1_amd64.deb
cuda-compat-11-3_465.19.01-1_amd64.deb
cuda-command-line-tools-11-3_11.3.1-1_amd64.deb
cuda-11-3_11.3.1-1_amd64.deb
```

This is quite a bit and includes the GPU drivers, which you'll have already installed. You can consult documentation for whatever you're trying to build, but at minimum you'll likely want the `cuda-toolkit` and `cuda-runtime`, which are each meta-packages that contain no data but simply define dependencies. To save you the trouble, I have extracted the package lists from the `control` file of each `.deb`. `cuda-runtime` consists of:

```
cuda-libraries
cuda-drivers
```

`cuda-toolkit` consists of:

```
cuda-compiler
cuda-libraries
cuda-libraries-dev
cuda-tools
cuda-documentation
cuda-nvml-dev
cuda-samples
```

You'll notice both contains `cuda-libraries`, so in total we'll want:

```
cuda-drivers
cuda-compiler
cuda-libraries
cuda-libraries-dev
cuda-tools
cuda-documentation
cuda-nvml-dev
cuda-samples
```

To extract what we need:

```sh
INSTALL=${PWD}/install
EXTRACT=${PWD}/extracted

[ -d $INSTALL ] || mkdir $INSTALL
[ -d $EXTRACT ] || mkdir $EXTRACT

RM=()
KEEP=()

for PKG in $(ls var/cuda-repo-ubuntu2004-11-3-local)
do
    grep '.deb' <<< "$PKG" >/dev/null || continue

    SHORT=$(sed 's/_\(amd64\|all\).deb//' <<< "$PKG")    
    TAR_NAME=$(ls "$EXTRACT/$SHORT/control.tar"* 2>/dev/null)
    [ -e "$TAR_NAME" ] || continue

    CONTENTS=$(tar xfO "$TAR_NAME" ./control)
    DESC=$(grep 'Description' <<< "$CONTENTS")
    echo "$DESC"
    if [ $(grep -i 'meta-package' <<< "$DESC") ] ; then
        RM+=($SHORT)
    else
        KEEP+=($SHORT)
    fi
done

for RMDIR in $RM
do
    rm -rf $RMDIR
done

for KEEPDIR in $KEEP
do
    tar xvf "$EXTRACT/$KEEPDIR/data.tar"* -C $INSTALL
done
```


    mkdir $SHORT

    ar x --output $EXTRACT/$SHORT/ var/cuda-repo-ubuntu2004-11-3-local/$PKG








