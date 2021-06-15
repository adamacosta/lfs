# Documentation generation tools

If you wish to build `pandoc`, it requires a Haskell compiler, and `mdbook` requires `rustc` and `cargo`.

## asciidoc

```sh
curl https://github.com/asciidoc/asciidoc-py3/releases/download/9.1.0/asciidoc-9.1.0.tar.gz -o /sources/asciidoc-9.1.0.tar.gz &&

tar xzvf /sources/asciidoc-9.1.0.tar.gz &&
cd        asciidoc-9.1.0                &&

sed -i 's:doc/testasciidoc.1::' Makefile.in &&
rm doc/testasciidoc.1.txt                   &&

./configure --prefix=/usr     \
            --sysconfdir=/etc \
            --docdir=/usr/share/doc/asciidoc-9.1.0 &&

make              &&
sudo make install &&
sudo make docs    &&

cd .. &&
rm -rf asciidoc-9.1.0
```

## Doxygen

```sh
curl http://doxygen.nl/files/doxygen-1.9.1.src.tar.gz -o /sources/doxygen-1.9.1.src.tar.gz &&

tar xzvf /sources/doxygen-1.9.1.src.tar.gz &&
cd        doxygen-1.9.1                    &&

rm src/._xmlgen.cpp &&

mkdir -v build &&
cd       build &&

cmake -G "Unix Makefiles"         \
      -DCMAKE_BUILD_TYPE=Release  \
      -DCMAKE_INSTALL_PREFIX=/usr \
      -Wno-dev .. &&

make                                               &&
make tests                                         &&
sudo make install                                  &&
sudo install -vm644 ../doc/*.1 /usr/share/man/man1 &&

cd ../.. &&
rm -rf doxygen-1.9.1
```

## Sphinx

`sphinx-build` is a Python application with dependencies on quite a few Python packages that will need to be installed first.

### snowballstemmer

```sh
curl https://files.pythonhosted.org/packages/a3/3d/d305c9112f35df6efb51e5acd0db7009b74d86f35580e033451b5994a0a9/snowballstemmer-2.1.0.tar.gz -o /sources/python-snowball-stemmer-2.1.0.tar.gz &&

tar xzvf /sources/python-snowball-stemmer-2.1.0.tar.gz &&
cd        snowballstemmer-2.1.0                        &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf snowballstemmer-2.1.0
```

### pytz

```sh
curl https://files.pythonhosted.org/packages/b0/61/eddc6eb2c682ea6fd97a7e1018a6294be80dba08fa28e7a3570148b4612d/pytz-2021.1.tar.gz -o /sources/pytz-2021.1.tar.gz &&

tar xzvf /sources/pytz-2021.1.tar.gz &&
cd        pytz-2020.1                &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf pytz-2020.1
```

### babel

```sh
curl https://files.pythonhosted.org/packages/17/e6/ec9aa6ac3d00c383a5731cc97ed7c619d3996232c977bb8326bcbb6c687e/Babel-2.9.1.tar.gz -o /sources/python-babel-2.9.1.tar.gz &&

tar xzvf /sources/python-babel-2.9.1.tar.gz &&
cd        Babel-2.9.1                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf Babel-2.9.1
```

### alabaster

```sh
curl https://files.pythonhosted.org/packages/cc/b4/ed8dcb0d67d5cfb7f83c4d5463a7614cb1d078ad7ae890c9143edebbf072/alabaster-0.7.12.tar.gz -o /sources/python-alabaster-0.7.12.tar.gz &&

tar xzvf /sources/python-alabaster-0.7.12.tar.gz &&
cd        alabaster-0.7.12                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf alabaster-0.7.12
```

### imagesize

```sh
curl https://files.pythonhosted.org/packages/e4/9f/0452b459c8ba97e07c3cd2bd243783936a992006cf4cd1353c314a927028/imagesize-1.2.0.tar.gz -o /sources/python-imagesize-1.2.0.tar.gz &&

tar xzvf /sources/python-imagesize-1.2.0.tar.gz &&
cd        imagesize-1.2.0                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf imagesize-1.2.0
```

### chardet

