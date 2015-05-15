# npscipy-binaries

This repository contains prebuilt ATLAS, numpy, and scipy packages suitable for
running on Heroku's `cedar-14` stack.

## Creating new binary releases

Binary packages should be created in the same environment that they will be run
on. The [`cedar-14` stack](https://devcenter.heroku.com/articles/stack) is
based on Ubuntu 14.04.

You may be able to build suitable binaries by using

  `heroku run bash --app [app name]`

These binaries have been built using a base Ubuntu 14.04 EC2 instance and the
Heroku python interpreter instead.

### General

1. Install the required compiler packages:

        sudo apt-get install build-essential gfortran

2. Set up the Heroku python interpreter:

        wget https://lang-python.s3.amazonaws.com/cedar-14/runtimes/python-2.7.9.tar.gz
        mkdir -p heroku-python
        tar xvfz python-2.7.9.tar.gz -C heroku-python

### ATLAS

1. Download the required precompiled libraries:

        sudo apt-get update
        apt-get download libatlas3-base libatlas-base-dev libgfortran3

2. Extract each and move the libraries into the correct hierarchy:

        mkdir atlas
        cd atlas
        dpkg -x ../libatlas-base-dev*.deb .
        dpkg -x ../libatlas3-base*.deb .
        dpkg -x ../libgfortran3*.deb .
        mv ./usr/lib .
        rm -rf ./usr
        mv ./lib/x86_64-linux-gnu/* ./lib/
        rmdir ./lib/x86_64-linux-gnu

3. Create a tarball containing the binaries:

        tar cvfz ../atlas-cedar-14.tar.gz ./lib/

4. Set up environment variables to point to these libraries for later builds:

        export ATLAS=$(pwd)/lib/atlas-base/libatlas.a
        export LIBRARY_PATH="$LIBRARY_PATH:$(pwd)/lib:$(pwd)/lib/atlas-base"
        export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:$(pwd)/lib:$(pwd)/lib/atlas-base"
        cd ..

### Numpy

1. Download and unpack the source of version you are building:

        wget https://pypi.python.org/packages/source/n/numpy/numpy-1.6.1.tar.gz
        tar xvfz ../numpy-1.6.1.tar.gz
        cd numpy-1.6.1

2. Build a `bdist` tarball using the heroku python interpreter:

        ../heroku-python/bin/python setup.py bdist

3. Fix up the paths in the `bdist` tarball:

        cd ..
        mkdir -p numpy
        cd numpy
        tar xvfz ../numpy-1.6.1/dist/numpy-*.tar.gz
        mv ./home/ubuntu/heroku-python/lib/ ./
        rm -rf ./home
        tar cvfz ../numpy-1.6.1-cedar-14.tar.gz ./lib/
        cd ..

4. Install the `bdist` into place:

        tar xvfz numpy-1.6.1-cedar-14.tar.gz -C heroku-python

### Scipy

1. Download and unpack the source of version you are building:

        wget https://pypi.python.org/packages/source/s/scipy/scipy-0.9.0.tar.gz
        tar xvfz scipy-0.9.0.tar.gz
        cd scipy-1.6.1

2. Build a `bdist` tarball using the heroku python interpreter:

        ../heroku-python/bin/python setup.py bdist

3. Fix up the paths in the `bdist` tarball:

        cd ..
        mkdir -p scipy
        cd scipy
        tar xvfz ../scipy-0.9.0/dist/scipy-*.tar.gz
        mv ./home/ubuntu/heroku-python/lib/ ./
        rm -rf ./home
        tar cvfz ../scipy-0.9.0-cedar-14.tar.gz ./lib/
        cd ..
