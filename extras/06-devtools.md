# Developer tools

## Source control

### git

```sh
curl https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.31.1.tar.xz -o git-2.31.1.tar.xz
curl https://mirrors.edge.kernel.org/pub/software/scm/git/git-manpages-2.31.0.tar.xz -o git-manpages-2.31.1.tar.xz
tar xvf git-2.30.1.tar.xz
cd git-2.30.1

./configure --prefix=/usr                   \
            --with-gitconfig=/etc/gitconfig \
            --with-python=python3
make -j
sudo make perllibdir=/usr/lib/perl5/5.32/site_perl install
sudo tar -xf ../git-manpages-2.31.1.tar.xz \
    -C /usr/share/man --no-same-owner --no-overwrite-dir

cd ..
rm -rf git-2.31.1
```

## Build tools

### CMake

Note that the BLFS lists `libarchive` as optional, but unless you edit the input files to the `bootstrap` script, it will fail, saying it is required. To install it:

```sh
curl https://github.com/libarchive/libarchive/releases/download/3.5.1/libarchive-3.5.1.tar.xz -o libarchive-3.5.1.tar.xz
tar xvf libarchive-3.5.1.tar.xz
cd libarchive-3.5.1

./configure --prefix=/usr --disable-static
make
sudo make install

cd ..
rm -rf libarchive-3.5.1
```

Beware that this package will not compile with `make` running parallel jobs.

```sh
curl https://cmake.org/files/v3.19/cmake-3.19.5.tar.gz -o cmake-3.19.5.tar.gz
tar xzvf cmake-3.19.5.tar.gz
cd cmake-3.19.5

sed -i '/"lib64"/s/64//' Modules/GNUInstallDirs.cmake &&
MAKEFLAGS="-j1"                  \
./bootstrap --prefix=/usr        \
            --system-libs        \
            --mandir=/share/man  \
            --no-system-jsoncpp  \
            --no-system-librhash \
            --docdir=/usr/share/doc/cmake-3.19.5
make -j1
sudo make install

cd ..
rm -rf cmake-3.19.5
```

## Programming languages

## golang

`go` needs to be bootstrapped from `go-1.4`, the last version that was still written in C, so we first need to download and build that.

```sh
curl https://dl.google.com/go/go1.4-bootstrap-20171003.tar.gz -o go1.4-bootstrap-20171003.tar.gz
tar xzvf go1.4-bootstrap-20171003.tar.gz
cd go/src

CGO_ENABLED=0 ./make.bash
```

Now we can build the latest stable `go` using `go-1.4`:

```sh
curl https://go.googlesource.com/go/+archive/refs/tags/go1.16.4.tar.gz -o go1.16.4.tar.gz
mkdir -v go1.16.4
tar xzvf go1.16.4.tar.gz -C go1.16.4
cd go1.16.4/src

export GOROOT_FINAL=/usr/lib/go
export GOROOT_BOOTSTRAP=/sources/go
export GOPATH=/sources/go1.16.4
./make.bash

PATH="$GOPATH/bin:$PATH" go install -v -race std
PATH="$GOPATH/bin:$PATH" go install -v -buildmode=shared std
```

To install:

```sh
sudo install -vdm755 /usr/lib/go &&
sudo install -vdm755 /usr/share/doc/go
sudo cp -a bin pkg src lib misc api test /usr/lib/go
sudo cp -r doc/* /usr/share/doc/go
sudo ln -svf /usr/lib/go/bin/go /usr/bin/go
sudo ln -svf /usr/lib/go/bin/gofmt /usr/bin/gofmt
```

Now delete both source trees:

```sh
cd ../..
rm -rf go*
```

## LLVM and Clang

```sh
curl https://github.com/llvm/llvm-project/releases/download/llvmorg-11.1.0/llvm-11.1.0.src.tar.xz -o llvm-11.1.0.src.tar.xz
curl https://github.com/llvm/llvm-project/releases/download/llvmorg-11.1.0/clang-11.1.0.src.tar.xz -o clang-11.1.0.src.tar.xz
curl https://github.com/llvm/llvm-project/releases/download/llvmorg-11.1.0/compiler-rt-11.1.0.src.tar.xz -o compiler-rt-11.1.0.src.tar.xz
tar xvf llvm-11.1.0.src.tar.xz
cd llvm-11.1.0.src

tar -xf ../clang-11.1.0.src.tar.xz -C tools &&
mv tools/clang-11.1.0.src tools/clang

tar -xf ../compiler-rt-11.1.0.src.tar.xz -C projects &&
mv projects/compiler-rt-11.1.0.src projects/compiler-rt

mkdir -v build &&
cd       build
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
      -Wno-dev -G Ninja ..
ninja
sudo ninja install

cd ../..
rm -rf llvm-11.1.0.src
```

