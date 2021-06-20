# Python scientific computing

## Additional build tools

### scikit-build

Build tools for `Python` sci stack.

```sh
wget https://files.pythonhosted.org/packages/71/02/1e94506b7bee5739317f2d141cebab7dab5bb5731b377e718fddd3b3e7e7/scikit-build-0.11.1.tar.gz -P /sources/python &&

tar xzvf /sources/scikit-build-0.11.1.tar.gz &&
cd        scikit-build-0.11.1                &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf scikit-build-0.11.1
```

### Cython

Syntax extensions to creat statically compiled `Python` extension modules.

```sh
wget https://files.pythonhosted.org/packages/d9/cd/0d2d90b27219c07f68f1c25bcc7b02dd27639d2180add9d4b73e70945869/Cython-0.29.23.tar.gz -P /sources/python &&

tar xzvf /sources/python/Cython-0.29.23.tar.gz &&
cd        Cython-0.29.23                       &&

python setup.py build                &&
python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf Cython-0.29.23
```

### pybind11

`Python` bindings for C++11.

```sh
wget https://files.pythonhosted.org/packages/46/0e/3131a9ae6e6cd1d718cfdd3e2c25f681f2db29be30763aa54b9310de5fea/pybind11-2.6.2.tar.gz -P /sources/python &&

tar xzvf /sources/python/pybind11-2.6.2.tar.gz &&
cd        pybind11-2.6.2                       &&

python setup.py build 
sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf pybind11-2.6.2
```

## Core libraries

### beautifulsoup

Web scraping library.

```sh
wget https://files.pythonhosted.org/packages/6b/c3/d31704ae558dcca862e4ee8e8388f357af6c9d9acb0cad4ba0fbbd350d9a/beautifulsoup4-4.9.3.tar.gz -P /sources/python &&

tar xzvf /sources/python/beautifulsoup4-4.9.3.tar.gz &&
cd        beautifulsoup4-4.9.3                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf beautifulsoup4-4.9.3
```

### soupsieve

CSS selector for `beautifulsoup`.

```sh
wget https://files.pythonhosted.org/packages/c8/3f/e71d92e90771ac2d69986aa0e81cf0dfda6271e8483698f4847b861dd449/soupsieve-2.2.1.tar.gz -P /sources/python &&

tar xzvf /sources/python/soupsieve-2.2.1.tar.gz &&
cd        soupsieve-2.2.1                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf soupsieve-2.2.1
```

### python-lz4

`Python` bindings for `lz4` to speedup `joblib`.

```sh
wget https://files.pythonhosted.org/packages/d9/c5/080234f5b6b698f56339edf7715d9256eca4eb3d35b36893227c399e69f9/lz4-3.1.3.tar.gz -P /sources/python &&

tar xzvf /sources/python/lz4-3.1.3.tar.gz &&
cd        lz4-3.1.3                       &&

python setup.py build -j 64               &&
sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf lz4-3.1.3
```

### PyICU

`Python` bindings for `icu`.

```sh
wget https://files.pythonhosted.org/packages/50/62/230bb78141a03cb3849a1f8ab390eed290a9430b3dabb496c51228271e0b/PyICU-2.7.3.tar.gz -P /sources/python &&

tar xzvf /sources/python/PyICU-2.7.3.tar.gz &&
cd        PyICU-2.7.3                       &&

python setup.py build -j 64               &&
sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf PyICU-2.7.3
```

### fastnumbers

Highly optimized string-to-number conversions library.

```sh
wget https://files.pythonhosted.org/packages/c9/8c/eb150c91beabdf4ac44d1aec8237709ca1593c615f6cd677914d4c3a2bdb/fastnumbers-3.1.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/fastnumbers-3.1.0.tar.gz &&
cd        fastnumbers-3.1.0                       &&

python setup.py build -j 64               &&
sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf fastnumbers-3.1.0
```

### natsort

Sorts strings with numbers embedded in them numerically rather than lexicograpically.

