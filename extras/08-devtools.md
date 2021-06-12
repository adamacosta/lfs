# Developer tools

## Source control

### git

```sh
curl https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.31.1.tar.xz -o /sources/git-2.31.1.tar.xz &&
curl https://mirrors.edge.kernel.org/pub/software/scm/git/git-manpages-2.31.0.tar.xz -o /sources/git-manpages-2.31.1.tar.xz &&

tar xvf /sources/git-2.31.1.tar.xz &&
cd       git-2.31.1                &&

./configure --prefix=/usr                   \
            --with-gitconfig=/etc/gitconfig &&

make                                                       &&
sudo make perllibdir=/usr/lib/perl5/5.32/site_perl install &&
sudo tar -xf /sources/git-manpages-2.31.1.tar.xz \
    -C /usr/share/man --no-same-owner --no-overwrite-dir   &&

cd .. &&
rm -rf git-2.31.1
```

## Debuggers and analyzers

### GDB

This requires `doxygen` to build the documentation. If you choose not to install that, remove the lines:

```sh
make -C gdb/doc doxy
sudo install -d /usr/share/doc/gdb-10.1
sudo rm -rf gdb/doc/doxy/xml
sudo cp -Rv gdb/doc/doxy /usr/share/doc/gdb-10.1
```

```sh
curl https://ftp.gnu.org/gnu/gdb/gdb-10.1.tar.xz -o /sources/gdb-10.1.tar.xz &&

tar xvf /sources/gdb-10.1.tar.xz &&
cd       gdb-10.1                &&

mkdir build &&
cd    build &&

../configure --prefix=/usr          \
             --with-system-readline \
             --with-python=/usr/bin/python3 &&

make                 &&
make -C gdb/doc doxy &&

pushd gdb/testsuite                          &&
make  site.exp                               &&
echo  "set gdb_test_timeout 120" >> site.exp &&
runtest                                      &&
popd                                         &&

sudo make -C gdb install                     &&

sudo install -d /usr/share/doc/gdb-10.1          &&
sudo rm -rf gdb/doc/doxy/xml                     &&
sudo cp -Rv gdb/doc/doxy /usr/share/doc/gdb-10.1 &&

cd ../.. &&
rm -rf gdb-10.1
```

### valgrind

```sh
curl https://sourceware.org/ftp/valgrind/valgrind-3.17.0.tar.bz2 -o /sources/valgrind-3.17.0.tar.bz2 &&

tar xvf /sources/valgrind-3.17.0.tar.bz2 &&
cd       valgrind-3.17.0                 &&

sed -i 's|/doc/valgrind||' docs/Makefile.in &&

./configure --prefix=/usr \
            --datadir=/usr/share/doc/valgrind-3.17.0 &&

make              &&
sudo make install &&

cd .. &&
rm -rf valgrind-3.17.0
```

## Build tools

### CMake

Note that the BLFS lists `libarchive` as optional, but unless you edit the input files to the `bootstrap` script, it will fail, saying it is required. To install it:

```sh
curl https://github.com/libarchive/libarchive/releases/download/3.5.1/libarchive-3.5.1.tar.xz -o /sources/libarchive-3.5.1.tar.xz &&

tar xvf /sources/libarchive-3.5.1.tar.xz &&
cd       libarchive-3.5.1                &&

./configure --prefix=/usr \
            --disable-static &&

make              &&
sudo make install &&

cd .. &&
rm -rf libarchive-3.5.1
```

We will run the build in a `Python` virtualenv to enable using `Sphinx` to build the html documentation.

```sh
curl https://cmake.org/files/v3.20/cmake-3.20.3.tar.gz -o /sources/cmake-3.20.3.tar.gz &&

tar xzvf /sources/cmake-3.20.3.tar.gz &&
cd        cmake-3.20.3                &&

python -m venv buildenv   &&
. ./buildenv/bin/activate &&
pip install sphinx        &&

sed -i '/"lib64"/s/64//' Modules/GNUInstallDirs.cmake &&

./bootstrap --prefix=/usr        \
            --system-libs        \
            --mandir=/share/man  \
            --no-system-jsoncpp  \
            --no-system-librhash \
            --docdir=/usr/share/doc/cmake-3.20.3 &&

make              &&
sudo make install &&

deactivate &&

cd .. &&
rm -rf cmake-3.20.3
```