```sh
curl https://files.pythonhosted.org/packages/ee/2d/9cdc2b527e127b4c9db64b86647d567985940ac3698eeabc7ffaccb4ea61/chardet-4.0.0.tar.gz -o /sources/python-chardet-4.0.0.tar.gz &&

tar xzvf /sources/python-chardet-4.0.0.tar.gz &&
cd        chardet-4.0.0                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf chardet-4.0.0
```

### idna

```sh
curl https://files.pythonhosted.org/packages/cb/38/4c4d00ddfa48abe616d7e572e02a04273603db446975ab46bbcd36552005/idna-3.2.tar.gz -o /sources/python-idna-3.2.tar.gz &&

tar xzvf /sources/python-idna-3.2.tar.gz &&
cd        idna-3.2                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf idna-3.2
```

### urllib3

```sh
curl https://files.pythonhosted.org/packages/94/40/c396b5b212533716949a4d295f91a4c100d51ba95ea9e2d96b6b0517e5a5/urllib3-1.26.5.tar.gz -o /sources/python-urllib3-1.26.5.tar.gz &&

tar xzvf /sources/python-urllib3-1.26.5.tar.gz &&
cd        urllib3-1.26.5                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf urllib3-1.26.5
```

### certifi

```sh
curl https://files.pythonhosted.org/packages/6d/78/f8db8d57f520a54f0b8a438319c342c61c22759d8f9a1cd2e2180b5e5ea9/certifi-2021.5.30.tar.gz -o /sources/python-certifi-2021.5.30.tar.gz &&

tar xzvf /sources/python-certifi-2021.5.30.tar.gz &&
cd        certifi-2021.5.30                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf certifi-2021.5.30
```

### requests

```sh
curl https://files.pythonhosted.org/packages/6b/47/c14abc08432ab22dc18b9892252efaf005ab44066de871e72a38d6af464b/requests-2.25.1.tar.gz -o /sources/python-requests-2.25.1.tar.gz &&

tar xzvf /sources/python-requests-2.25.1.tar.gz &&
cd        requests-2.25.1                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf requests-2.25.1
```

### packaging

```sh
curl https://files.pythonhosted.org/packages/86/3c/bcd09ec5df7123abcf695009221a52f90438d877a2f1499453c6938f5728/packaging-20.9.tar.gz -o /sources/python-packaging-20.9.tar.gz &&

tar xzvf /sources/python-packaging-20.9.tar.gz &&
cd        packaging-20.9                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf packaging-20.9
```

Now install `sphinx` itself.

```sh
curl https://files.pythonhosted.org/packages/8d/4d/8a80613d0ceefca5a84e2e30b29da7719d429b4adcdb793d86079fad3790/Sphinx-4.0.2.tar.gz -o /sources/sphinx-4.0.2.tar.gz &&

tar xzvf /sources/sphinx-4.0.2.tar.gz &&
cd        Sphinx-4.0.2                &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf Sphinx-4.0.2
```

### pandoc

`pandoc` has many dependencies.