```sh
wget https://files.pythonhosted.org/packages/06/73/132e1986f7d37826e39825097e09f2c86b1339609926210ebdaec74a3b72/natsort-7.1.1.tar.gz -P /sources/python &&

tar xzvf /sources/python/natsort-7.1.1.tar.gz &&
cd        natsort-7.1.1                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf natsort-7.1.1
```

### psutil

Process monitoring, used by `joblib` and other `Python` libraries to prevent memory leaks in parallel code.

```sh
wget https://files.pythonhosted.org/packages/e1/b0/7276de53321c12981717490516b7e612364f2cb372ee8901bd4a66a000d7/psutil-5.8.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/psutil-5.8.0.tar.gz &&
cd        psutil-5.8.0                       &&

python setup.py build -j 64  &&
sudo python setup.py install &&

cd .. &&
sudo rm -rf psutil-5.8.0
```

### joblib

Run `Python` functions as pipelines.

```sh
wget https://files.pythonhosted.org/packages/6e/1b/b6705e67e288b4f66f99d362ffff0ac93bda3570b0fcb4a5a4e87172108d/joblib-1.0.1.tar.gz -P /sources/python &&

tar xzvf /sources/python/joblib-1.0.1.tar.gz &&
cd        joblib-1.0.1                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf joblib-1.0.1
```

### importlib_metadata

Third-party access to package metadata that is periodically merged into mainline `CPython`.

```sh
wget https://files.pythonhosted.org/packages/1c/7f/fe573d2225d7e0ea4bd9e8673b2c7fc0b2b2b2f86b36f8c5fe75b77e59c4/importlib_metadata-4.5.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/importlib_metadata-4.5.0.tar.gz &&
cd        importlib_metadata-4.5.0                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf importlib_metadata-4.5.0
```

### multipledispatch

Multiple dispatch generic functions for `Python`.

```sh
wget https://files.pythonhosted.org/packages/37/86/76c69eb0dac361c83e2e3952051bec40bd2f488127f5479d6222b5853f04/multipledispatch-0.6.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/multipledispatch-0.6.0.tar.gz &&
cd        multipledispatch-0.6.0                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf multipledispatch-0.6.0
```

### threadpoolctl

Fine-grained control of threadpool used by parallel native libraries to prevent oversubscription.

```sh
wget https://files.pythonhosted.org/packages/78/e8/e39dc842f512ab5be11efe83160ddb7ad3c0cc1b8d42ce8c0469a0d2b926/threadpoolctl-2.1.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/threadpoolctl-2.1.0.tar.gz &&
cd        threadpoolctl-2.1.0                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf threadpoolctl-2.1.0
```

### tornado

Asynchronous web framework use by in-browser interactive computing and data visualization packages.

```sh
wget https://files.pythonhosted.org/packages/cf/44/cc9590db23758ee7906d40cacff06c02a21c2a6166602e095a56cbf2f6f6/tornado-6.1.tar.gz -P /sources/python &&

tar xzvf /sources/python/tornado-6.1.tar.gz &&
cd        tornado-6.1                       &&

python setup.py build -j 64               &&
sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf tornado-6.1
```

### numpydoc

Extension to `Sphinx` allowing `Python` docstrings in `NumPy` format. Needed to build documentation for some of these libraries.

```sh
wget https://files.pythonhosted.org/packages/3d/fb/a70f636102045fc646656f2221c7fcdf92f7a9d71ba7c9875a949a58b3e8/numpydoc-1.1.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/numpydoc-1.1.0.tar.gz &&
cd        numpydoc-1.1.0                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf numpydoc-1.1.0
```

### sphinx-gallery

Extension to `sphinx` that automatically wraps `Python` scripts into html files to include in an examples gallery.

```sh
wget https://files.pythonhosted.org/packages/40/6e/c494d573669eb6a581d7c270b7efd9c1244086f818b9a6e767d6a2adfeda/sphinx-gallery-0.9.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/sphinx-gallery-0.9.0.tar.gz &&
cd        sphinx-gallery-0.9.0                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf sphinx-gallery-0.9.0
```

### nbsphinx

Render `jupyter` notebooks in documentation.