## Assemblers

### nasm

The Netwide Assembler. Required by `libass`.

```sh
curl http://www.nasm.us/pub/nasm/releasebuilds/2.15.05/nasm-2.15.05.tar.xz -o /sources/nasm-2.15.05.tar.xz &&
curl http://www.nasm.us/pub/nasm/releasebuilds/2.15.05/nasm-2.15.05-xdoc.tar.xz -o /sources/nasm-2.15.05-xdoc.tar.xz &&

tar xvf /sources/nasm-2.15.05.tar.xz                           &&
tar xvf /sources/nasm-2.15.05-xdoc.tar.xz --strip-components=1 &&
cd       nasm-2.15.05                                          &&

./configure --prefix=/usr &&

make                                                   &&
sudo make install                                      &&
sudo install -m755 -d     /usr/share/doc/nasm-2.15.05/ &&
sudo cp -v doc/*.{txt,ps} /usr/share/doc/nasm-2.15.05  &&

cd .. &&
rm -rf nasm-2.15.05
```

### yasm

```sh
curl http://www.tortall.net/projects/yasm/releases/yasm-1.3.0.tar.gz -o /sources/yasm-1.3.0.tar.gz &&

tar xzvf /sources/yasm-1.3.0.tar.gz &&
cd        yasm-1.3.0                &&

sed -i 's#) ytasm.*#)#' Makefile.in &&
./configure --prefix=/usr           &&

make              &&
sudo make install &&

cd .. &&
rm -rf yasm-1.3.0
```

## Programming languages

### golang

`go` needs to be bootstrapped from `go-1.4`, the last version that was still written in C, so we first need to download and build that.

```sh
curl https://dl.google.com/go/go1.4-bootstrap-20171003.tar.gz -o /sources/go1.4-bootstrap-20171003.tar.gz &&

tar xzvf /sources/go1.4-bootstrap-20171003.tar.gz &&
cd        go/src                                  &&

CGO_ENABLED=0 ./make.bash &&

cd ../..
```

Now we can build the latest stable `go` using `go-1.4`:

```sh
curl https://go.googlesource.com/go/+archive/refs/tags/go1.16.4.tar.gz -o /sources/go1.16.4.tar.gz &&

mkdir -v go1.16.4                             &&
tar xzvf /sources/go1.16.4.tar.gz -C go1.16.4 &&
cd go1.16.4/src                               &&

GOROOT_FINAL=/usr/lib/go               \
GOROOT_BOOTSTRAP=/sources/build_dir/go \
GOPATH=/sources/build_dir/go1.16.4     \
./make.bash &&

PATH="/sources/build_dir/go1.16.4/bin:$PATH" go install -v -race std             &&
PATH="/sources/build_dir/go1.16.4/bin:$PATH" go install -v -buildmode=shared std &&

sudo install -vdm755 /usr/lib/go                     &&
sudo install -vdm755 /usr/share/doc/go-1.16.4        &&
cd ..                                                &&
sudo cp -a bin pkg src lib misc api test /usr/lib/go &&
sudo cp -r doc/* /usr/share/doc/go-1.16.4            &&
sudo ln -svf /usr/lib/go/bin/go /usr/bin/go          &&
sudo ln -svf /usr/lib/go/bin/gofmt /usr/bin/gofmt
```

Now delete both source trees:

```sh
cd .. &&
sudo rm -rf go*
```

### LLVM and Clang