```
Glob                  >= 0.7      && < 0.11,
HTTP                  >= 4000.0.5 && < 4000.4,
HsYAML                >= 0.2      && < 0.3,
JuicyPixels           >= 3.1.6.1  && < 3.4,
SHA                   >= 1.6      && < 1.7,
aeson                 >= 0.7      && < 1.6,
aeson-pretty          >= 0.8.5    && < 0.9,
array                 >= 0.5      && < 0.6,
attoparsec            >= 0.12     && < 0.15,
base64-bytestring     >= 0.1      && < 1.3,
binary                >= 0.7      && < 0.11,
blaze-html            >= 0.9      && < 0.10,
blaze-markup          >= 0.8      && < 0.9,
bytestring            >= 0.9      && < 0.12,
case-insensitive      >= 1.2      && < 1.3,
citeproc              >= 0.4      && < 0.4.1,
commonmark            >= 0.2      && < 0.3,
commonmark-extensions >= 0.2.1.2  && < 0.3,
commonmark-pandoc     >= 0.2.1    && < 0.3,
connection            >= 0.3.1,
containers            >= 0.4.2.1  && < 0.7,
data-default          >= 0.4      && < 0.8,
deepseq               >= 1.3      && < 1.5,
directory             >= 1.2.3    && < 1.4,
doclayout             >= 0.3.0.1  && < 0.4,
doctemplates          >= 0.9      && < 0.10,
emojis                >= 0.1      && < 0.2,
exceptions            >= 0.8      && < 0.11,
file-embed            >= 0.0      && < 0.1,
filepath              >= 1.1      && < 1.5,
haddock-library       >= 1.10     && < 1.11,
hslua                 >= 1.1      && < 1.4,
hslua-module-path     >= 0.1.0    && < 0.2.0,
hslua-module-system   >= 0.2      && < 0.3,
hslua-module-text     >= 0.2.1    && < 0.4,
http-client           >= 0.4.30   && < 0.8,
http-client-tls       >= 0.2.4    && < 0.4,
http-types            >= 0.8      && < 0.13,
ipynb                 >= 0.1      && < 0.2,
jira-wiki-markup      >= 1.4      && < 1.5,
mtl                   >= 2.2      && < 2.3,
network               >= 2.6,
network-uri           >= 2.6      && < 2.8,
pandoc-types          >= 1.22     && < 1.23,
parsec                >= 3.1      && < 3.2,
process               >= 1.2.3    && < 1.7,
random                >= 1        && < 1.3,
safe                  >= 0.3.18   && < 0.4,
scientific            >= 0.3      && < 0.4,
skylighting           >= 0.10.5.1 && < 0.10.6,
skylighting-core      >= 0.10.5.1 && < 0.10.6,
split                 >= 0.2      && < 0.3,
syb                   >= 0.1      && < 0.8,
tagsoup               >= 0.14.6   && < 0.15,
temporary             >= 1.1      && < 1.4,
texmath               >= 0.12.3   && < 0.12.4,
text                  >= 1.1.1.0  && < 1.3,
text-conversions      >= 0.3      && < 0.4,
time                  >= 1.5      && < 1.12,
unicode-transforms    >= 0.3      && < 0.4,
unordered-containers  >= 0.2      && < 0.3,
xml                   >= 1.3.12   && < 1.4,
xml-conduit           >= 1.9.1.1  && < 1.10,
unicode-collation     >= 0.1.1    && < 0.2,
zip-archive           >= 0.2.3.4  && < 0.5,
zlib                  >= 0.5      && < 0.7
```

```sh
curl https://hackage.haskell.org/package/pandoc-2.14.0.1/pandoc-2.14.0.1.tar.gz -o /sources/pandoc-2.14.0.1.tar.gz &&

tar xvzf /sources/pandoc-2.14.0.1.tar.gz &&
cd        pandoc-2.14.0.1                &&

cabal update                           &&
sudo cabal install --lib                       \
                   --prefix=/usr               \
                   --libdir=/usr/lib           \
                   --dynlibdir=/usr/lib        \
                   --global                    \
                   --enable-shared             \
                   --enable-optimization=2     \
                   --enable-executable-dynamic \
                   --disable-library-vanilla   \
                   --disable-static            \
                   --only-dependencies &&

cabal configure --prefix=/usr               \
                --global                    \
                --enable-executable-dynamic \
                --enable-optimization=2     \
                --docdir=/usr/share/doc/pandoc-2.14.0.1 &&

runghc Setup build -j      &&
rungch Setup test          &&
sudo runghc Setup copy     &&
sudo rungch Setup register &&

cd .. &&
rm -rf pandoc-2.14.0.1
```

```
th-compat-0.1.2
splitmix-0.1.0.3
random-1.2.0
hashable-1.3.2.0
async-2.2.3
base16-bytestring-0.1.1.7
base64-bytestring-1.0.0.3
digest-0.0.1.2
zlib-0.6.2.3
network-3.1.2.1
network-uri-2.6.4.1
```

```sh
sudo ghc-pkg unregister --force network-uri-2.6.4.1  &&
sudo cabal install --lib                       \
                   --prefix=/usr               \
                   --libdir=/usr/lib           \
                   --dynlibdir=/usr/lib        \
                   --global                    \
                   --enable-shared             \
                   --enable-optimization=2     \
                   --enable-executable-dynamic \
                   --disable-library-vanilla   \
                   --disable-static            \
                   --reinstall                 \
                   network-uri-2.6.4.1
```

### mdbook

```sh

```
