---
layout: post
title: Installation
category: docs
---
{% include JB/setup %}


### Dependencies

SINGA is developed and tested on Linux platforms with the following external libraries.

The following dependenies are required:

  * glog version 0.3.3

  * google-protobuf version 2.6.0

  * openblas version >= 0.2.10

  * zeromq version >= 3.2

  * czmq version >= 3

  * zookeeper version 3.4.6


Optional dependencies include:

  * gtest version 1.7.0

  * opencv version 2.4.9

  * lmdb version 0.9.10


SINGA comes with a script for installing the external libraries (see below).

### Building SINGA from source

The build system of SINGA is based on GNU autotools. To build SINGA, you need gcc version >= 4.8.
The common steps to build SINGA can be:

	1.Extract source files;
	2.Run configure script to generate makefiles;
	3.Build SINGA.

There are three ways to build SINGA,

  * If you want to use the latest code, please clone it from
  [Github](https://github.com/apache/incubator-singa.git) and execute
  the following commands,

        $ git clone git@github.com:apache/incubator-singa.git
        $ cd incubator-singa
        $ ./autogen.sh
        $ ./configure
        $ make


  * If you download a release package, please follow the instructions below,

        $ tar xvf singa-xxx
        $ cd singa-xxx
        $ ./configure
        $ make

    Some features of SINGA depend on external libraries. These features can be
    compiled with `--enable-<feature>`.
    For example, to build SINGA with lmdb support, you can run:

        $ ./configure --enable-lmdb


  * In case you do not have the GNU auto tools to run `autogen.sh`, SINGA
  provides a Makefile.example file, which is used as

        $ cp Makefile.example Makefile
        $ make

    Code depending on lmdb can be added into the compilation by

        make -DUSE_LMDB


After compiling SINGA successfully, the `libsinga.so` will be generated into
.lib/ folder and an executable file `singa` is generated under bin/.

If some dependent libraries are missing (or not detected), you can use the
following script to download and install them:

    $ cd thirdparty
    $ ./install.sh MISSING_LIBRARY_NAME1 YOUR_INSTALL_PATH1 MISSING_LIBRARY_NAME2 YOUR_INSTALL_PATH2 ...

If you do not specify the installation path, the library will be installed in
the default folder specified by the software itself.  For example, if you want
to build `zeromq` library in system folder and `gflags` in `/usr/local`, just run:

    $ ./install.sh zeromq gflags /usr/local

You can also install all dependencies in `/usr/local` directory:

    $ ./install.sh all /usr/local

Here is a table showing the first arguments:

    MISSING_LIBRARY_NAME  LIBRARIES
    cmake                 cmake tools
    czmq*                 czmq lib
    gflags                gflags lib
    glog                  glog lib
    lmdb                  lmdb lib
    OpenBLAS              OpenBLAS lib
    opencv                OpenCV
    protobuf              Google protobuf
    zeromq                zeromq lib
    zookeeper             Apache zookeeper

*: Since `czmq` depends on `zeromq`, the script offers you one more argument to
indicate `zeromq` location.
The installation commands of `czmq` can be:

    $./install.sh czmq  /usr/local /usr/local/zeromq

After the execution, `czmq` will be installed in `/usr/local` while `zeromq` is
installed in `/usr/local/zeromq`.

### FAQ

Q1:While compiling SINGA and installing `glog` on max OS X, I get fatal error
`'ext/slist' file not found`

A1:You may install `glog` individually and try command :

    $ make CFLAGS='-stdlib=libstdc++' CXXFLAGS='stdlib=libstdc++'


Q2:While compiling SINGA, I get error `SSE2 instruction set not enabled`

A2:You can try following command:

    $ make CFLAGS='-msse2' CXXFLAGS='-msse2'

Q3:I get error `./configure --> cannot find blas_segmm() function` even I
run `install.sh OpenBLAS`.

A3:Since `OpenBLAS` library is installed in `/opt` folder by default or
`/other/folder` by your preference, you may edit your environment settings.
You need add its default installation directories before linking, just
run:

    $ export LDFLAGS=-L/opt

Or as an alternative option, you can also edit LIBRARY_PATH to figure it out.


Q4:I get ImportError from google.protobuf.internal when I try to import .py
files.

    ImportError: cannot import name enum_type_wrapper

A4:After install google protobuf by "make install", we should install python
runtime libraries. Go to protobuf source directory, run:

    $ cd /PROTOBUF/SOURCE/FOLDER
    $ cd python
    $ python setup.py build
    $ python setup.py install

You may need `sudo` when you try to install python runtime libraries in
system folder.