```sh
curl https://github.com/llvm/llvm-project/releases/download/llvmorg-12.0.0/llvm-12.0.0.src.tar.xz -o /sources/llvm-12.0.0.src.tar.xz &&
curl https://github.com/llvm/llvm-project/releases/download/llvmorg-12.0.0/clang-12.0.0.src.tar.xz -o /sources/clang-12.0.0.src.tar.xz &&
curl https://github.com/llvm/llvm-project/releases/download/llvmorg-12.0.0/compiler-rt-12.0.0.src.tar.xz -o /sources/compiler-rt-12.0.0.src.tar.xz &&

tar xvf /sources/llvm-12.0.0.src.tar.xz &&
cd       llvm-12.0.0.src                &&

tar xf /sources/clang-12.0.0.src.tar.xz -C tools &&
mv      tools/clang-12.0.0.src tools/clang       &&

tar xf /sources/compiler-rt-12.0.0.src.tar.xz -C projects    &&
mv      projects/compiler-rt-12.0.0.src projects/compiler-rt &&

mkdir -v build &&
cd       build &&

CC=gcc CXX=g++                                  \
cmake -DCMAKE_INSTALL_PREFIX=/usr               \
      -DLLVM_ENABLE_FFI=ON                      \
      -DCMAKE_BUILD_TYPE=Release                \
      -DLLVM_BUILD_LLVM_DYLIB=ON                \
      -DLLVM_LINK_LLVM_DYLIB=ON                 \
      -DLLVM_ENABLE_RTTI=ON                     \
      -DLLVM_TARGETS_TO_BUILD="host;AMDGPU;BPF" \
      -DLLVM_BUILD_TESTS=ON                     \
      -DLLVM_BINUTILS_INCDIR=/usr/include       \
      -Wno-dev -G Ninja .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf llvm-12.0.0.src
```

### Haskell

`ghc` requires `ghc` to build, and unlike `go`, there is no option for bootstrapping via a C compiler. Thus, we need to first install a pre-built binary distribution of `ghc` via `ghcup`.

```sh
mkdir -pv ~/.ghcup/bin &&

curl https://downloads.haskell.org/~ghcup/0.1.14/x86_64-linux-ghcup-0.1.14 -o ~/.ghcup/bin/ghcup &&
chmod +x ~/.ghcup/bin/ghcup          &&
~/.ghcup/bin/ghcup install ghc-9.0.1 &&
~/.ghcup/bin/ghcup set 9.0.1
```

Tests are optional, but if you choose to run them, `ghc` takes a `THREADS` argument and offers various suites, including `fasttest`, `test`, and `slowtest`. The example invocation uses `fasttest`. If built with `LLVM`, at least 5 tests from the fragile suite were failing as of `LLVM-12.0.0` and `ghc-9.1.0`.

Beware that `ghc` applies heavy optimizations unless you tell it not to, so this will likely take a very long time. `ghc` is also quite massive, so unless you expect to be doing a lot of debugging, you may wish to use `make install-strip` rather than `make install` to install it and its libs.

To build html documentation, `sphinx-build` is required, which can be installed by running `pip install [--user] sphinx` if you did not already do that earlier, or it can be installed in a virtualenv used only for the build.

```sh
curl https://downloads.haskell.org/~ghc/9.0.1/ghc-9.0.1-src.tar.xz -o /sources/ghc-9.0.1.src.tar.xz       &&
curl https://downloads.haskell.org/~ghc/9.0.1/ghc-9.0.1-testsuite.tar.xz -o /sources/ghc-9.0.1-testsuite.tar.xz &&

tar xvf /sources/ghc-9.0.1.src.tar.xz       &&
tar xvf /sources/ghc-9.0.1-testsuite.tar.xz &&
cd       ghc-9.0.1                          &&

PATH=/home/lfs/.local/bin:$PATH  \
GHC=/home/lfs/.ghcup/bin/ghc     \
./configure --prefix=/usr        \
            --with-system-libffi &&

cat > mk/build.mk << EOF
BUILD_EXTRA_PKGS=YES
V=0
EOF

make                                                                           &&
make fasttest \
     THREADS=$(lscpu | grep CPU\(s\) | head -n 1 | cut -d ':' -f2 | tr -d ' ') &&
sudo make install                                                              &&

cd .. &&
rm -rf ghc-9.0.1
```

