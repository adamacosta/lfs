# Libraries

## libimagequant

Adds image quantization features to `PILLOW`.

```sh
curl https://github.com/ImageOptim/libimagequant/archive/refs/tags/2.15.1.zip -o /sources/libimagequant-2.15.1.zip &&

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
curl https://github.com/lz4/lz4/archive/refs/tags/v1.9.3.tar.gz -o /sources/lz4-1.9.3.tar.gz &&

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
curl "https://www.hdfgroup.org/package/hdf5-1-12-0-tar-gz/?wpdmdl=14582&refresh=60ca8587d16d31623885191" -o /sources/hdf5-1.12.0.tar.gz &&

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