```sh
wget https://files.pythonhosted.org/packages/32/66/9b496b382c09b81f0ab0135987c4662360d2ec69c228cdd283dcb2fb096a/nbsphinx-0.8.6.tar.gz -P /sources/python &&

tar xzvf /sources/python/nbsphinx-0.8.6.tar.gz &&
cd        nbsphinx-0.8.6                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf nbsphinx-0.8.6
```

### PyData Sphinx Theme

Build the documentation for `NumPy`, `SciPy`, `pandas`, and `scikit-learn`.

```sh
wget https://files.pythonhosted.org/packages/21/6d/b72bd3ea7ad7fa43acda66c664d88e3578704e13fcf059d22dd3a2594315/pydata-sphinx-theme-0.6.3.tar.gz -P /sources/python &&

tar xzvf /sources/python/pydata-sphinx-theme-0.6.3.tar.gz &&
cd        pydata-sphinx-theme-0.6.3                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf pydata-sphinx-theme-0.6.3
```

## IPython and jupyter-lab

Improved interactive `Python` shell and in-browser execution framework and server.

### parso

Python parser that supports error recovery and round-trip parsing for different Python versions pulled out of `jedi` to suppor other projects.

```sh
wget https://files.pythonhosted.org/packages/5e/61/d119e2683138a934550e47fc8ec023eb7f11b194883e9085dca3af5d4951/parso-0.8.2.tar.gz -P /sources/python &&

tar xzvf /sources/python/parso-0.8.2.tar.gz &&
cd        parso-0.8.2                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf parso-0.8.2
```

### jedi

Static analysis and autocompletion tool for `Python`.

```sh
wget https://files.pythonhosted.org/packages/ac/11/5c542bf206efbae974294a61febc61e09d74cb5d90d8488793909db92537/jedi-0.18.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/jedi-0.18.0.tar.gz &&
cd        jedi-0.18.0                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf jedi-0.18.0
```

### decorator

The goal of the `decorator` module is to make it easy to define signature-preserving function decorators and decorator factories. It also includes an implementation of multiple dispatch and other niceties.

```sh
wget https://files.pythonhosted.org/packages/4f/51/15a4f6b8154d292e130e5e566c730d8ec6c9802563d58760666f1818ba58/decorator-5.0.9.tar.gz -P /sources/python &&

tar xzvf /sources/python/decorator-5.0.9.tar.gz &&
cd        decorator-5.0.9                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf decorator-5.0.9
```

### pickleshare

A small datastore with concurrency support.

```sh
wget https://files.pythonhosted.org/packages/d8/b6/df3c1c9b616e9c0edbc4fbab6ddd09df9535849c64ba51fcb6531c32d4d8/pickleshare-0.7.5.tar.gz -P /sources/python &&

tar xzvf /sources/python/pickleshare-0.7.5.tar.gz &&
cd        pickleshare-0.7.5                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf pickleshare-0.7.5
```

### traitlets

Enabled strong typing of attributes of `Python` objects. Powers the configuration system of IPython and Jupyter and the declarative API of IPython interactive widgets.

```sh
wget https://files.pythonhosted.org/packages/b8/24/e6dfe45ecfc4c2b0d6774565e631dc1b9e50de2c3536625d377963d56cae/traitlets-5.0.5.tar.gz -P /sources/python &&

tar xzvf /sources/python/traitlets-5.0.5.tar.gz &&
cd        traitlets-5.0.5                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf traitlets-5.0.5
```

### prompt_toolkit

Library for building powerful interactive command line applications in Python.

```sh
wget https://files.pythonhosted.org/packages/f7/e0/47738dadee0ec15ffbfa926f01586db2397201e0af3e06a0e669adfd6f2f/prompt_toolkit-3.0.18.tar.gz -P /sources/python &&

tar xzvf /sources/python/prompt_toolkit-3.0.18.tar.gz &&
cd        prompt_toolkit-3.0.18                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf prompt_toolkit-3.0.18
```

### backcall

Specifications for callback functions passed into an API.

```sh
wget https://files.pythonhosted.org/packages/a2/40/764a663805d84deee23043e1426a9175567db89c8b3287b5c2ad9f71aa93/backcall-0.2.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/backcall-0.2.0.tar.gz &&
cd        backcall-0.2.0                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf backcall-0.2.0
```