Don't worry about `make` throwing errors because `haddock` isn't happy. It just means some links in the generated html documentation won't work.

To also build `cabal`, the Haskell package manager, we first need to build quite a few dependencies. The README for `cabal-install` claims there is a `bootstrap.sh` script that does all of this automatically, but as of 25 May 2021, I can find no evidence of that file's existence in the source tree for `cabal-install-3.4.0.0`. Instead, we will create a script manually that does roughly what the `bootstrap` script used to do. From `'/sources`:

```sh
mkdir cabal-libs &&
cd    cabal-libs

cat > hackage-build.sh << EOF
#!/bin/sh
set -e

download() {
    curl https://hackage.haskell.org/packages/archive/\${1}/\${2}/\${1}-\${2}.tar.gz -o \${1}-\${2}.tar.gz
    tar xzf \${1}-\${2}.tar.gz
    rm \${1}-\${2}.tar.gz
}

build() {
    pushd \${1}-\${2}

    sed -i 's/base.*<.*4.1[45]/base < 4.16/' \${1}.cabal
    sed -i 's/ghc-prim.*<.*0.[567]/ghc-prim < 0.8/' \${1}.cabal
    sed -i 's/Cabal.*<.*3.4/Cabal < 3.5/' \${1}.cabal
    sed -i 's/<.*3.4/< 3.5/' \${1}.cabal
    sed -i 's/template-haskell.*<.*2.17/template-haskell < 2.18/' \${1}.cabal

    if [ -e ./src/Text/Regex/Base/Context.hs ]; then
        sed -i 's/(!0)/(! 0)/g' ./src/Text/Regex/Base/Context.hs
    fi

    runghc Setup configure                    \
        --prefix=/usr                         \
        --with-compiler=/usr/bin/ghc          \
        --with-hc-pkg=/usr/bin/ghc-pkg        \
        --with-gcc=/usr/bin/gcc               \
        --with-ld=/usr/bin/ld                 \
        --global                              \
        --enable-optimization=2               \
        --enable-shared                       \
        --enable-executable-dynamic           \
        --disable-library-vanilla             \
        --docdir=/usr/share/doc/\${1}-\${2}   \
        --libdir=/usr/lib                     \
        --dynlibdir=/usr/lib                  \
        --libsubdir=cabal-3.4.0.0/\${1}-\${2}
    runghc Setup build \${MAKEFLAGS}

    popd
}

install() {
    pushd \${1}-\${2}

    sudo runghc Setup copy
    sudo runghc Setup register

    popd
}

download "\$1" "\$2"
build "\$1" "\$2"
install "\$1" "\$2"
EOF
chmod +x hackage-build.sh
```

`cabal-install-3.4.0.0` has the following dependencies:

```
async >=2.0 && <2.3,
base >=4.8 && <4.15,
base16-bytestring >=0.1.1 && <0.2,
cryptohash-sha256 >=0.11 && <0.12,
echo >=0.1.3 && <0.2,
edit-distance >=0.2.2 && <0.3,
hackage-security >=0.6.0.1 && <0.7,
hashable >=1.0 && <1.4,
network-uri >=2.6.0.2 && <2.7,
random >=1.2 && <1.3,
regex-base >=0.94.0.0 && <0.95,
regex-posix >=0.96.0.0 && <0.97,
resolv >=0.1.1 && <0.2,
tar >=0.5.0.3 && <0.6,
zlib >=0.5.3 && <0.7
```

We can search through `hackage` to find the latest version of each library in each of these ranges, plus their own transitive dependencies, minus what is provided by `ghc`, put them in a dependency order that allows them to build, and put those into a separate file:

```sh
cat > cabal-deps << EOF
th-compat 0.1.2
network-uri 2.6.4.1
network 3.1.2.1
HTTP 4000.3.16
zlib 0.6.2.3
splitmix 0.1.0.3
random 1.2.0
hashable 1.3.2.0
async 2.2.3
base16-bytestring 0.1.1.7
base64-bytestring 1.0.0.3
cryptohash-sha256 0.11.102.0
resolv 0.1.2.0
mintty 0.1.2
echo 0.1.4
edit-distance 0.2.2.1
ed25519 0.0.5.0
tar 0.5.1.1
digest 0.0.1.2
lukko 0.1.1.3
hackage-security 0.6.0.1
regex-base 0.94.0.0
regex-posix 0.96.0.0
EOF
```

Build each of these in order:

```sh
IFS=$'\n'
for dep in $(cat cabal-deps); do
    ./hackage-build.sh "$dep"