## Haskell

`ghc` requires `ghc` to build, and unlike `go`, there is no option for bootstrapping via a C compiler. Thus, we need to first install a pre-built binary distribution of `ghc` via `ghcup`. When attempting to install `ghc`, `ghcup` will try to link against `libtinfo` from the `ncurses` package. This library is built into `libncursesw`, so a link needs to be created:

```sh
sudo ln -sv ../../lib/libncursesw.so.6 /usr/lib/libtinfo.so.6
```

```sh
mkdir -pv ~/.ghcup/bin
curl https://downloads.haskell.org/~ghcup/0.1.14/x86_64-linux-ghcup-0.1.14 -o ~/.ghcup/bin/ghcup
chmod +x ~/.ghcup/bin/ghcup
~/.ghcup/bin/ghcup install ghc-9.0.1
~/.ghcup/bin/ghcup set 9.0.1
```

Tests are optional, but if you choose to run them, `ghc` takes a `THREADS` argument and offers various suites, including `fasttest`, `test`, and `slowtest`. The example invocation uses `fasttest`. If built with `LLVM`, at least 5 tests were failing as of `LLVM-11.1.0` and `ghc-9.1.0`.

To build html documentation, `sphinx-build` is required, which can be installed by running `pip install --user sphinx`. Beware that `ghc` applies heavy optimizations unless you tell it not to, so this will likely take a very long time. Beware that `ghc` is quite massive, so unless you expect to be doing a lot of debugging, you may wish to use `make install-strip` rather than `make install` to install it and its libs.

```sh
curl https://downloads.haskell.org/~ghc/9.0.1/ghc-9.0.1-src.tar.xz -o ghc-9.0.1.src.tar.xz
curl https://downloads.haskell.org/~ghc/9.0.1/ghc-9.0.1-testsuite.tar.xz -o ghc-9.0.1-testsuite.tar.xz
tar xvf ghc-9.0.1.src.tar.xz
tar xvf ghc-9.0.1-testsuite.tar.xz
cd ghc-9.0.1

PATH=/home/lfs/.local/bin:$PATH  \
GHC=/home/lfs/.ghcup/bin/ghc     \
./configure --prefix=/usr        \
            --with-system-libffi
cat > mk/build.mk << EOF
BUILD_EXTRA_PKGS=YES
V=0
EOF
make
make fasttest THREADS=$(lscpu | grep CPU\(s\) | head -n 1 | cut -d ':' -f2 | tr -d ' ')
sudo make install
```

Don't worry about `make` throwing errors because `haddock` isn't happy. It just means some links in the generated html documentation won't work.

To also build `cabal`, the Haskell package manager, we first need to build quite a few dependencies. The README for `cabal-install` claims there is a `bootstrap.sh` script that does all of this automatically, but as of 25 May 2021, I can find no evidence of that file's existence in the source tree for `cabal-install-3.4.0.0`. Instead, we will create a script manually that does roughly what the `bootstrap` script used to do. From `'/sources`:

```sh
mkdir cabal-libs
cd cabal-libs

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
    sed -i 's/ghc-prim.*<.*0.7/ghc-prim < 0.8/' \${1}.cabal

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

## Rust

```sh

```

## Ruby

```sh
curl https://cache.ruby-lang.org/pub/ruby/3.0/ruby-3.0.1.tar.gz -o ruby-3.0.1.tar.gz
tar xzvf ruby-3.0.1.tar.gz
cd ruby-3.0.1

./configure --prefix=/usr   \
            --enable-shared \
            --docdir=/usr/share/doc/ruby-3.0.0
make
make -k check
sudo make install

cd ..
rm -rf ruby-3.0.1
```

## Documentation generation

### Doxygen

```sh

```

### pandoc

```sh

```

### mdbook

```sh

```