### executing



### astokens



### pure_eval



### stack_data

```sh
wget https://github.com/alexmojaki/stack_data/archive/refs/tags/v0.1.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/stack_data-0.1.0.tar.gz &&
cd        stack_data-0.1.0                       &&

git init                                   &&
git add .                                  &&
git config user.name "Nobody"              &&
git config user.email "noreply@nobody.com" &&
git commit -m "version"                    &&
git tag v0.1.0                             &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf stack_data-0.1.0
```

### matplotlib_inline

Display plots inline instead of popping up a window.

```sh
wget https://files.pythonhosted.org/packages/c1/fb/3361c4bf5ae8ffb9249132e70f4853ef511c0894a938fdff29320df55534/matplotlib-inline-0.1.2.tar.gz -P /sources/python &&

tar xzvf /sources/python/matplotlib_inline-0.1.2.tar.gz &&
cd        matplotlib_inline-0.1.2                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf matplotlib_inline-0.1.2
```

### ipython_genutils

```sh
wget https://files.pythonhosted.org/packages/e8/69/fbeffffc05236398ebfcfb512b6d2511c622871dca1746361006da310399/ipython_genutils-0.2.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/ipython_genutils-0.2.0.tar.gz &&
cd        ipython_genutils-0.2.0                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf ipython_genutils-0.2.0
```

### pexpect

Automated control of interactive processes.

```sh
wget https://files.pythonhosted.org/packages/e5/9b/ff402e0e930e70467a7178abb7c128709a30dfb22d8777c043e501bc1b10/pexpect-4.8.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/pexpect-4.8.0.tar.gz &&
cd        pexpect-4.8.0                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf pexpect-4.8.0
```

### ptyprocess

Run a subprocess in a pseudoterminal.

```sh
wget https://files.pythonhosted.org/packages/20/e5/16ff212c1e452235a90aeb09066144d0c5a6a8c0834397e03f5224495c4e/ptyprocess-0.7.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/ptyprocess-0.7.0.tar.gz &&
cd        ptyprocess-0.7.0                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf ptyprocess-0.7.0
```

### IPython

A powerful `Python` interactive shell and kernel for `Jupyter`.

```sh
wget https://files.pythonhosted.org/packages/59/4a/06f4527f0b47d80f0f86c17e14ee0bf0fedd7028a63a4f81c314f53c6636/ipython-7.24.1.tar.gz -P /sources/python &&

tar xzvf /sources/python/ipython-7.24.1.tar.gz &&
cd        ipython-7.24.1                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf ipython-7.24.1
```

### jupyter_core

Libraries on which all `Jupyter` projects depend.

```sh
wget https://files.pythonhosted.org/packages/24/9a/0ca76ccc95eeb3ee376c671e81bda2c61d148c7627443004d1ba0d085b80/jupyter_core-4.7.1.tar.gz -P /sources/python &&

tar xzvf /sources/python/jupyter_core-4.7.1.tar.gz &&
cd        jupyter_core-4.7.1                       &&

sudo pip install . &&

cd .. &&
sudo rm -rf jupyter_core-4.7.1
```

### jupyterlab-server

```sh
wget https://files.pythonhosted.org/packages/6a/d1/c1d3b3236d4ae99ba0c6994bdf9851378cfcdc948cbdd1b5881dd09c112a/jupyterlab_server-2.6.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/jupyterlab_server-2.6.0.tar.gz &&
cd        jupyterlab_server-2.6.0                       &&

sudo pip install . &&

cd .. &&
sudo rm -rf jupyterlab_server-2.6.0
```

### nbclassic

```sh
wget https://files.pythonhosted.org/packages/04/1f/0bee7bae33531275a2fc193bfe24cc4602c97d9b8e49007bcf6c057a4c0f/nbclassic-0.3.1.tar.gz -P /sources/python &&

tar xzvf /sources/python/nbclassic-0.3.1.tar.gz &&
cd        nbclassic-0.3.1                       &&

sudo pip install . &&

cd .. &&
sudo rm -rf nbclassic-0.3.1
```