done
```

Several packages claim they depend on earlier versions of `base`, but theoretically that should mean they won't compile with `ghc-9.0.1` if that were true, so it is likely they are being too conservative in their `cabal` files.

The `regex-base` package needs to be patched in the file `src/Text/Regex/Base/Context.hs` on lines 303 and 304 adding a space between the `!` and `0` in the expression `(!0)`, which is found on both lines. `ed25519` also claims to depend on `ghc-prim < 0.7`, which needs to be changed, as again, if that were true, it should probably not compile. We attempt to apply these patches automatically in the build script, but if it fails, you know where to look.

Now build `cabal` itself:

```sh
./hackage-build.sh cabal-install 3.4.0.0
```

I have confirmed that `cabal` will run after applying these patches to the dependencies.

Go ahead and clean up the massive installation of `ghc` left behind by `ghcup`:

```sh
rm -rf /home/lfs/.ghcup
```

### Rust

Take the warnings about `ghc` and quadruple them. `rust` vendors its own version of `LLVM` and caches a ton of dependencies. We can tell it to use the system `LLVM`, but the download and build will nonetheless be quite large. The bootstrapping process creates multiple copies of all libraries and tools and it is not clear what is safe to delete before attempting to install. I had a 30 GB disk (Linux From Scratch's recommended size) and had to shut down my VM to expand it because that was not enough space to build the documentation, even after deleting all other downloaded source tarballs and re-stripping all executables and libraries.

The build below takes up roughly 15 GB on disk when completed. The installation is 1.5 GB, with the rest taken up by all of the intermediate bootstrap and test binaries.

To start, first we'll grab some patches from the folks at `Arch Linux`:

```sh
curl https://raw.githubusercontent.com/archlinux/svntogit-packages/packages/rust/trunk/0001-bootstrap-Change-libexec-dir.patch -o rustc-bootstrap-change-libexec-dir.patch
curl https://raw.githubusercontent.com/archlinux/svntogit-packages/packages/rust/trunk/0001-cargo-Change-libexec-dir.patch -o cargo-change-libexec-dir.patch
curl https://raw.githubusercontent.com/archlinux/svntogit-packages/packages/rust/trunk/0002-compiler-Change-LLVM-targets.patch -o rustc-change-llvm-targets.patch
```

Now download the `rustc` tarball itself, apply the patches, and create the build config:

```sh
curl https://static.rust-lang.org/dist/rustc-1.52.1-src.tar.gz -o rustc-1.52.1-src.tar.gz
tar xzvf rustc-1.52.1-src.tar.gz
cd rustc-1.52.1-src
```

Now apply the patches and create a config file for `cargo`:

```sh
patch -Np1 -i ../rustc-bootstrap-change-libexec-dir.patch         &&
patch -d src/tools/cargo -Np1 < ../cargo-change-libexec-dir.patch &&
patch -Np1 -i ../rustc-change-llvm-targets.patch

cat > config.toml << EOF
[llvm]
link-shared = true

[build]
target = ["x86_64-unknown-linux-gnu"]
tools = ["cargo", "rls", "clippy", "miri", "rustfmt", "analysis", "src"]
python = "/usr/bin/python"
extended = true
sanitizers = true
profiler = true
vendor = true

[install]
prefix = "/usr/lib/rustc-1.52.1"

[rust]
# LLVM crashes when passing an object through ThinLTO twice.  This is triggered
# when using rust code in cross-language LTO if libstd was built using ThinLTO.
# http://blog.llvm.org/2019/09/closing-gap-cross-language-lto-between.html
# https://github.com/rust-lang/rust/issues/54872
codegen-units-std = 1
codegen-tests = false
debuginfo-level-std = 2
channel = "stable"
rpath = false

[target.x86_64]
llvm-config = "/usr/bin/llvm-config"
EOF
```

Build:

```sh
export RUSTFLAGS="$RUSTFLAGS -C link-args=-lffi"
./x.py build -j 64
```

Test:

```sh
./x.py test -j 64 --verbose --no-fail-fast | tee rustc-tests.log
```

First, do a destdir install:

```sh
export LIBSSH2_SYS_USE_PKG_CONFIG=1           &&
DESTDIR=${PWD}/install python3 ./x.py install &&
unset LIBSSH2_SYS_USE_PKG_CONFIG
```

Now, install to the system:

```sh
sudo chown -R root:root install                                            &&
sudo cp -a install/* /                                                     &&
sudo mv -v /usr/lib/rustc-1.52.1/share/doc/rust /usr/share/doc/rust-1.52.1 &&
sudo mv -v /usr/lib/rustc-1.52.1/share/man/man1/* /usr/share/man/man1/     &&
sudo mv -v /usr/lib/rustc-1.52.1/share/zsh/site-functions/* /usr/share/zsh/site-functions/ &&
for TOOL in $(ls /usr/lib/rustc-1.52.1/bin); do
    sudo ln -sv ../../../usr/lib/rustc-1.52.1/bin/${TOOL} /usr/bin/${TOOL}
done
```

Configure `ld.so` to find the runtime libraries:

```sh
sudo su -
cat > /etc/ld.so.conf.d/rustc-1.52.1.conf << EOF
# Begin rustc addition

/usr/lib/rustc-1.52.1/lib

# End rustc addition
EOF

ldconfig
```

To sanity check the installation:

```sh
mkdir -v ~/hellor && cd ~/hellor
cargo init
cargo run
```

This should print out "Hello world!" if all went well.

### Ruby

```sh
curl https://cache.ruby-lang.org/pub/ruby/3.0/ruby-3.0.1.tar.gz -o /sources/ruby-3.0.1.tar.gz &&

tar xzvf /sources/ruby-3.0.1.tar.gz &&
cd        ruby-3.0.1                &&

./configure --prefix=/usr   \
            --enable-shared \
            --docdir=/usr/share/doc/ruby-3.0.0 &&

make              &&
make -k check     &&
sudo make install &&

cd .. &&
rm -rf ruby-3.0.1
```

### SWIG

```sh
curl https://downloads.sourceforge.net/swig/swig-4.0.2.tar.gz -o /sources/swig-4.0.2.tar.gz &&

tar xzvf /sources/swig-4.0.2.tar.gz &&
cd        swig-4.0.2                &&

./configure --prefix=/usr \
            --without-maximum-compile-warnings &&

make                                               &&
sudo make install                                  &&
sudo install -v -m755 -d /usr/share/doc/swig-4.0.2 &&
sudo cp -v -R Doc/* /usr/share/doc/swig-4.0.2      &&

cd .. &&
rm -rf swig-4.0.2
```

### Additional Python modules

#### docutils

```sh
curl https://downloads.sourceforge.net/docutils/docutils-0.17.1.tar.gz -o /sources/docutils-0.17.1.tar.gz &&

tar xzvf /sources/docutils-0.17.1.tar.gz &&
cd        docutils-0.17.1                &&

python setup.py build                     &&
sudo python setup.py install --optimize=1 &&

for f in /usr/bin/rst*.py; do
  sudo ln -svf $(basename $f) /usr/bin/$(basename $f .py)
done

cd .. &&
sudo rm -rf docutils-0.17.1
```

#### PyCairo

```sh
curl https://github.com/pygobject/pycairo/releases/download/v1.20.1/pycairo-1.20.1.tar.gz -o /sources/pycairo-1.20.1.tar.gz &&

tar xzvf /sources/pycairo-1.20.1.tar.gz &&
cd        pycairo-1.20.1                &&

python3 setup.py build                       &&
sudo python3 setup.py install --optimize=1   &&
sudo python3 setup.py install_pycairo_header &&
sudo python3 setup.py install_pkgconfig      &&

cd .. &&
sudo rm -rf pycairo-1.20.1
```

#### PyGObject

```sh
curl https://download.gnome.org/sources/pygobject/3.40/pygobject-3.40.1.tar.xz -o /sources/pygobject-3.40.1.tar.xz &&

tar xvf /sources/pygobject-3.40.1.tar.xz &&
cd       pygobject-3.40.1                &&

mkdir build &&
cd    build &&

meson --prefix=/usr \
      --buildtype=release .. &&

ninja              &&
sudo ninja install &&

cd ../.. &&
rm -rf pygobject-3.40.1
```

#### d-bus

```sh
curl https://dbus.freedesktop.org/releases/dbus-python/dbus-python-1.2.16.tar.gz -o /sources/dbus-python-1.2.16.tar.gz &&

tar xzvf /sources/dbus-python-1.2.16.tar.gz &&
cd        dbus-python-1.2.16                &&

mkdir python3 &&
pushd python3 &&

PYTHON=/usr/bin/python3 \
../configure --prefix=/usr \
             --docdir=/usr/share/doc/dbus-python-1.2.16 &&

make &&
popd &&

make -C python3 check        &&
sudo make -C python3 install &&

cd .. &&
rm -rf dbus-python-1.2.16
```

#### Pygments

```sh
curl https://files.pythonhosted.org/packages/source/P/Pygments/Pygments-2.9.0.tar.gz -o /sources/Pygments-2.9.0.tar.gz &&

tar xzvf /sources/Pygments-2.9.0.tar.gz &&
cd        Pygments-2.9.0                &&

sudo python3 setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf Pygments-2.9.0
```

#### lxml

```sh
curl https://files.pythonhosted.org/packages/source/l/lxml/lxml-4.6.3.tar.gz -o /sources/lxml-4.6.3.tar.gz &&

tar xzvf /sources/lxml-4.6.3.tar.gz &&
cd        lxml-4.6.3                &&

python3 setup.py build                     &&
sudo python3 setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf lxml-4.6.3
```

#### MarkupSafe

```sh
curl https://files.pythonhosted.org/packages/source/M/MarkupSafe/MarkupSafe-2.0.1.tar.gz -o /sources/MarkupSafe-2.0.1.tar.gz &&

tar xzvf /sources/MarkupSafe-2.0.1.tar.gz &&
cd        MarkupSafe-2.0.1                &&

python3 setup.py build                     &&
sudo python3 setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf MarkupSafe-2.0.1
```

#### PyYAML

```sh
curl https://pyyaml.org/download/pyyaml/PyYAML-5.3.1.tar.gz -o /sources/PyYAML-5.3.1.tar.gz &&

tar xzvf /sources/PyYAML-5.3.1.tar.gz &&
cd        PyYAML-5.3.1                &&

python3 setup.py build                     &&
sudo python3 setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf PyYAML-5.3.1
```

#### Jinja2

```sh
curl https://files.pythonhosted.org/packages/source/J/Jinja2/Jinja2-3.0.1.tar.gz -o /sources/Jinja2-3.0.1.tar.gz &&

tar xzvf /sources/Jinja2-3.0.1.tar.gz &&
cd        Jinja2-3.0.1                &&

sudo python3 setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf Jinja2-3.0.1
```

#### Mako

```sh
curl https://files.pythonhosted.org/packages/source/M/Mako/Mako-1.1.4.tar.gz -o /sources/Mako-1.1.4.tar.gz &&

tar xzvf /sources/Mako-1.1.4.tar.gz &&
cd        Mako-1.1.4                &&

sudo python3 setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf Mako-1.1.4
```