### jupyterlab

```sh
wget https://files.pythonhosted.org/packages/85/17/4c6b94f3e28f53acc2b18e6d234a6ce8d274ac52cc4e6870a096695b38e3/jupyterlab-3.0.16.tar.gz -P /sources/python &&

tar xzvf /sources/python/jupyterlab-3.0.16.tar.gz &&
cd        jupyterlab-3.0.16                       &&

sudo pip install . &&

cd .. &&
sudo rm -rf jupyterlab-3.0.16
```

To test that it works:

```sh
jupyter lab
```

This will start a server you can then connect to at `localhost:8888`. If you wish to connect remotely (i.e. if the system is headless or you're doing this via ssh):

```sh
jupyter lab --ip=<ip address of server>
```

To listen on some port other than the default `8888`, `jupyter lab --port=<port>`.

You can listen to `0.0.0.0` if you wish, but beware that may open you up beyond your own LAN.

`jupyterlab` utilizes token authentication and you will need a token from the server to connect via the browser. To get a token, run in another terminal on the server:

```sh
jupyter server list
```

This will give you a URL to use with the token appended. Append that token to the IP address and port

## Core scientific libraries

### NumPy

Array computing for `Python`. Requires a `Fortran` compiler as well as `gcc`. Will use accelerated BLAS libraries if they are found on your system. `-j` allows parallel build jobs. I happen to have 64 threads on my processor. You can set this to an appropriate value for your machine.

`NumPy` comes with a pretty extensive test suite you may want to run, in which case don't immediately delete the souce dir but run `python runtests.py -v -m full` between the build and install steps. This requires `pytest`.

```sh
wget https://files.pythonhosted.org/packages/f3/1f/fe9459e39335e7d0e372b5e5dcd60f4381d3d1b42f0b9c8222102ff29ded/numpy-1.20.3.zip -P /sources/python &&

unzip /sources/python/numpy-1.20.3.zip &&
cd     numpy-1.20.3                    &&

python setup.py build -j 64               &&
sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf numpy-1.20.3
```

### SciPy

Scientific libraries offering common algorithms, i.e. basic curve fitting, interpolation, convolution, basis transforms, optimization, linear programming.

```sh
wget https://files.pythonhosted.org/packages/fe/fd/8704c7b7b34cdac850485e638346025ca57c5a859934b9aa1be5399b33b7/scipy-1.6.3.tar.gz -P /sources/python &&

tar xzvf /sources/python/scipy-1.6.3.tar.gz &&
cd        scipy-1.6.3                       &&

python setup.py build -j 64               &&
sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf scipy-1.6.3
```

### numexpr

Fast numeric expression evaluation for `NumPy`.

```sh
wget https://files.pythonhosted.org/packages/17/91/688234440a7b45a4f6a9931d2541de5e9e48b2c54b8fcd5951ab14bd6a12/numexpr-2.7.3.tar.gz -P /sources/python &&

tar xzvf /sources/python/numexpr-2.7.3.tar.gz &&
cd        numexpr-2.7.3                       &&

python setup.py build -j 64               &&
sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf numexpr-2.7.3
```

### PILLOW

Newer implementation of the `Python` image library.

```sh
wget https://files.pythonhosted.org/packages/21/23/af6bac2a601be6670064a817273d4190b79df6f74d8012926a39bc7aa77f/Pillow-8.2.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/pillow-8.2.0.tar.gz &&
cd        Pillow-8.2.0                       &&

python setup.py build -j 64               &&
sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf Pillow-8.2.0
```

### imread

Read image files into `NumPy` arrays.

```sh
wget https://files.pythonhosted.org/packages/4a/f9/9aa28049f0e03786b59ba9a142fa7209701f291af1b7c02b8ab4bdf39122/imread-0.7.4.tar.gz -P /sources/python &&

tar xzvf /sources/python/imread-0.7.4.tar.gz &&
cd        imread-0.7.4                       &&

python setup.py build -j 64               &&
sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf imread-0.7.4
```

### imageio

Library for reading and writing a wide range of image, video, scientific, and volumetric data formats.

```sh
wget https://files.pythonhosted.org/packages/c3/73/f37f428748c4f10a7991ac5bff00f113a34bcc0d0a78957d6e1cdc29a94e/imageio-2.9.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/imageio-2.9.0.tar.gz &&
cd        imageio-2.9.0                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf imageio-2.9.0
```

### tifffile

Read and write TIFF files. Can also write `NumPy` arrays to TIFF files.

```sh
wget https://files.pythonhosted.org/packages/e5/d5/9b0bf7e19cddb0ca0d45c2274a8824dbeaa2b166935e5f9ce2bc3aae331d/tifffile-2021.6.14.tar.gz -P /sources/python &&

tar xzvf /sources/python/tifffile-2021.6.14.tar.gz &&
cd        tifffile-2021.6.14                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf tifffile-2021.6.14
```

### PyWavelets

Library for performing wavelet transforms in `Python`.

```sh
wget https://files.pythonhosted.org/packages/17/6b/ef989cfb3acff2ea966c5f28a7443ccaec2ee136675dfa0366db2635f78a/PyWavelets-1.1.1.tar.gz -P /sources/python &&

tar xzvf /sources/python/PyWavelets-1.1.1.tar.gz &&
cd        PyWavelets-1.1.1                       &&

python setup.py build -j 64
sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf PyWavelets-1.1.1
```

### kiwisolver

C++ implementation of the Cassowary constraint solving algorithm..

```sh
wget https://files.pythonhosted.org/packages/90/55/399ab9f2e171047d28933ae4b686d9382d17e6c09a01bead4a6f6b5038f4/kiwisolver-1.3.1.tar.gz -P /sources/python &&

tar xzvf /sources/python/kiwisolver-1.3.1.tar.gz &&
cd        kiwisolver-1.3.1                       &&

python setup.py build -j 64
sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf kiwisolver-1.3.1
```

### Cycler

Library for cyclically iterating over keyword arguments.

```sh
wget https://files.pythonhosted.org/packages/c2/4b/137dea450d6e1e3d474e1d873cd1d4f7d3beed7e0dc973b06e8e10d32488/cycler-0.10.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/cycler-0.10.0.tar.gz &&
cd        cycler-0.10.0                       &&

python setup.py build                     &&
sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf cycler-0.10.0
```

### matplotlib

The foundational `Python` plotting library, inspired by `MATLAB`.

```sh
wget https://files.pythonhosted.org/packages/60/d3/286925802edaeb0b8834425ad97c9564ff679eb4208a184533969aa5fc29/matplotlib-3.4.2.tar.gz -P /sources/python &&

tar xzvf /sources/python/matplotlib-3.4.2.tar.gz &&
cd        matplotlib-3.4.2                       &&

python setup.py build -j 64 &&
sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf matplotlib-3.4.2
```

## Python OpenCV

Python bindings for `OpenCV`. I am not sure why they don't provide a source distribution for the latest 4.5.2 versions, but this seems to work.

```sh
wget https://files.pythonhosted.org/packages/bb/08/9dbc183a3ac6baa95fabf749ddb531bd26256edfff5b6c2195eca26258e9/opencv-python-4.5.1.48.tar.gz -P /sources/python &&
 
tar xzvf /sources/python/opencv-python-4.5.1.48.tar.gz &&
cd        opencv-python-4.5.1.48                       &&

python setup.py build -j 64
sudo python setup.py install --optimize==1 &&

cd .. &&
sudo rm -rf opencv-python-4.5.1.48
```

## Data Analysis and Machine Learning Toolkits

### PyTables

Read and store massive amounts of data using the `hdf5` standard.

```sh
wget https://files.pythonhosted.org/packages/2b/32/847ee3f521aae6a0be380d923a736162d698586f444df1ac24b98c65025c/tables-3.6.1.tar.gz -P /sources/python &&

tar xzvf /sources/python/tables-3.6.1.tar.gz &&
cd        tables-3.6.1                       &&

python setup.py build -j 64               &&
sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf tables-3.6.1
```

### pandas

Implementation of the panel data concept for `Python`. Provides the `DataFrame` class, foundational to in-memory tabular data manipulation and analysis. 

```sh
wget https://files.pythonhosted.org/packages/e8/81/f7be049fe887865200a0450b137f2c574647b9154503865502cfd720ab5d/pandas-1.2.4.tar.gz -P /sources/python &&

tar xzvf /sources/python/pandas-1.2.4.tar.gz &&
cd        pandas-1.2.4                       &&

python setup.py build -j 64  &&
sudo python setup.py install &&

cd .. &&
sudo rm -rf pandas-1.2.4
```

### pandas-datareader

Functions for reading data from remote APIs into `pandas` `DataFrames`, split out from the core library.

```sh
wget https://files.pythonhosted.org/packages/3e/60/233346865ffbac1d8bd0af01eeb5dba7f84be640cf3206a85722b735a972/pandas-datareader-0.9.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/pandas-datareader-0.9.0.tar.gz &&
cd        pandas-datareader-0.9.0                       &&

sudo python setup.py install &&

cd .. &&
sudo rm -rf pandas-datareader-0.9.0
```

### patsy

`Python` package for creating design matrices.

```sh
wget https://files.pythonhosted.org/packages/49/c7/b971d8685c52512dbaa45bf8d076695432245a9f59509fb20a6c8e4ff69a/patsy-0.5.1.tar.gz -P /sources/python &&

tar xzvf /sources/python/patsy-0.5.1.tar.gz &&
cd        patsy-0.5.1                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf patsy-0.5.1
```

### statsmodels

Statistical modeling functionality beyond what is offered by `SciPy`.

```sh
wget https://files.pythonhosted.org/packages/10/26/0fd61f95667e56fd597ecca715dff3623ed1122b6f895ed2b4dfb54afc04/statsmodels-0.12.2.tar.gz -P /sources/python &&

tar xzvf /sources/python/statsmodels-0.12.2.tar.gz &&
cd        statsmodels-0.12.2                       &&

python setup.py build -j 64  &&
sudo python setup.py install &&

cd .. &&
sudo rm -rf statsmodels-0.12.2
```

### scikit-learn

`Python` implementation of machine learning algorithms built on top of `SciPy`.

```sh
wget https://files.pythonhosted.org/packages/05/04/507280f20fafc8bc94b41e0592938c6f4a910d0e066be7c8ff1299628f5d/scikit-learn-0.24.2.tar.gz -P /sources/python &&

tar xzvf /sources/python/scikit-learn-0.24.2.tar.gz &&
cd        scikit-learn-0.24.2                       &&

python setup.py build -j 64
sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf scikit-learn-0.24.2
```

### xarray

Labelled ndarrays.

```sh
wget https://files.pythonhosted.org/packages/6c/3f/fee12087871642bce75e1e1a88363e709e6feb3a01050dd381b1977fc981/xarray-0.18.2.tar.gz -P /sources/python &&

tar xzvf /sources/python/xarray-0.18.2.tar.gz &&
cd        xarray-0.18.2                       &&

python setup.py build -j 64
sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf xarray-0.18.2
```

### pandarallel

Parallelize operations on `pandas` `DataFrames`.

```sh
wget https://files.pythonhosted.org/packages/f9/c9/2350222cec65593ab5f2f00f2e57dfd1fa4e697dbe92fcaff641485354e6/pandarallel-1.5.2.tar.gz -P /sources/python &&

tar xzvf /sources/python/pandarallel-1.5.2.tar.gz &&
cd        pandarallel-1.5.2                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf pandarallel-1.5.2
```

### pandas-flavor

Add custom extensions to `pandas` `DataFrame` and `Series` objects.

```sh
wget https://files.pythonhosted.org/packages/b1/8b/a92a6ecb43ff6f2354a68c05ff5ec923505efe604eb46c54ee203ae2d2a3/pandas_flavor-0.2.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/pandas_flavor-0.2.0.tar.gz &&
cd        pandas_flavor-0.2.0                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf pandas_flavor-0.2.0
```

### pyjanitor

Data cleaning for `pandas` `DataFrames`.

```sh
wget https://files.pythonhosted.org/packages/f4/d0/52b11c355c93b53ecb5efab68ae3d3da557745ef0e7cbef82f82031b2590/pyjanitor-0.20.14.tar.gz -P /sources/python &&

tar xzvf /sources/python/pyjanitor-0.20.14.tar.gz &&
cd        pyjanitor-0.20.14                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf pyjanitor-0.20.14
```

### pandas-path

Offers `pathlib` style functions for `pandas` `Series` objects.

```sh
wget https://files.pythonhosted.org/packages/c9/a1/4f433fa8c870ec43cb5c0c3fff22a2953bdc57f5570dac42dd0ff1523b41/pandas_path-0.3.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/pandas_path-0.3.0.tar.gz &&
cd        pandas_path-0.3.0                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf pandas_path-0.3.0
```

### sklearn-pandas

`DataFrame` integration to `scikit-learn`. Feed your learning algorithms tabular data directly without having to convert to a matrix first.

```sh
wget https://files.pythonhosted.org/packages/4e/86/fb4b70763f12dfe5585d8741f50b5b55c58fdf5c1f255e32827518e440ac/sklearn-pandas-2.2.0.tar.gz -P /sources/python &&

tar xzvf /sources/python/sklearn-pandas-2.2.0.tar.gz &&
cd        sklearn-pandas-2.2.0                       &&

sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf sklearn-pandas-2.2.0
```

## Data Visualization

### seaborn

Statistical data visualization library. Built on top of `matplotlib` and tightly integrated with `pandas`.

```sh
wget https://files.pythonhosted.org/packages/ef/f4/1927dc0e07f34d54617ce7d31e83b0e3345f14e893b138e44eddd5fad806/seaborn-0.11.1.tar.gz -P /sources/python &&

tar xzvf /sources/python/seaborn-0.11.1.tar.gz &&
cd        seaborn-0.11.1                       &&

sudo python setup.py install &&

cd .. &&
sudo rm -rf seaborn-0.11.1
```

### bokeh

Browser-based data visualization.

```sh
wget https://files.pythonhosted.org/packages/9e/8b/0ead79a7673e0360ed2d06864587af00f57bfaf0ead5327dddf90a466778/bokeh-2.3.2.tar.gz -P /sources/python &&

tar xzvf /sources/python/bokeh-2.3.2.tar.gz &&
cd        bokeh-2.3.2                       &&

sudo python setup.py install &&

cd .. &&
sudo rm -rf bokeh-2.3.2
```

### pandas-bokeh

`pandas` plotting backend for `bokeh`.

```sh
wget https://files.pythonhosted.org/packages/52/52/909116aad919a11c18a4c45afececa03076b05a5d2971fec244d8f7ef12d/pandas-bokeh-0.5.5.tar.gz -P /sources/python &&

tar xzvf /sources/python/pandas-bokeh-0.5.5.tar.gz &&
cd        pandas-bokeh-0.5.5                       &&

sudo python setup.py install &&

cd .. &&
sudo rm -rf pandas-bokeh-0.5.5
```

### networkx

Library for visualizing networks.

```sh
wget https://files.pythonhosted.org/packages/b0/21/adfbf6168631e28577e4af9eb9f26d75fe72b2bb1d33762a5f2c425e6c2a/networkx-2.5.1.tar.gz -P /sources/python &&

tar xzvf /sources/python/networkx-2.5.1.tar.gz &&
cd        networkx-2.5.1                       &&

python setup.py build -j 64               &&
sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf networkx-2.5.1
```

### scikit-image

Image processing for `Python`.

```sh
wget https://files.pythonhosted.org/packages/6e/be/a8ccf9d949a55023cf02438310e068c263ce3dd6bbf31f9229d3db6e551a/scikit-image-0.18.1.tar.gz -P /sources/python &&

tar xzvf /sources/python/scikit-image-0.18.1.tar.gz &&
cd        scikit-image-0.18.1                       &&

python setup.py build -j 64               &&
sudo python setup.py install --optimize=1 &&

cd .. &&
sudo rm -rf scikit-image-0.18.1
```
